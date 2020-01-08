---
layout: post
title: "Android P system bt启动流程追踪"
tags: [android,bluetooth]
comments: true
---

程序入口文件

`bt\service\main.cc`
```c++
int main(int argc, char* argv[]) {
    ...

    // [标记]
    if (!bluetooth::Daemon::Initialize()) {
        LOG(ERROR) << "Failed to initialize Daemon";
        return EXIT_FAILURE;
    }

    // Start the main event loop.
    bluetooth::Daemon::Get()->StartMainLoop();

    // The main message loop has exited; clean up the Daemon.
    bluetooth::Daemon::Get()->ShutDown();

    return EXIT_SUCCESS;
}
```
`bt\service\daemon.cc`
```c++
// static
bool Daemon::Initialize() {
    ...

    // [标记]
    if (g_daemon->Init()) return true;

    ...
}

namespace bluetooth {

namespace {

// The global Daemon instance.
Daemon* g_daemon = nullptr;

class DaemonImpl : public Daemon, public ipc::IPCManager::Delegate {
    ...

    // ipc::IPCManager::Delegate implementation:
    void OnIPCHandlerStarted(ipc::IPCManager::Type /* type */) override {
        if (!settings_->EnableOnStart()) return;
        // [标记]
        adapter_->Enable(false /* start_restricted */);
    }

    bool StartUpBluetoothInterfaces() {
        // [标记]
        if (!hal::BluetoothInterface::Initialize()) goto failed;

        if (!hal::BluetoothGattInterface::Initialize()) goto failed;

        return true;

    failed:
        ShutDownBluetoothInterfaces();
        return false;
    }

    bool Init() override {
        ...

        // [标记]
        if (!StartUpBluetoothInterfaces()) {
            LOG(ERROR) << "Failed to set up HAL Bluetooth interfaces";
            return false;
        }

        // [标记]
        adapter_ = Adapter::Create();
        ipc_manager_.reset(new ipc::IPCManager(adapter_.get()));

        ...
    }

    ...
}
```
`bt\service\hal\bluetooth_interface.cc`
```c++
class BluetoothInterfaceImpl : public BluetoothInterface {
    ...

    // Initialize the interface. This loads the shared Bluetooth library and sets
    // up the callbacks.
    bool Initialize() {
        // Load the Bluetooth shared library module.
        const bt_interface_t* interface;
        // [标记]
        // libbluetooth.so的加载 
        // bt\btcore\src\hal_util.cc
        int status = hal_util_load_bt_library(&interface);
        if (status) {
        LOG(ERROR) << "Failed to open the Bluetooth module";
        return false;
        }

        hal_iface_ = interface;

        // Initialize the Bluetooth interface. Set up the adapter (Bluetooth DM) API
        // callbacks.
        // [标记]
        status = hal_iface_->init(&bt_callbacks);
        if (status != BT_STATUS_SUCCESS) {
        LOG(ERROR) << "Failed to initialize Bluetooth stack";
        return false;
        }

        status = hal_iface_->set_os_callouts(&bt_os_callouts);
        if (status != BT_STATUS_SUCCESS) {
        LOG(ERROR) << "Failed to set up Bluetooth OS callouts";
        return false;
        }

        return true;
    }

    ...
}
```
`bt\service\adapter.cc`
```c++
namespace bluetooth {

    class AdapterImpl : public Adapter, public hal::BluetoothInterface::Observer {
        // [标记]
        bool Enable(bool start_restricted) override {
        AdapterState current_state = GetState();
        if (current_state != ADAPTER_STATE_OFF) {
            LOG(INFO) << "Adapter not disabled - state: "
                    << AdapterStateToString(current_state);
            return false;
        }

        // Set the state before calling enable() as there might be a race between
        // here and the AdapterStateChangedCallback.
        state_ = ADAPTER_STATE_TURNING_ON;
        NotifyAdapterStateChanged(current_state, state_);

        // [标记]
        int status = hal::BluetoothInterface::Get()->GetHALInterface()->enable(
            start_restricted);
        if (status != BT_STATUS_SUCCESS) {
            LOG(ERROR) << "Failed to enable Bluetooth - status: "
                        << BtStatusText((const bt_status_t)status);
            state_ = ADAPTER_STATE_OFF;
            NotifyAdapterStateChanged(ADAPTER_STATE_TURNING_ON, state_);
            return false;
        }

        return true;
        }
    }

    // static
    // [标记]
    std::unique_ptr<Adapter> Adapter::Create() {
    return std::unique_ptr<Adapter>(new AdapterImpl());
    }
}
```

`bt\btif\src\bluetooth.cc`
```c++
/*****************************************************************************
 *
 *   BLUETOOTH HAL INTERFACE FUNCTIONS
 *
 ****************************************************************************/

static int init(bt_callbacks_t* callbacks) {
    LOG_INFO(LOG_TAG, "%s", __func__);

    if (interface_ready()) return BT_STATUS_DONE;

    #ifdef BLUEDROID_DEBUG
        allocation_tracker_init();
    #endif

    bt_hal_cbacks = callbacks;
    // [标记]
    stack_manager_get_interface()->init_stack();
    btif_debug_init();
    return BT_STATUS_SUCCESS;
}

static int enable(bool start_restricted) {
  LOG_INFO(LOG_TAG, "%s: start restricted = %d", __func__, start_restricted);

  restricted_mode = start_restricted;

  if (!interface_ready()) return BT_STATUS_NOT_READY;

  stack_manager_get_interface()->start_up_stack_async();
  return BT_STATUS_SUCCESS;
}
```
`bt\btif\src\stack_manager.cc`
```c++
    ...

    // [标记]
    const stack_manager_t* stack_manager_get_interface() {
        ensure_manager_initialized();
        return &interface;
    }

    ...

    // Interface functions

    static void init_stack(void) {
        ...

        // [标记]
        thread_post(management_thread, event_init_stack, semaphore);
        
        ...
    }

    static void start_up_stack_async(void) {
        // [标记]
        thread_post(management_thread, event_start_up_stack, NULL);
    }

    // Synchronous function to initialize the stack
    static void event_init_stack(void* context) {
        ...

        // [标记]
        btif_init_bluetooth();

        ...
    }

    // Synchronous function to start up the stack
    static void event_start_up_stack(UNUSED_ATTR void* context) {
        ...

        // [标记]
        bte_main_enable();

        ...
    }
```
`bt\btif\src\btif_core.cc`
```c++
/*******************************************************************************
 *
 * Function         btif_init_bluetooth
 *
 * Description      Creates BTIF task and prepares BT scheduler for startup
 *
 * Returns          bt_status_t
 *
 ******************************************************************************/
bt_status_t btif_init_bluetooth() {
    LOG_INFO(LOG_TAG, "%s entered", __func__);

    // [标记]
    bte_main_boot_entry();

    bt_jni_workqueue_thread = thread_new(BT_JNI_WORKQUEUE_NAME);
    if (bt_jni_workqueue_thread == NULL) {
        LOG_ERROR(LOG_TAG, "%s Unable to create thread %s", __func__,
                BT_JNI_WORKQUEUE_NAME);
        goto error_exit;
    }

    thread_post(bt_jni_workqueue_thread, run_message_loop, nullptr);

    LOG_INFO(LOG_TAG, "%s finished", __func__);
    return BT_STATUS_SUCCESS;

    error_exit:;
        thread_free(bt_jni_workqueue_thread);

        bt_jni_workqueue_thread = NULL;

        return BT_STATUS_FAIL;
}
```
`bt\main\bte_main.cc`
```c++
/******************************************************************************
 *
 * Function         bte_main_boot_entry
 *
 * Description      BTE MAIN API - Entry point for BTE chip/stack initialization
 *
 * Returns          None
 *
 *****************************************************************************/
void bte_main_boot_entry(void) {
    ...

    // [标记]
    hci = hci_layer_get_interface();
    if (!hci) {
        LOG_ERROR(LOG_TAG, "%s could not get hci layer interface.", __func__);
        return;
    }

    ...
}

/******************************************************************************
 *
 * Function         bte_main_enable
 *
 * Description      BTE MAIN API - Creates all the BTE tasks. Should be called
 *                  part of the Bluetooth stack enable sequence
 *
 * Returns          None
 *
 *****************************************************************************/
void bte_main_enable() {
    ...

    // [标记]
    BTU_StartUp();
}
```
`bt\hci\src\hci_layer.cc`
```c++
const hci_t* hci_layer_get_interface() {
    buffer_allocator = buffer_allocator_get_interface();
    btsnoop = btsnoop_get_interface();
    packet_fragmenter = packet_fragmenter_get_interface();

    // [标记]
    init_layer_interface();

    return &interface;
}

static void init_layer_interface() {
    if (!interface_created) {
        // It's probably ok for this to live forever. It's small and
        // there's only one instance of the hci interface.

        interface.set_data_cb = set_data_cb;
        interface.transmit_command = transmit_command;
        interface.transmit_command_futured = transmit_command_futured;
        interface.transmit_downward = transmit_downward;
        interface_created = true;
    }
}
```
`bt\stack\btu\btu_init.cc`
```c++
    ...

    /*****************************************************************************
    *
    * Function         BTU_StartUp
    *
    * Description      Initializes the BTU control block.
    *
    *                  NOTE: Must be called before creating any tasks
    *                      (RPC, BTU, HCIT, APPL, etc.)
    *
    * Returns          void
    *
    *****************************************************************************/
    void BTU_StartUp(void) {
        btu_trace_level = HCI_INITIAL_TRACE_LEVEL;

        bt_workqueue_thread = thread_new(BT_WORKQUEUE_NAME);
        if (bt_workqueue_thread == NULL) goto error_exit;

        thread_set_rt_priority(bt_workqueue_thread, BTU_TASK_RT_PRIORITY);

        // Continue startup on bt workqueue thread.
        thread_post(bt_workqueue_thread, btu_task_start_up, NULL);
        return;

        error_exit:;
        LOG_ERROR(LOG_TAG, "%s Unable to allocate resources for bt_workqueue",
                    __func__);
        BTU_ShutDown();
    }

    ...
```



---
layout: post
title: "Android P HFP Profile 代码流程"
tags: [bluetooth,source code,android p,hfp,profile]
comments: true
---

\packages\apps\Bluetooth\jni\com_android_bluetooth.h
```java
class CallbackEnv {
    ...
    int register_com_android_bluetooth_hfp(JNIEnv* env);

    int register_com_android_bluetooth_hfpclient(JNIEnv* env);
    ...
}
```
\system\bt\include\hardware\bluetooth_headset_callbacks.h
```java
class Callbacks {
    public:
    virtual ~Callbacks() = default;
    /**
    * Callback for connection state change.
    *
    * @param state one of the values from bthf_connection_state_t
    * @param bd_addr remote device address
    */
    virtual void ConnectionStateCallback(bthf_connection_state_t state,
                                        RawAddress* bd_addr) = 0;

    ...
}
```
\system\bt\include\hardware\bluetooth_headset_interface.h
```java
class Interface {
    public:
    virtual ~Interface() = default;
    /**
    * Register the BtHf callbacks
    *
    * @param callbacks callbacks for the user of the native stack
    * @param max_hf_clients maximum number of headset clients
    * @param inband_ringing_enabled whether inband ringtone is enabled
    * @return BT_STATUS_SUCCESS on success
    */
    virtual bt_status_t Init(Callbacks* callbacks, int max_hf_clients,
                            bool inband_ringing_enabled) = 0;

    ...
```
\packages\apps\Bluetooth\jni\com_android_bluetooth_hfp.cpp
```java
static jmethodID method_onConnectionStateChanged;
...
static bluetooth::headset::Interface* sBluetoothHfpInterface = nullptr;
...
class JniHeadsetCallbacks : bluetooth::headset::Callbacks {
 public:
  static bluetooth::headset::Callbacks* GetInstance() {
    static bluetooth::headset::Callbacks* instance = new JniHeadsetCallbacks();
    return instance;
  }

  void ConnectionStateCallback(
      bluetooth::headset::bthf_connection_state_t state,
      RawAddress* bd_addr) override {
    ALOGI("%s %d for %s", __func__, state, bd_addr->ToString().c_str());

    std::shared_lock<std::shared_timed_mutex> lock(callbacks_mutex);
    CallbackEnv sCallbackEnv(__func__);
    if (!sCallbackEnv.valid() || !mCallbacksObj) return;

    ScopedLocalRef<jbyteArray> addr(sCallbackEnv.get(), marshall_bda(bd_addr));
    if (!addr.get()) return;

    sCallbackEnv->CallVoidMethod(mCallbacksObj, method_onConnectionStateChanged,
                                 (jint)state, addr.get());
  }
  ...
}
...
static void classInitNative(JNIEnv* env, jclass clazz) {
  method_onConnectionStateChanged =
      env->GetMethodID(clazz, "onConnectionStateChanged", "(I[B)V");
  method_onAudioStateChanged =
      env->GetMethodID(clazz, "onAudioStateChanged", "(I[B)V");
  method_onVrStateChanged =
      env->GetMethodID(clazz, "onVrStateChanged", "(I[B)V");
  method_onAnswerCall = env->GetMethodID(clazz, "onAnswerCall", "([B)V");
  method_onHangupCall = env->GetMethodID(clazz, "onHangupCall", "([B)V");
  method_onVolumeChanged =
      env->GetMethodID(clazz, "onVolumeChanged", "(II[B)V");
  method_onDialCall =
      env->GetMethodID(clazz, "onDialCall", "(Ljava/lang/String;[B)V");
  method_onSendDtmf = env->GetMethodID(clazz, "onSendDtmf", "(I[B)V");
  method_onNoiceReductionEnable =
      env->GetMethodID(clazz, "onNoiceReductionEnable", "(Z[B)V");
  method_onWBS = env->GetMethodID(clazz, "onWBS", "(I[B)V");
  method_onAtChld = env->GetMethodID(clazz, "onAtChld", "(I[B)V");
  method_onAtCnum = env->GetMethodID(clazz, "onAtCnum", "([B)V");
  method_onAtCind = env->GetMethodID(clazz, "onAtCind", "([B)V");
  method_onAtCops = env->GetMethodID(clazz, "onAtCops", "([B)V");
  method_onAtClcc = env->GetMethodID(clazz, "onAtClcc", "([B)V");
  method_onUnknownAt =
      env->GetMethodID(clazz, "onUnknownAt", "(Ljava/lang/String;[B)V");
  method_onKeyPressed = env->GetMethodID(clazz, "onKeyPressed", "([B)V");
  method_onAtBind =
      env->GetMethodID(clazz, "onATBind", "(Ljava/lang/String;[B)V");
  method_onAtBiev = env->GetMethodID(clazz, "onATBiev", "(II[B)V");
  method_onAtBia = env->GetMethodID(clazz, "onAtBia", "(ZZZZ[B)V");

  ALOGI("%s: succeeds", __func__);
}

static void initializeNative(JNIEnv* env, jobject object, jint max_hf_clients,
                             jboolean inband_ringing_enabled) {
  std::unique_lock<std::shared_timed_mutex> interface_lock(interface_mutex);
  std::unique_lock<std::shared_timed_mutex> callbacks_lock(callbacks_mutex);

  const bt_interface_t* btInf = getBluetoothInterface();
  if (!btInf) {
    ALOGE("%s: Bluetooth module is not loaded", __func__);
    jniThrowIOException(env, EINVAL);
    return;
  }

  if (sBluetoothHfpInterface) {
    ALOGI("%s: Cleaning up Bluetooth Handsfree Interface before initializing",
          __func__);
    sBluetoothHfpInterface->Cleanup();
    sBluetoothHfpInterface = nullptr;
  }

  if (mCallbacksObj) {
    ALOGI("%s: Cleaning up Bluetooth Handsfree callback object", __func__);
    env->DeleteGlobalRef(mCallbacksObj);
    mCallbacksObj = nullptr;
  }

  sBluetoothHfpInterface =
      (bluetooth::headset::Interface*)btInf->get_profile_interface(
          BT_PROFILE_HANDSFREE_ID);
  if (!sBluetoothHfpInterface) {
    ALOGW("%s: Failed to get Bluetooth Handsfree Interface", __func__);
    jniThrowIOException(env, EINVAL);
  }
  bt_status_t status =
      sBluetoothHfpInterface->Init(JniHeadsetCallbacks::GetInstance(),
                                   max_hf_clients, inband_ringing_enabled);
  if (status != BT_STATUS_SUCCESS) {
    ALOGE("%s: Failed to initialize Bluetooth Handsfree Interface, status: %d",
          __func__, status);
    sBluetoothHfpInterface = nullptr;
    return;
  }

  mCallbacksObj = env->NewGlobalRef(object);
}
...
static JNINativeMethod sMethods[] = {
    {"classInitNative", "()V", (void*)classInitNative},
    {"initializeNative", "(IZ)V", (void*)initializeNative},
    {"cleanupNative", "()V", (void*)cleanupNative},
    {"connectHfpNative", "([B)Z", (void*)connectHfpNative},
    {"disconnectHfpNative", "([B)Z", (void*)disconnectHfpNative},
    {"connectAudioNative", "([B)Z", (void*)connectAudioNative},
    {"disconnectAudioNative", "([B)Z", (void*)disconnectAudioNative},
    {"startVoiceRecognitionNative", "([B)Z",
     (void*)startVoiceRecognitionNative},
    {"stopVoiceRecognitionNative", "([B)Z", (void*)stopVoiceRecognitionNative},
    {"setVolumeNative", "(II[B)Z", (void*)setVolumeNative},
    {"notifyDeviceStatusNative", "(IIII[B)Z", (void*)notifyDeviceStatusNative},
    {"copsResponseNative", "(Ljava/lang/String;[B)Z",
     (void*)copsResponseNative},
    {"cindResponseNative", "(IIIIIII[B)Z", (void*)cindResponseNative},
    {"atResponseStringNative", "(Ljava/lang/String;[B)Z",
     (void*)atResponseStringNative},
    {"atResponseCodeNative", "(II[B)Z", (void*)atResponseCodeNative},
    {"clccResponseNative", "(IIIIZLjava/lang/String;I[B)Z",
     (void*)clccResponseNative},
    {"phoneStateChangeNative", "(IIILjava/lang/String;I[B)Z",
     (void*)phoneStateChangeNative},
    {"setScoAllowedNative", "(Z)Z", (void*)setScoAllowedNative},
    {"sendBsirNative", "(Z[B)Z", (void*)sendBsirNative},
    {"setActiveDeviceNative", "([B)Z", (void*)setActiveDeviceNative},
};

int register_com_android_bluetooth_hfp(JNIEnv* env) {
  return jniRegisterNativeMethods(
      env, "com/android/bluetooth/hfp/HeadsetNativeInterface", sMethods,
      NELEM(sMethods));
}
```
\packages\apps\Bluetooth\jni\com_android_bluetooth_btservice_AdapterService.cpp
```java
...
/*
 * JNI Initialization
 */
jint JNI_OnLoad(JavaVM* jvm, void* reserved) {
    JNIEnv* e;
    int status;

    ALOGV("Bluetooth Adapter Service : loading JNI\n");

    ...
    status = android::register_com_android_bluetooth_hfp(e);
    if (status < 0) {
        ALOGE("jni hfp registration failure, status: %d", status);
        return JNI_ERR;
    }
    ...
}
...
```
\packages\apps\Bluetooth\src\com\android\bluetooth\hfp\HeadsetNativeInterface.java
```java
public class HeadsetNativeInterface {
    ...
    private void sendMessageToService(HeadsetStackEvent event) {
        HeadsetService service = HeadsetService.getHeadsetService();
        if (service != null) {
            service.messageFromNative(event);
        } else {
            // Service must call cleanup() when quiting and native stack shouldn't send any event
            // after cleanup() -> cleanupNative() is called.
            Log.wtfStack(TAG, "FATAL: Stack sent event while service is not available: " + event);
        }
    }
    ...
    void onConnectionStateChanged(int state, byte[] address) {
        HeadsetStackEvent event =
                new HeadsetStackEvent(HeadsetStackEvent.EVENT_TYPE_CONNECTION_STATE_CHANGED, state,
                        getDevice(address));
        sendMessageToService(event);
    }

    // Callbacks for native code

    private void onAudioStateChanged(int state, byte[] address) {
        HeadsetStackEvent event =
                new HeadsetStackEvent(HeadsetStackEvent.EVENT_TYPE_AUDIO_STATE_CHANGED, state,
                        getDevice(address));
        sendMessageToService(event);
    }
    ...
    /* Native methods */
    private static native void classInitNative();
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\hfp\HeadsetService.java
```java
public class HeadsetService extends ProfileService {
    private HeadsetNativeInterface mNativeInterface;
    ...
    @Override
    protected boolean start() {
        Log.i(TAG, "start()");
        ...
        // Step 4: Initialize native interface
        ...
        mNativeInterface = HeadsetObjectsFactory.getInstance().getNativeInterface();
        // Add 1 to allow a pending device to be connecting or disconnecting
        mNativeInterface.init(mMaxHeadsetConnections + 1, isInbandRingingEnabled());
        ...
    }

    @Override
    protected boolean stop() {
        Log.i(TAG, "stop()");
        ...
        // Step 4: Destroy native interface
        mNativeInterface.cleanup();
        ...
    }
    ...
    /**
     * Handlers for incoming service calls
     */
    private static class BluetoothHeadsetBinder extends IBluetoothHeadset.Stub
            implements IProfileServiceBinder {
        ...
        @Override
        public boolean connect(BluetoothDevice device) {
            HeadsetService service = getService();
            if (service == null) {
                return false;
            }
            return service.connect(device);
        }
        ...
    }
    ...
    /**
     * Handle messages from native (JNI) to Java. This needs to be synchronized to avoid posting
     * messages to state machine before start() is done
     *
     * @param stackEvent event from native stack
     */
    void messageFromNative(HeadsetStackEvent stackEvent) {
        Objects.requireNonNull(stackEvent.device,
                "Device should never be null, event: " + stackEvent);
        synchronized (mStateMachines) {
            HeadsetStateMachine stateMachine = mStateMachines.get(stackEvent.device);
            if (stackEvent.type == HeadsetStackEvent.EVENT_TYPE_CONNECTION_STATE_CHANGED) {
                switch (stackEvent.valueInt) {
                    case HeadsetHalConstants.CONNECTION_STATE_CONNECTED:
                    case HeadsetHalConstants.CONNECTION_STATE_CONNECTING: {
                        // Create new state machine if none is found
                        if (stateMachine == null) {
                            stateMachine = HeadsetObjectsFactory.getInstance()
                                    .makeStateMachine(stackEvent.device,
                                            mStateMachinesThread.getLooper(), this, mAdapterService,
                                            mNativeInterface, mSystemInterface);
                            mStateMachines.put(stackEvent.device, stateMachine);
                        }
                        break;
                    }
                }
            }
            if (stateMachine == null) {
                throw new IllegalStateException(
                        "State machine not found for stack event: " + stackEvent);
            }
            stateMachine.sendMessage(HeadsetStateMachine.STACK_EVENT, stackEvent);
        }
    }
    ...
    // API methods
    public static synchronized HeadsetService getHeadsetService() {
        if (sHeadsetService == null) {
            Log.w(TAG, "getHeadsetService(): service is NULL");
            return null;
        }
        if (!sHeadsetService.isAvailable()) {
            Log.w(TAG, "getHeadsetService(): service is not available");
            return null;
        }
        logD("getHeadsetService(): returning " + sHeadsetService);
        return sHeadsetService;
    }

    private static synchronized void setHeadsetService(HeadsetService instance) {
        logD("setHeadsetService(): set to: " + instance);
        sHeadsetService = instance;
    }

    public boolean connect(BluetoothDevice device) {
        enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM, "Need BLUETOOTH ADMIN permission");
        if (getPriority(device) == BluetoothProfile.PRIORITY_OFF) {
            Log.w(TAG, "connect: PRIORITY_OFF, device=" + device + ", " + Utils.getUidPidString());
            return false;
        }
        ParcelUuid[] featureUuids = mAdapterService.getRemoteUuids(device);
        if (!BluetoothUuid.containsAnyUuid(featureUuids, HEADSET_UUIDS)) {
            Log.e(TAG, "connect: Cannot connect to " + device + ": no headset UUID, "
                    + Utils.getUidPidString());
            return false;
        }
        synchronized (mStateMachines) {
            Log.i(TAG, "connect: device=" + device + ", " + Utils.getUidPidString());
            HeadsetStateMachine stateMachine = mStateMachines.get(device);
            if (stateMachine == null) {
                stateMachine = HeadsetObjectsFactory.getInstance()
                        .makeStateMachine(device, mStateMachinesThread.getLooper(), this,
                                mAdapterService, mNativeInterface, mSystemInterface);
                mStateMachines.put(device, stateMachine);
            }
            int connectionState = stateMachine.getConnectionState();
            if (connectionState == BluetoothProfile.STATE_CONNECTED
                    || connectionState == BluetoothProfile.STATE_CONNECTING) {
                Log.w(TAG, "connect: device " + device
                        + " is already connected/connecting, connectionState=" + connectionState);
                return false;
            }
            List<BluetoothDevice> connectingConnectedDevices =
                    getDevicesMatchingConnectionStates(CONNECTING_CONNECTED_STATES);
            boolean disconnectExisting = false;
            if (connectingConnectedDevices.size() >= mMaxHeadsetConnections) {
                // When there is maximum one device, we automatically disconnect the current one
                if (mMaxHeadsetConnections == 1) {
                    disconnectExisting = true;
                } else {
                    Log.w(TAG, "Max connection has reached, rejecting connection to " + device);
                    return false;
                }
            }
            if (disconnectExisting) {
                for (BluetoothDevice connectingConnectedDevice : connectingConnectedDevices) {
                    disconnect(connectingConnectedDevice);
                }
                setActiveDevice(null);
            }
            stateMachine.sendMessage(HeadsetStateMachine.CONNECT, device);
        }
        return true;
    }
    ...    
}
```

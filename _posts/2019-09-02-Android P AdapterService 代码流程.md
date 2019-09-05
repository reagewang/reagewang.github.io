---
layout: post
title: "Android P AdapterService 代码流程"
tags: [bluetooth,source code,android p,dapterservice,profile]
comments: true
---

\system\bt\include\hardware\bluetooth.h
```java
...
/** Represents the standard Bluetooth DM interface. */
typedef struct {
  /** set to sizeof(bt_interface_t) */
  size_t size;
  /**
   * Opens the interface and provides the callback routines
   * to the implemenation of this interface.
   */
  int (*init)(bt_callbacks_t* callbacks);

  /** Enable Bluetooth. */
  int (*enable)(bool guest_mode);

  /** Disable Bluetooth. */
  int (*disable)(void);

  /** Closes the interface. */
  void (*cleanup)(void);

  /** Get all Bluetooth Adapter properties at init */
  int (*get_adapter_properties)(void);

  /** Get Bluetooth Adapter property of 'type' */
  int (*get_adapter_property)(bt_property_type_t type);

  /** Set Bluetooth Adapter property of 'type' */
  /* Based on the type, val shall be one of
   * RawAddress or bt_bdname_t or bt_scanmode_t etc
   */
  int (*set_adapter_property)(const bt_property_t* property);

  /** Get all Remote Device properties */
  int (*get_remote_device_properties)(RawAddress* remote_addr);

  /** Get Remote Device property of 'type' */
  int (*get_remote_device_property)(RawAddress* remote_addr,
                                    bt_property_type_t type);

  /** Set Remote Device property of 'type' */
  int (*set_remote_device_property)(RawAddress* remote_addr,
                                    const bt_property_t* property);

  /** Get Remote Device's service record  for the given UUID */
  int (*get_remote_service_record)(const RawAddress& remote_addr,
                                   const bluetooth::Uuid& uuid);

  /** Start SDP to get remote services */
  int (*get_remote_services)(RawAddress* remote_addr);

  /** Start Discovery */
  int (*start_discovery)(void);

  /** Cancel Discovery */
  int (*cancel_discovery)(void);

  /** Create Bluetooth Bonding */
  int (*create_bond)(const RawAddress* bd_addr, int transport);

  /** Create Bluetooth Bond using out of band data */
  int (*create_bond_out_of_band)(const RawAddress* bd_addr, int transport,
                                 const bt_out_of_band_data_t* oob_data);

  /** Remove Bond */
  int (*remove_bond)(const RawAddress* bd_addr);

  /** Cancel Bond */
  int (*cancel_bond)(const RawAddress* bd_addr);

  /**
   * Get the connection status for a given remote device.
   * return value of 0 means the device is not connected,
   * non-zero return status indicates an active connection.
   */
  int (*get_connection_state)(const RawAddress* bd_addr);

  /** BT Legacy PinKey Reply */
  /** If accept==FALSE, then pin_len and pin_code shall be 0x0 */
  int (*pin_reply)(const RawAddress* bd_addr, uint8_t accept, uint8_t pin_len,
                   bt_pin_code_t* pin_code);

  /** BT SSP Reply - Just Works, Numeric Comparison and Passkey
   * passkey shall be zero for BT_SSP_VARIANT_PASSKEY_COMPARISON &
   * BT_SSP_VARIANT_CONSENT
   * For BT_SSP_VARIANT_PASSKEY_ENTRY, if accept==FALSE, then passkey
   * shall be zero */
  int (*ssp_reply)(const RawAddress* bd_addr, bt_ssp_variant_t variant,
                   uint8_t accept, uint32_t passkey);

  /** Get Bluetooth profile interface */
  const void* (*get_profile_interface)(const char* profile_id);

  /** Bluetooth Test Mode APIs - Bluetooth must be enabled for these APIs */
  /* Configure DUT Mode - Use this mode to enter/exit DUT mode */
  int (*dut_mode_configure)(uint8_t enable);

  /* Send any test HCI (vendor-specific) command to the controller. Must be in
   * DUT Mode */
  int (*dut_mode_send)(uint16_t opcode, uint8_t* buf, uint8_t len);
  /** BLE Test Mode APIs */
  /* opcode MUST be one of: LE_Receiver_Test, LE_Transmitter_Test, LE_Test_End
   */
  int (*le_test_mode)(uint16_t opcode, uint8_t* buf, uint8_t len);

  /** Sets the OS call-out functions that bluedroid needs for alarms and wake
   * locks. This should be called immediately after a successful |init|.
   */
  int (*set_os_callouts)(bt_os_callouts_t* callouts);

  /** Read Energy info details - return value indicates BT_STATUS_SUCCESS or
   * BT_STATUS_NOT_READY Success indicates that the VSC command was sent to
   * controller
   */
  int (*read_energy_info)();

  /**
   * Native support for dumpsys function
   * Function is synchronous and |fd| is owned by caller.
   * |arguments| are arguments which may affect the output, encoded as
   * UTF-8 strings.
   */
  void (*dump)(int fd, const char** arguments);

  /**
   * Native support for metrics protobuf dumping. The dumping format will be
   * raw byte array
   *
   * @param output an externally allocated string to dump serialized protobuf
   */
  void (*dumpMetrics)(std::string* output);

  /**
   * Clear /data/misc/bt_config.conf and erase all stored connections
   */
  int (*config_clear)(void);

  /**
   * Clear (reset) the dynamic portion of the device interoperability database.
   */
  void (*interop_database_clear)(void);

  /**
   * Add a new device interoperability workaround for a remote device whose
   * first |len| bytes of the its device address match |addr|.
   * NOTE: |feature| has to match an item defined in interop_feature_t
   * (interop.h).
   */
  void (*interop_database_add)(uint16_t feature, const RawAddress* addr,
                               size_t len);

  /**
   * Get the AvrcpTarget Service interface to interact with the Avrcp Service
   */
  bluetooth::avrcp::ServiceInterface* (*get_avrcp_service)(void);
} bt_interface_t;
...
```
\packages\apps\Bluetooth\jni\com_android_bluetooth_btservice_AdapterService.cpp
```java
...
static jmethodID method_stateChangeCallback;
static jmethodID method_adapterPropertyChangedCallback;
static jmethodID method_devicePropertyChangedCallback;
static jmethodID method_deviceFoundCallback;
static jmethodID method_pinRequestCallback;
static jmethodID method_sspRequestCallback;
static jmethodID method_bondStateChangeCallback;
static jmethodID method_aclStateChangeCallback;
static jmethodID method_discoveryStateChangeCallback;
static jmethodID method_setWakeAlarm;
static jmethodID method_acquireWakeLock;
static jmethodID method_releaseWakeLock;
static jmethodID method_energyInfo;
...
static void adapter_state_change_callback(bt_state_t status) {
  CallbackEnv sCallbackEnv(__func__);
  if (!sCallbackEnv.valid()) return;
  ALOGV("%s: Status is: %d", __func__, status);

  sCallbackEnv->CallVoidMethod(sJniCallbacksObj, method_stateChangeCallback,
                               (jint)status);
}
...
static void classInitNative(JNIEnv* env, jclass clazz) {
  jclass jniUidTrafficClass = env->FindClass("android/bluetooth/UidTraffic");
  android_bluetooth_UidTraffic.constructor =
      env->GetMethodID(jniUidTrafficClass, "<init>", "(IJJ)V");

  jclass jniCallbackClass =
      env->FindClass("com/android/bluetooth/btservice/JniCallbacks");
  sJniCallbacksField = env->GetFieldID(
      clazz, "mJniCallbacks", "Lcom/android/bluetooth/btservice/JniCallbacks;");

  method_stateChangeCallback =
      env->GetMethodID(jniCallbackClass, "stateChangeCallback", "(I)V");

  method_adapterPropertyChangedCallback = env->GetMethodID(
      jniCallbackClass, "adapterPropertyChangedCallback", "([I[[B)V");
  method_discoveryStateChangeCallback = env->GetMethodID(
      jniCallbackClass, "discoveryStateChangeCallback", "(I)V");

  method_devicePropertyChangedCallback = env->GetMethodID(
      jniCallbackClass, "devicePropertyChangedCallback", "([B[I[[B)V");
  method_deviceFoundCallback =
      env->GetMethodID(jniCallbackClass, "deviceFoundCallback", "([B)V");
  method_pinRequestCallback =
      env->GetMethodID(jniCallbackClass, "pinRequestCallback", "([B[BIZ)V");
  method_sspRequestCallback =
      env->GetMethodID(jniCallbackClass, "sspRequestCallback", "([B[BIII)V");

  method_bondStateChangeCallback =
      env->GetMethodID(jniCallbackClass, "bondStateChangeCallback", "(I[BI)V");

  method_aclStateChangeCallback =
      env->GetMethodID(jniCallbackClass, "aclStateChangeCallback", "(I[BI)V");

  method_setWakeAlarm = env->GetMethodID(clazz, "setWakeAlarm", "(JZ)Z");
  method_acquireWakeLock =
      env->GetMethodID(clazz, "acquireWakeLock", "(Ljava/lang/String;)Z");
  method_releaseWakeLock =
      env->GetMethodID(clazz, "releaseWakeLock", "(Ljava/lang/String;)Z");
  method_energyInfo = env->GetMethodID(
      clazz, "energyInfoCallback", "(IIJJJJ[Landroid/bluetooth/UidTraffic;)V");

  if (hal_util_load_bt_library((bt_interface_t const**)&sBluetoothInterface)) {
    ALOGE("No Bluetooth Library found");
  }
}
...
static JNINativeMethod sMethods[] = {
    /* name, signature, funcPtr */
    {"classInitNative", "()V", (void*)classInitNative},
    {"initNative", "()Z", (void*)initNative},
    {"cleanupNative", "()V", (void*)cleanupNative},
    {"enableNative", "(Z)Z", (void*)enableNative},
    {"disableNative", "()Z", (void*)disableNative},
    {"setAdapterPropertyNative", "(I[B)Z", (void*)setAdapterPropertyNative},
    {"getAdapterPropertiesNative", "()Z", (void*)getAdapterPropertiesNative},
    {"getAdapterPropertyNative", "(I)Z", (void*)getAdapterPropertyNative},
    {"getDevicePropertyNative", "([BI)Z", (void*)getDevicePropertyNative},
    {"setDevicePropertyNative", "([BI[B)Z", (void*)setDevicePropertyNative},
    {"startDiscoveryNative", "()Z", (void*)startDiscoveryNative},
    {"cancelDiscoveryNative", "()Z", (void*)cancelDiscoveryNative},
    {"createBondNative", "([BI)Z", (void*)createBondNative},
    {"createBondOutOfBandNative", "([BILandroid/bluetooth/OobData;)Z",
     (void*)createBondOutOfBandNative},
    {"removeBondNative", "([B)Z", (void*)removeBondNative},
    {"cancelBondNative", "([B)Z", (void*)cancelBondNative},
    {"getConnectionStateNative", "([B)I", (void*)getConnectionStateNative},
    {"pinReplyNative", "([BZI[B)Z", (void*)pinReplyNative},
    {"sspReplyNative", "([BIZI)Z", (void*)sspReplyNative},
    {"getRemoteServicesNative", "([B)Z", (void*)getRemoteServicesNative},
    {"getSocketManagerNative", "()Landroid/os/IBinder;",
     (void*)getSocketManagerNative},
    {"setSystemUiUidNative", "(I)V", (void*)setSystemUiUidNative},
    {"setForegroundUserIdNative", "(I)V", (void*)setForegroundUserIdNative},
    {"alarmFiredNative", "()V", (void*)alarmFiredNative},
    {"readEnergyInfo", "()I", (void*)readEnergyInfo},
    {"dumpNative", "(Ljava/io/FileDescriptor;[Ljava/lang/String;)V",
     (void*)dumpNative},
    {"dumpMetricsNative", "()[B", (void*)dumpMetricsNative},
    {"factoryResetNative", "()Z", (void*)factoryResetNative},
    {"interopDatabaseClearNative", "()V", (void*)interopDatabaseClearNative},
    {"interopDatabaseAddNative", "(I[BI)V", (void*)interopDatabaseAddNative}};

int register_com_android_bluetooth_btservice_AdapterService(JNIEnv* env) {
  return jniRegisterNativeMethods(
      env, "com/android/bluetooth/btservice/AdapterService", sMethods,
      NELEM(sMethods));
}
...
/*
 * JNI Initialization
 */
jint JNI_OnLoad(JavaVM* jvm, void* reserved) {
    JNIEnv* e;
    int status;

    ALOGV("Bluetooth Adapter Service : loading JNI\n");
    ...
    status = android::register_com_android_bluetooth_btservice_AdapterService(e);
    if (status < 0) {
    ALOGE("jni adapter service registration failure, status: %d", status);
    return JNI_ERR;
    }
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterApp.java
```java
public class AdapterApp extends Application {
    ...
    static {
        if (DBG) {
            Log.d(TAG, "Loading JNI Library");
        }
        System.loadLibrary("bluetooth_jni");
    }
    ...
    @Override
    public void onCreate() {
        super.onCreate();
        if (DBG) {
            Log.d(TAG, "onCreate");
        }
        Config.init(this);
    }
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\btservice\Config.java
```java
public class Config {
    ...
    private static class ProfileConfig {
        Class mClass;
        int mSupported;
        long mMask;

        ProfileConfig(Class theClass, int supportedFlag, long mask) {
            mClass = theClass;
            mSupported = supportedFlag;
            mMask = mask;
        }
    }
    ...
    /**
     * List of profile services with the profile-supported resource flag and bit mask.
     */
    private static final ProfileConfig[] PROFILE_SERVICES_AND_FLAGS = {
            new ProfileConfig(HeadsetService.class, R.bool.profile_supported_hs_hfp,
                    (1 << BluetoothProfile.HEADSET)),
            new ProfileConfig(A2dpService.class, R.bool.profile_supported_a2dp,
                    (1 << BluetoothProfile.A2DP)),
            new ProfileConfig(A2dpSinkService.class, R.bool.profile_supported_a2dp_sink,
                    (1 << BluetoothProfile.A2DP_SINK)),
            new ProfileConfig(HidHostService.class, R.bool.profile_supported_hid_host,
                    (1 << BluetoothProfile.HID_HOST)),
            new ProfileConfig(HealthService.class, R.bool.profile_supported_hdp,
                    (1 << BluetoothProfile.HEALTH)),
            new ProfileConfig(PanService.class, R.bool.profile_supported_pan,
                    (1 << BluetoothProfile.PAN)),
            new ProfileConfig(GattService.class, R.bool.profile_supported_gatt,
                    (1 << BluetoothProfile.GATT)),
            new ProfileConfig(BluetoothMapService.class, R.bool.profile_supported_map,
                    (1 << BluetoothProfile.MAP)),
            new ProfileConfig(HeadsetClientService.class, R.bool.profile_supported_hfpclient,
                    (1 << BluetoothProfile.HEADSET_CLIENT)),
            new ProfileConfig(AvrcpTargetService.class, R.bool.profile_supported_avrcp_target,
                    (1 << BluetoothProfile.AVRCP)),
            new ProfileConfig(AvrcpControllerService.class,
                    R.bool.profile_supported_avrcp_controller,
                    (1 << BluetoothProfile.AVRCP_CONTROLLER)),
            new ProfileConfig(SapService.class, R.bool.profile_supported_sap,
                    (1 << BluetoothProfile.SAP)),
            new ProfileConfig(PbapClientService.class, R.bool.profile_supported_pbapclient,
                    (1 << BluetoothProfile.PBAP_CLIENT)),
            new ProfileConfig(MapClientService.class, R.bool.profile_supported_mapmce,
                    (1 << BluetoothProfile.MAP_CLIENT)),
            new ProfileConfig(HidDeviceService.class, R.bool.profile_supported_hid_device,
                    (1 << BluetoothProfile.HID_DEVICE)),
            new ProfileConfig(BluetoothOppService.class, R.bool.profile_supported_opp,
                    (1 << BluetoothProfile.OPP)),
            new ProfileConfig(BluetoothPbapService.class, R.bool.profile_supported_pbap,
                    (1 << BluetoothProfile.PBAP)),
            new ProfileConfig(HearingAidService.class, R.bool.profile_supported_hearing_aid,
                    (1 << BluetoothProfile.HEARING_AID))
    };
    ...
    static void init(Context ctx) {
        if (ctx == null) {
            return;
        }
        Resources resources = ctx.getResources();
        if (resources == null) {
            return;
        }

        ArrayList<Class> profiles = new ArrayList<>(PROFILE_SERVICES_AND_FLAGS.length);
        for (ProfileConfig config : PROFILE_SERVICES_AND_FLAGS) {
            boolean supported = resources.getBoolean(config.mSupported);
            if (supported && !isProfileDisabled(ctx, config.mMask)) {
                Log.v(TAG, "Adding " + config.mClass.getSimpleName());
                profiles.add(config.mClass);
            }
            sSupportedProfiles = profiles.toArray(new Class[profiles.size()]);
        }
    }
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\btservice\AdapterService.java
```java
public class AdapterService extends Service {
    ...
    static {
        classInitNative();
    }
    ...
    @Override
    public void onCreate() {
        super.onCreate();
        debugLog("onCreate()");
        mRemoteDevices = new RemoteDevices(this, Looper.getMainLooper());
        mRemoteDevices.init();
        mBinder = new AdapterServiceBinder(this);
        mAdapterProperties = new AdapterProperties(this);
        mAdapterStateMachine = AdapterState.make(this);
        mJniCallbacks = new JniCallbacks(this, mAdapterProperties);
        initNative();
        mNativeAvailable = true;
        mCallbacks = new RemoteCallbackList<IBluetoothCallback>();
        ...
    }
    ...
        /**
     * Handlers for incoming service calls
     */
    private AdapterServiceBinder mBinder;

    /**
     * The Binder implementation must be declared to be a static class, with
     * the AdapterService instance passed in the constructor. Furthermore,
     * when the AdapterService shuts down, the reference to the AdapterService
     * must be explicitly removed.
     *
     * Otherwise, a memory leak can occur from repeated starting/stopping the
     * service...Please refer to android.os.Binder for further details on
     * why an inner instance class should be avoided.
     *
     */
    private static class AdapterServiceBinder extends IBluetooth.Stub {
        private AdapterService mService;

        AdapterServiceBinder(AdapterService svc) {
            mService = svc;
        }
        ...
    }
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\btservice\ProfileService.java
```java
public abstract class ProfileService extends Service {
    ...
    public interface IProfileServiceBinder extends IBinder {
        /**
         * Called in {@link #onDestroy()}
         */
        void cleanup();
    }
    ...
    /**
     * Called in {@link #onCreate()} to init binder interface for this profile service
     *
     * @return initialized binder interface for this profile service
     */
    protected abstract IProfileServiceBinder initBinder();

    /**
     * Called in {@link #onCreate()} to init basic stuff in this service
     */
    protected void create() {}

    /**
     * Called in {@link #onStartCommand(Intent, int, int)} when the service is started by intent
     *
     * @return True in successful condition, False otherwise
     */
    protected abstract boolean start();

    /**
     * Called in {@link #onStartCommand(Intent, int, int)} when the service is stopped by intent
     *
     * @return True in successful condition, False otherwise
     */
    protected abstract boolean stop();

    /**
     * Called in {@link #onDestroy()} when this object is completely discarded
     */
    protected void cleanup() {}
    ...
        @Override
    public void onCreate() {
        if (DBG) {
            Log.d(mName, "onCreate");
        }
        super.onCreate();
        mAdapter = BluetoothAdapter.getDefaultAdapter();
        mBinder = initBinder();
        create();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        if (DBG) {
            Log.d(mName, "onStartCommand()");
        }

        if (checkCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM)
                != PackageManager.PERMISSION_GRANTED) {
            Log.e(mName, "Permission denied!");
            return PROFILE_SERVICE_MODE;
        }

        if (intent == null) {
            Log.d(mName, "onStartCommand ignoring null intent.");
            return PROFILE_SERVICE_MODE;
        }

        String action = intent.getStringExtra(AdapterService.EXTRA_ACTION);
        if (AdapterService.ACTION_SERVICE_STATE_CHANGED.equals(action)) {
            int state = intent.getIntExtra(BluetoothAdapter.EXTRA_STATE, BluetoothAdapter.ERROR);
            if (state == BluetoothAdapter.STATE_OFF) {
                doStop();
            } else if (state == BluetoothAdapter.STATE_ON) {
                doStart();
            }
        }
        return PROFILE_SERVICE_MODE;
    }

    @Override
    public IBinder onBind(Intent intent) {
        if (DBG) {
            Log.d(mName, "onBind");
        }
        if (mAdapter != null && mBinder == null) {
            // initBinder returned null, you can't bind
            throw new UnsupportedOperationException("Cannot bind to " + mName);
        }
        return mBinder;
    }

    @Override
    public boolean onUnbind(Intent intent) {
        if (DBG) {
            Log.d(mName, "onUnbind");
        }
        return super.onUnbind(intent);
    }
    ...
}
```
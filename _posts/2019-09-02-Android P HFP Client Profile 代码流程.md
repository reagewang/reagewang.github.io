---
layout: post
title: "Android P HFP Client Profile 代码流程"
tags: [bluetooth,source code,android p,hfp client,profile]
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
    status = android::register_com_android_bluetooth_hfpclient(e);
    if (status < 0) {
        ALOGE("jni hfp client registration failure, status: %d", status);
        return JNI_ERR;
    }
    ...
}
...
```

\system\bt\include\hardware\bt_hf_client.h
```java
    ...
/** BT-HF callback structure. */
typedef struct {
    /** set to sizeof(BtHfClientCallbacks) */
    size_t size;
    bthf_client_connection_state_callback connection_state_cb;
    bthf_client_audio_state_callback audio_state_cb;
    bthf_client_vr_cmd_callback vr_cmd_cb;
    bthf_client_network_state_callback network_state_cb;
    bthf_client_network_roaming_callback network_roaming_cb;
    bthf_client_network_signal_callback network_signal_cb;
    bthf_client_battery_level_callback battery_level_cb;
    bthf_client_current_operator_callback current_operator_cb;
    bthf_client_call_callback call_cb;
    bthf_client_callsetup_callback callsetup_cb;
    bthf_client_callheld_callback callheld_cb;
    bthf_client_resp_and_hold_callback resp_and_hold_cb;
    bthf_client_clip_callback clip_cb;
    bthf_client_call_waiting_callback call_waiting_cb;
    bthf_client_current_calls current_calls_cb;
    bthf_client_volume_change_callback volume_change_cb;
    bthf_client_cmd_complete_callback cmd_complete_cb;
    bthf_client_subscriber_info_callback subscriber_info_cb;
    bthf_client_in_band_ring_tone_callback in_band_ring_tone_cb;
    bthf_client_last_voice_tag_number_callback last_voice_tag_number_callback;
    bthf_client_ring_indication_callback ring_indication_cb;
} bthf_client_callbacks_t;

/** Represents the standard BT-HF interface. */
typedef struct {
  /** set to sizeof(BtHfClientInterface) */
  size_t size;
  /**
   * Register the BtHf callbacks
   */
  bt_status_t (*init)(bthf_client_callbacks_t* callbacks);

  /** connect to audio gateway */
  bt_status_t (*connect)(RawAddress* bd_addr);

  /** disconnect from audio gateway */
  bt_status_t (*disconnect)(const RawAddress* bd_addr);

  /** create an audio connection */
  bt_status_t (*connect_audio)(const RawAddress* bd_addr);

  /** close the audio connection */
  bt_status_t (*disconnect_audio)(const RawAddress* bd_addr);

  /** start voice recognition */
  bt_status_t (*start_voice_recognition)(const RawAddress* bd_addr);

  /** stop voice recognition */
  bt_status_t (*stop_voice_recognition)(const RawAddress* bd_addr);

  /** volume control */
  bt_status_t (*volume_control)(const RawAddress* bd_addr,
                                bthf_client_volume_type_t type, int volume);

  /** place a call with number a number
   * if number is NULL last called number is called (aka re-dial)*/
  bt_status_t (*dial)(const RawAddress* bd_addr, const char* number);

  /** place a call with number specified by location (speed dial) */
  bt_status_t (*dial_memory)(const RawAddress* bd_addr, int location);

  /** perform specified call related action
   * idx is limited only for enhanced call control related action
   */
  bt_status_t (*handle_call_action)(const RawAddress* bd_addr,
                                    bthf_client_call_action_t action, int idx);

  /** query list of current calls */
  bt_status_t (*query_current_calls)(const RawAddress* bd_addr);

  /** query name of current selected operator */
  bt_status_t (*query_current_operator_name)(const RawAddress* bd_addr);

  /** Retrieve subscriber information */
  bt_status_t (*retrieve_subscriber_info)(const RawAddress* bd_addr);

  /** Send DTMF code*/
  bt_status_t (*send_dtmf)(const RawAddress* bd_addr, char code);

  /** Request a phone number from AG corresponding to last voice tag recorded */
  bt_status_t (*request_last_voice_tag_number)(const RawAddress* bd_addr);

  /** Closes the interface. */
  void (*cleanup)(void);

  /** Send AT Command. */
  bt_status_t (*send_at_cmd)(const RawAddress* bd_addr, int cmd, int val1,
                             int val2, const char* arg);
} bthf_client_interface_t;
    ...
```

\packages\apps\Bluetooth\jni\com_android_bluetooth_hfpclient.cpp
```java
...
static bthf_client_interface_t* sBluetoothHfpClientInterface = NULL;
...
static jmethodID method_onConnectionStateChanged;
...
static void connection_state_cb(const RawAddress* bd_addr,
                                bthf_client_connection_state_t state,
                                unsigned int peer_feat,
                                unsigned int chld_feat) {
  CallbackEnv sCallbackEnv(__func__);
  if (!sCallbackEnv.valid()) return;

  ScopedLocalRef<jbyteArray> addr(sCallbackEnv.get(), marshall_bda(bd_addr));
  if (!addr.get()) return;

  ALOGD("%s: state %d peer_feat %d chld_feat %d", __func__, state, peer_feat, chld_feat);
  sCallbackEnv->CallVoidMethod(mCallbacksObj, method_onConnectionStateChanged,
                               (jint)state, (jint)peer_feat, (jint)chld_feat,
                               addr.get());
}
...
static bthf_client_callbacks_t sBluetoothHfpClientCallbacks = {
    sizeof(sBluetoothHfpClientCallbacks),
    connection_state_cb,
    audio_state_cb,
    vr_cmd_cb,
    network_state_cb,
    network_roaming_cb,
    network_signal_cb,
    battery_level_cb,
    current_operator_cb,
    call_cb,
    callsetup_cb,
    callheld_cb,
    resp_and_hold_cb,
    clip_cb,
    call_waiting_cb,
    current_calls_cb,
    volume_change_cb,
    cmd_complete_cb,
    subscriber_info_cb,
    in_band_ring_cb,
    last_voice_tag_number_cb,
    ring_indication_cb,
};
...
static void classInitNative(JNIEnv* env, jclass clazz) {
  method_onConnectionStateChanged =
      env->GetMethodID(clazz, "onConnectionStateChanged", "(III[B)V");
  method_onAudioStateChanged =
      env->GetMethodID(clazz, "onAudioStateChanged", "(I[B)V");
  method_onVrStateChanged =
      env->GetMethodID(clazz, "onVrStateChanged", "(I[B)V");
  method_onNetworkState = env->GetMethodID(clazz, "onNetworkState", "(I[B)V");
  method_onNetworkRoaming = env->GetMethodID(clazz, "onNetworkRoaming", "(I[B)V");
  method_onNetworkSignal = env->GetMethodID(clazz, "onNetworkSignal", "(I[B)V");
  method_onBatteryLevel = env->GetMethodID(clazz, "onBatteryLevel", "(I[B)V");
  method_onCurrentOperator =
      env->GetMethodID(clazz, "onCurrentOperator", "(Ljava/lang/String;[B)V");
  method_onCall = env->GetMethodID(clazz, "onCall", "(I[B)V");
  method_onCallSetup = env->GetMethodID(clazz, "onCallSetup", "(I[B)V");
  method_onCallHeld = env->GetMethodID(clazz, "onCallHeld", "(I[B)V");
  method_onRespAndHold = env->GetMethodID(clazz, "onRespAndHold", "(I[B)V");
  method_onClip = env->GetMethodID(clazz, "onClip", "(Ljava/lang/String;[B)V");
  method_onCallWaiting =
      env->GetMethodID(clazz, "onCallWaiting", "(Ljava/lang/String;[B)V");
  method_onCurrentCalls =
      env->GetMethodID(clazz, "onCurrentCalls", "(IIIILjava/lang/String;[B)V");
  method_onVolumeChange = env->GetMethodID(clazz, "onVolumeChange", "(II[B)V");
  method_onCmdResult = env->GetMethodID(clazz, "onCmdResult", "(II[B)V");
  method_onSubscriberInfo =
      env->GetMethodID(clazz, "onSubscriberInfo", "(Ljava/lang/String;I[B)V");
  method_onInBandRing = env->GetMethodID(clazz, "onInBandRing", "(I[B)V");
  method_onLastVoiceTagNumber =
      env->GetMethodID(clazz, "onLastVoiceTagNumber", "(Ljava/lang/String;[B)V");
  method_onRingIndication = env->GetMethodID(clazz, "onRingIndication", "([B)V");

  ALOGI("%s succeeds", __func__);
}

static void initializeNative(JNIEnv* env, jobject object) {
  ALOGD("%s: HfpClient", __func__);
  const bt_interface_t* btInf = getBluetoothInterface();
  if (btInf == NULL) {
    ALOGE("Bluetooth module is not loaded");
    return;
  }

  if (sBluetoothHfpClientInterface != NULL) {
    ALOGW("Cleaning up Bluetooth HFP Client Interface before initializing");
    sBluetoothHfpClientInterface->cleanup();
    sBluetoothHfpClientInterface = NULL;
  }

  if (mCallbacksObj != NULL) {
    ALOGW("Cleaning up Bluetooth HFP Client callback object");
    env->DeleteGlobalRef(mCallbacksObj);
    mCallbacksObj = NULL;
  }

  sBluetoothHfpClientInterface =
      (bthf_client_interface_t*)btInf->get_profile_interface(
          BT_PROFILE_HANDSFREE_CLIENT_ID);
  if (sBluetoothHfpClientInterface == NULL) {
    ALOGE("Failed to get Bluetooth HFP Client Interface");
    return;
  }

  bt_status_t status =
      sBluetoothHfpClientInterface->init(&sBluetoothHfpClientCallbacks);
  if (status != BT_STATUS_SUCCESS) {
    ALOGE("Failed to initialize Bluetooth HFP Client, status: %d", status);
    sBluetoothHfpClientInterface = NULL;
    return;
  }

  mCallbacksObj = env->NewGlobalRef(object);
}
...
static JNINativeMethod sMethods[] = {
    {"classInitNative", "()V", (void*)classInitNative},
    {"initializeNative", "()V", (void*)initializeNative},
    {"cleanupNative", "()V", (void*)cleanupNative},
    {"connectNative", "([B)Z", (void*)connectNative},
    {"disconnectNative", "([B)Z", (void*)disconnectNative},
    {"connectAudioNative", "([B)Z", (void*)connectAudioNative},
    {"disconnectAudioNative", "([B)Z", (void*)disconnectAudioNative},
    {"startVoiceRecognitionNative", "([B)Z",
     (void*)startVoiceRecognitionNative},
    {"stopVoiceRecognitionNative", "([B)Z", (void*)stopVoiceRecognitionNative},
    {"setVolumeNative", "([BII)Z", (void*)setVolumeNative},
    {"dialNative", "([BLjava/lang/String;)Z", (void*)dialNative},
    {"dialMemoryNative", "([BI)Z", (void*)dialMemoryNative},
    {"handleCallActionNative", "([BII)Z", (void*)handleCallActionNative},
    {"queryCurrentCallsNative", "([B)Z", (void*)queryCurrentCallsNative},
    {"queryCurrentOperatorNameNative", "([B)Z",
     (void*)queryCurrentOperatorNameNative},
    {"retrieveSubscriberInfoNative", "([B)Z",
     (void*)retrieveSubscriberInfoNative},
    {"sendDtmfNative", "([BB)Z", (void*)sendDtmfNative},
    {"requestLastVoiceTagNumberNative", "([B)Z",
     (void*)requestLastVoiceTagNumberNative},
    {"sendATCmdNative", "([BIIILjava/lang/String;)Z", (void*)sendATCmdNative},
};

int register_com_android_bluetooth_hfpclient(JNIEnv* env) {
  return jniRegisterNativeMethods(
      env, "com/android/bluetooth/hfpclient/NativeInterface",
      sMethods, NELEM(sMethods));
}
```

\packages\apps\Bluetooth\src\com\android\bluetooth\hfpclient\NativeInterface.java
```java
class NativeInterface {
    ...
    // Native methods that call into the JNI interface
    static native void classInitNative();
    ...
    // Callbacks from the native back into the java framework. All callbacks are routed via the
    // Service which will disambiguate which state machine the message should be routed through.
    private void onConnectionStateChanged(int state, int peerFeat, int chldFeat, byte[] address) {
        StackEvent event = new StackEvent(StackEvent.EVENT_TYPE_CONNECTION_STATE_CHANGED);
        event.valueInt = state;
        event.valueInt2 = peerFeat;
        event.valueInt3 = chldFeat;
        event.device = getDevice(address);
        // BluetoothAdapter.getDefaultAdapter().getRemoteDevice(Utils.getAddressStringFromByte
        // (address));
        if (DBG) {
            Log.d(TAG, "Device addr " + event.device.getAddress() + " State " + state);
        }
        HeadsetClientService service = HeadsetClientService.getHeadsetClientService();
        if (service != null) {
            service.messageFromNative(event);
        } else {
            Log.w(TAG, "Ignoring message because service not available: " + event);
        }
    }
    ...
}
```

\packages\apps\Bluetooth\src\com\android\bluetooth\hfpclient\HeadsetClientService.java
```java
/**
 * Provides Bluetooth Headset Client (HF Role) profile, as a service in the
 * Bluetooth application.
 *
 * @hide
 */
public class HeadsetClientService extends ProfileService {
    ...
    @Override
    protected synchronized boolean start() {
        if (DBG) {
            Log.d(TAG, "start()");
        }
        // Setup the JNI service
        NativeInterface.initializeNative();
        mAudioManager = (AudioManager) getSystemService(Context.AUDIO_SERVICE);
        if (mAudioManager == null) {
            Log.e(TAG, "AudioManager service doesn't exist?");
        } else {
            // start AudioManager in a known state
            mAudioManager.setParameters("hfp_enable=false");
        }

        mSmFactory = new HeadsetClientStateMachineFactory();
        mStateMachineMap.clear();

        IntentFilter filter = new IntentFilter(AudioManager.VOLUME_CHANGED_ACTION);
        registerReceiver(mBroadcastReceiver, filter);

        mNativeInterface = new NativeInterface();

        // Start the HfpClientConnectionService to create connection with telecom when HFP
        // connection is available.
        Intent startIntent = new Intent(this, HfpClientConnectionService.class);
        startService(startIntent);

        // Create the thread on which all State Machines will run
        mSmThread = new HandlerThread("HeadsetClient.SM");
        mSmThread.start();

        setHeadsetClientService(this);
        return true;
    }

    @Override
    protected synchronized boolean stop() {
        if (sHeadsetClientService == null) {
            Log.w(TAG, "stop() called without start()");
            return false;
        }
        setHeadsetClientService(null);

        unregisterReceiver(mBroadcastReceiver);

        for (Iterator<Map.Entry<BluetoothDevice, HeadsetClientStateMachine>> it =
                mStateMachineMap.entrySet().iterator(); it.hasNext(); ) {
            HeadsetClientStateMachine sm =
                    mStateMachineMap.get((BluetoothDevice) it.next().getKey());
            sm.doQuit();
            it.remove();
        }

        // Stop the HfpClientConnectionService.
        Intent stopIntent = new Intent(this, HfpClientConnectionService.class);
        stopIntent.putExtra(HFP_CLIENT_STOP_TAG, true);
        startService(stopIntent);
        mNativeInterface = null;

        // Stop the handler thread
        mSmThread.quit();
        mSmThread = null;

        NativeInterface.cleanupNative();

        return true;
    }
    ...
    /**
     * Handlers for incoming service calls
     */
    private static class BluetoothHeadsetClientBinder extends IBluetoothHeadsetClient.Stub
            implements IProfileServiceBinder {
        ...
        @Override
        public boolean connect(BluetoothDevice device) {
            HeadsetClientService service = getService();
            if (service == null) {
                return false;
            }
            return service.connect(device);
        }
        ...
    }
    ...
    // API methods
    public static synchronized HeadsetClientService getHeadsetClientService() {
        if (sHeadsetClientService == null) {
            Log.w(TAG, "getHeadsetClientService(): service is null");
            return null;
        }
        if (!sHeadsetClientService.isAvailable()) {
            Log.w(TAG, "getHeadsetClientService(): service is not available ");
            return null;
        }
        return sHeadsetClientService;
    }

    private static synchronized void setHeadsetClientService(HeadsetClientService instance) {
        if (DBG) {
            Log.d(TAG, "setHeadsetClientService(): set to: " + instance);
        }
        sHeadsetClientService = instance;
    }

    public boolean connect(BluetoothDevice device) {
        enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM, "Need BLUETOOTH ADMIN permission");
        if (DBG) {
            Log.d(TAG, "connect " + device);
        }
        HeadsetClientStateMachine sm = getStateMachine(device);
        if (sm == null) {
            Log.e(TAG, "Cannot allocate SM for device " + device);
            return false;
        }

        if (getPriority(device) == BluetoothProfile.PRIORITY_OFF) {
            Log.w(TAG, "Connection not allowed: <" + device.getAddress() + "> is PRIORITY_OFF");
            return false;
        }

        sm.sendMessage(HeadsetClientStateMachine.CONNECT, device);
        return true;
    }
    ...
    // Handle messages from native (JNI) to java
    public void messageFromNative(StackEvent stackEvent) {
        HeadsetClientStateMachine sm = getStateMachine(stackEvent.device);
        if (sm == null) {
            Log.w(TAG, "No SM found for event " + stackEvent);
        }

        sm.sendMessage(StackEvent.STACK_EVENT, stackEvent);
    }
    ...
}
```
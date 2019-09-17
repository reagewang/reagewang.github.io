---
layout: post
title: "Android P Car应用涉及蓝牙的代码流程"
tags: [bluetooth,source code,android p,car]
comments: true
---

\packages\apps\Car\Settings\src\com\android\car\settings\bluetooth\BluetoothSettingsFragment.java
```java
public class BluetoothSettingsFragment extends BaseFragment implements BluetoothCallback {
    ...
    private Switch mBluetoothSwitch;
    ...
    private LocalBluetoothAdapter mLocalAdapter;
    private LocalBluetoothManager mLocalManager;    
    ...
    private BroadcastReceiver mBroadcastReceiver = new BroadcastReceiver() {
        @Override
        public void onReceive(Context context, Intent intent) {
            if (BluetoothAdapter.ACTION_DISCOVERY_STARTED.equals(intent.getAction())) {
                ...
                mBluetoothSwitch.setChecked(true);
                ...
            } else if (BluetoothAdapter.ACTION_DISCOVERY_FINISHED.equals(intent.getAction())) {
                ...
            }
        }
    };
    ...
    @Override
    public void onActivityCreated(Bundle savedInstanceState) {
        super.onActivityCreated(savedInstanceState);
        mBluetoothSwitch = getActivity().findViewById(R.id.toggle_switch);
        ...

        mBluetoothSwitch.setOnCheckedChangeListener((v, isChecked) -> {
                if (mBluetoothSwitch.isChecked()) {
                    // bt scan was turned on at state listener, when state is on.
                    mLocalAdapter.setBluetoothEnabled(true);
                } else {
                    mLocalAdapter.stopScanning();
                    mLocalAdapter.setBluetoothEnabled(false);
                }
            });

        ...
        mLocalManager =
                LocalBluetoothManager.getInstance(getContext(), /* onInitCallback= */ null);
        if (mLocalManager == null) {
            LOG.e("Bluetooth is not supported on this device");
            return;
        }
        mLocalAdapter = mLocalManager.getBluetoothAdapter();    
    }
    ...
    @Override
    public void onStart() {
        super.onStart();
        ...
        mBluetoothSwitch.setChecked(mLocalAdapter.isEnabled());
        ...
    }
    ...
    @Override
    public void onBluetoothStateChanged(int bluetoothState) {
        switch (bluetoothState) {
            case BluetoothAdapter.STATE_OFF:
                ...
                mBluetoothSwitch.setChecked(false);
                ...
                break;
            case BluetoothAdapter.STATE_ON:
            case BluetoothAdapter.STATE_TURNING_ON:
                ...
                mBluetoothSwitch.setChecked(true);
                ...
                break;
            case BluetoothAdapter.STATE_TURNING_OFF:
                ...
                break;
        }
    }
}
```
\frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\LocalBluetoothAdapter.java
```java
public class LocalBluetoothAdapter {
    /** This class does not allow direct access to the BluetoothAdapter. */
    private final BluetoothAdapter mAdapter;
    ...
    private static LocalBluetoothAdapter sInstance;
    ...
    private LocalBluetoothAdapter(BluetoothAdapter adapter) {
        mAdapter = adapter;
    }
    ...
    /**
     * Get the singleton instance of the LocalBluetoothAdapter. If this device
     * doesn't support Bluetooth, then null will be returned. Callers must be
     * prepared to handle a null return value.
     * @return the LocalBluetoothAdapter object, or null if not supported
     */
    static synchronized LocalBluetoothAdapter getInstance() {
        if (sInstance == null) {
            BluetoothAdapter adapter = BluetoothAdapter.getDefaultAdapter();
            if (adapter != null) {
                sInstance = new LocalBluetoothAdapter(adapter);
            }
        }

        return sInstance;
    }

}
```
\frameworks\base\packages\SettingsLib\src\com\android\settingslib\bluetooth\LocalBluetoothManager.java
```java
public class LocalBluetoothManager {
    ...
    /** Singleton instance. */
    private static LocalBluetoothManager sInstance;
    ...
    private final LocalBluetoothAdapter mLocalAdapter;
    ...
    public static synchronized LocalBluetoothManager getInstance(Context context,
            BluetoothManagerCallback onInitCallback) {
        if (sInstance == null) {
            LocalBluetoothAdapter adapter = LocalBluetoothAdapter.getInstance();
            if (adapter == null) {
                return null;
            }
            // This will be around as long as this process is
            Context appContext = context.getApplicationContext();
            sInstance = new LocalBluetoothManager(adapter, appContext);
            if (onInitCallback != null) {
                onInitCallback.onBluetoothManagerInitialized(appContext, sInstance);
            }
        }

        return sInstance;
    }

    private LocalBluetoothManager(LocalBluetoothAdapter adapter, Context context) {
        ...
        mLocalAdapter = adapter;
        ...
    }

    public LocalBluetoothAdapter getBluetoothAdapter() {
        return mLocalAdapter;
    }    
    ...
    public interface BluetoothManagerCallback {
        void onBluetoothManagerInitialized(Context appContext,
                LocalBluetoothManager bluetoothManager);
    }
}
```
\frameworks\base\core\java\android\bluetooth\BluetoothAdapter.java
```java
public final class BluetoothAdapter {
    ...
    /**
     * Lazily initialized singleton. Guaranteed final after first object
     * constructed.
     */
    private static BluetoothAdapter sAdapter;
    ...
    private final IBluetoothManager mManagerService;
    private IBluetooth mService;
    ...
    /**
     * Get a handle to the default local Bluetooth adapter.
     * <p>Currently Android only supports one Bluetooth adapter, but the API
     * could be extended to support more. This will always return the default
     * adapter.
     * </p>
     *
     * @return the default local adapter, or null if Bluetooth is not supported on this hardware
     * platform
     */
    public static synchronized BluetoothAdapter getDefaultAdapter() {
        if (sAdapter == null) {
            IBinder b = ServiceManager.getService(BLUETOOTH_MANAGER_SERVICE);
            if (b != null) {
                IBluetoothManager managerService = IBluetoothManager.Stub.asInterface(b);
                sAdapter = new BluetoothAdapter(managerService);
            } else {
                Log.e(TAG, "Bluetooth binder is null");
            }
        }
        return sAdapter;
    }
    ...
    /**
     * Use {@link #getDefaultAdapter} to get the BluetoothAdapter instance.
     */
    BluetoothAdapter(IBluetoothManager managerService) {

        if (managerService == null) {
            throw new IllegalArgumentException("bluetooth manager service is null");
        }
        try {
            ...
            mService = managerService.registerAdapter(mManagerCallback);
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        } finally {
            ...
        }
        mManagerService = managerService;
        ...
    }
    ...
    /**
     * Return true if Bluetooth is currently enabled and ready for use.
     * <p>Equivalent to:
     * <code>getBluetoothState() == STATE_ON</code>
     *
     * @return true if the local adapter is turned on
     */
    @RequiresPermission(Manifest.permission.BLUETOOTH)
    public boolean isEnabled() {
        try {
            mServiceLock.readLock().lock();
            if (mService != null) {
                return mService.isEnabled();
            }
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        } finally {
            mServiceLock.readLock().unlock();
        }

        return false;
    }
    ...
    /**
     * Turn on the local Bluetooth adapter&mdash;do not use without explicit
     * user action to turn on Bluetooth.
     * <p>This powers on the underlying Bluetooth hardware, and starts all
     * Bluetooth system services.
     * <p class="caution"><strong>Bluetooth should never be enabled without
     * direct user consent</strong>. If you want to turn on Bluetooth in order
     * to create a wireless connection, you should use the {@link
     * #ACTION_REQUEST_ENABLE} Intent, which will raise a dialog that requests
     * user permission to turn on Bluetooth. The {@link #enable()} method is
     * provided only for applications that include a user interface for changing
     * system settings, such as a "power manager" app.</p>
     * <p>This is an asynchronous call: it will return immediately, and
     * clients should listen for {@link #ACTION_STATE_CHANGED}
     * to be notified of subsequent adapter state changes. If this call returns
     * true, then the adapter state will immediately transition from {@link
     * #STATE_OFF} to {@link #STATE_TURNING_ON}, and some time
     * later transition to either {@link #STATE_OFF} or {@link
     * #STATE_ON}. If this call returns false then there was an
     * immediate problem that will prevent the adapter from being turned on -
     * such as Airplane mode, or the adapter is already turned on.
     *
     * @return true to indicate adapter startup has begun, or false on immediate error
     */
    @RequiresPermission(Manifest.permission.BLUETOOTH_ADMIN)
    public boolean enable() {
        if (isEnabled()) {
            if (DBG) {
                Log.d(TAG, "enable(): BT already enabled!");
            }
            return true;
        }
        try {
            return mManagerService.enable(ActivityThread.currentPackageName());
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        }
        return false;
    }
    ...
    /**
     * Turn off the local Bluetooth adapter&mdash;do not use without explicit
     * user action to turn off Bluetooth.
     * <p>This gracefully shuts down all Bluetooth connections, stops Bluetooth
     * system services, and powers down the underlying Bluetooth hardware.
     * <p class="caution"><strong>Bluetooth should never be disabled without
     * direct user consent</strong>. The {@link #disable()} method is
     * provided only for applications that include a user interface for changing
     * system settings, such as a "power manager" app.</p>
     * <p>This is an asynchronous call: it will return immediately, and
     * clients should listen for {@link #ACTION_STATE_CHANGED}
     * to be notified of subsequent adapter state changes. If this call returns
     * true, then the adapter state will immediately transition from {@link
     * #STATE_ON} to {@link #STATE_TURNING_OFF}, and some time
     * later transition to either {@link #STATE_OFF} or {@link
     * #STATE_ON}. If this call returns false then there was an
     * immediate problem that will prevent the adapter from being turned off -
     * such as the adapter already being turned off.
     *
     * @return true to indicate adapter shutdown has begun, or false on immediate error
     */
    @RequiresPermission(Manifest.permission.BLUETOOTH_ADMIN)
    public boolean disable() {
        try {
            return mManagerService.disable(ActivityThread.currentPackageName(), true);
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        }
        return false;
    }            
    ...
    private final IBluetoothManagerCallback mManagerCallback =
        new IBluetoothManagerCallback.Stub() {
            public void onBluetoothServiceUp(IBluetooth bluetoothService) {
                ...
        
                mService = bluetoothService;

                ...
            }

            public void onBluetoothServiceDown() {
                if (DBG) {
                    Log.d(TAG, "onBluetoothServiceDown: " + mService);
                }

                try {
                    ...
                    mService = null;
                    ...
                } finally {
                    ...
                }

                ...
            }

            public void onBrEdrDown() {
                ...
            }
        };
    ...
    protected void finalize() throws Throwable {
        try {
            mManagerService.unregisterAdapter(mManagerCallback);
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        } finally {
            super.finalize();
        }
    }
    ...
}
```
\frameworks\base\services\core\java\com\android\server\BluetoothService.java
```java
class BluetoothService extends SystemService {
    private BluetoothManagerService mBluetoothManagerService;

    public BluetoothService(Context context) {
        super(context);
        mBluetoothManagerService = new BluetoothManagerService(context);
    }

    @Override
    public void onStart() {
    }

    @Override
    public void onBootPhase(int phase) {
        if (phase == SystemService.PHASE_SYSTEM_SERVICES_READY) {
            publishBinderService(BluetoothAdapter.BLUETOOTH_MANAGER_SERVICE,
                    mBluetoothManagerService);
        } else if (phase == SystemService.PHASE_ACTIVITY_MANAGER_READY) {
            mBluetoothManagerService.handleOnBootPhase();
        }
    }

    @Override
    public void onSwitchUser(int userHandle) {
        mBluetoothManagerService.handleOnSwitchUser(userHandle);
    }

    @Override
    public void onUnlockUser(int userHandle) {
        mBluetoothManagerService.handleOnUnlockUser(userHandle);
    }
}
```
\frameworks\base\services\core\java\com\android\server\BluetoothManagerService.java
```java
class BluetoothManagerService extends IBluetoothManager.Stub {
    ...
    private final IBluetoothCallback mBluetoothCallback = new IBluetoothCallback.Stub() {
        @Override
        public void onBluetoothStateChange(int prevState, int newState) throws RemoteException {
            Message msg =
                    mHandler.obtainMessage(MESSAGE_BLUETOOTH_STATE_CHANGE, prevState, newState);
            mHandler.sendMessage(msg);
        }
    };
    ...
}
```

\packages\apps\Car\Dialer\src\com\android\car\dialer\ui\CallHistoryFragment.java
\packages\apps\Car\Dialer\src\com\android\car\dialer\ui\CallHistoryListItemProvider.java
\packages\apps\Car\Dialer\src\com\android\car\dialer\ui\CallLogListingTask.java
\packages\apps\Car\Dialer\src\com\android\car\dialer\livedata\CallHistoryLiveData.java
\packages\apps\Car\Dialer\src\com\android\car\dialer\telecom\PhoneLoader.java
\frameworks\base\core\java\android\content\CursorLoader.java
\frameworks\base\core\java\android\content\Loader.java
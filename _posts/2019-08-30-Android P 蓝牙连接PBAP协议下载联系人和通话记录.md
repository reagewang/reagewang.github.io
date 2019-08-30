---
layout: post
title: "Android P 蓝牙连接PBAP协议下载联系人和通话记录"
tags: [bluetooth,source code,android p]
comments: true
---

\packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PbapClientService.java
```java
public class PbapClientService extends ProfileService {
    ...
    @Override
    public IProfileServiceBinder initBinder() {
        return new BluetoothPbapClientBinder(this);
    }
    ...
    /**
    * Handler for incoming service calls
    */
    private static class BluetoothPbapClientBinder extends IBluetoothPbapClient.Stub
        implements IProfileServiceBinder {
        ...
        @Override
            public boolean connect(BluetoothDevice device) {
                PbapClientService service = getService();
                if (DBG) {
                    Log.d(TAG, "PbapClient Binder connect ");
                }
                if (service == null) {
                    Log.e(TAG, "PbapClient Binder connect no service");
                    return false;
                }
                return service.connect(device);
            }
        ...
    }
    ...
    public boolean connect(BluetoothDevice device) {
        if (device == null) {
            throw new IllegalArgumentException("Null device");
        }
        enforceCallingOrSelfPermission(BLUETOOTH_ADMIN_PERM, "Need BLUETOOTH ADMIN permission");
        Log.d(TAG, "Received request to ConnectPBAPPhonebook " + device.getAddress());
        if (getPriority(device) <= BluetoothProfile.PRIORITY_OFF) {
            return false;
        }
        synchronized (mPbapClientStateMachineMap) {
            PbapClientStateMachine pbapClientStateMachine = mPbapClientStateMachineMap.get(device);
            if (pbapClientStateMachine == null
                    && mPbapClientStateMachineMap.size() < MAXIMUM_DEVICES) {
                pbapClientStateMachine = new PbapClientStateMachine(this, device);
                pbapClientStateMachine.start();
                mPbapClientStateMachineMap.put(device, pbapClientStateMachine);
                return true;
            } else {
                Log.w(TAG, "Received connect request while already connecting/connected.");
                return false;
            }
        }
    }
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PbapClientStateMachine.java
```java
final class PbapClientStateMachine extends StateMachine {
    ...
    class Connected extends State {
        @Override
        public void enter() {
            Log.d(TAG, "Enter Connected: " + getCurrentMessage().what);
            onConnectionStateChanged(mCurrentDevice, mMostRecentState,
                    BluetoothProfile.STATE_CONNECTED);
            mMostRecentState = BluetoothProfile.STATE_CONNECTED;
            if (mUserManager.isUserUnlocked()) {
                mConnectionHandler.obtainMessage(PbapClientConnectionHandler.MSG_DOWNLOAD)
                        .sendToTarget();
            }
        }

        @Override
        public boolean processMessage(Message message) {
            if (DBG) {
                Log.d(TAG, "Processing MSG " + message.what + " from " + this.getName());
            }
            switch (message.what) {
                case MSG_DISCONNECT:
                    if ((message.obj instanceof BluetoothDevice)
                            && ((BluetoothDevice) message.obj).equals(mCurrentDevice)) {
                        transitionTo(mDisconnecting);
                    }
                    break;

                case MSG_RESUME_DOWNLOAD:
                    mConnectionHandler.obtainMessage(PbapClientConnectionHandler.MSG_DOWNLOAD)
                            .sendToTarget();
                    break;

                default:
                    Log.w(TAG, "Received unexpected message while Connected");
                    return NOT_HANDLED;
            }
            return HANDLED;
        }
    }
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PbapClientConnectionHandler.java
```java
class PbapClientConnectionHandler extends Handler {
    ...
    @Override
    public void handleMessage(Message msg) {
        ...
        switch (msg.what) {
            ...
            case MSG_DOWNLOAD:
                try {
                    mAccountCreated = addAccount(mAccount);
                    if (!mAccountCreated) {
                        Log.e(TAG, "Account creation failed.");
                        return;
                    }
                    // Start at contact 1 to exclued Owner Card PBAP 1.1 sec 3.1.5.2
                    BluetoothPbapRequestPullPhoneBook request =
                            new BluetoothPbapRequestPullPhoneBook(PB_PATH, mAccount,
                                    PBAP_REQUESTED_FIELDS, VCARD_TYPE_30, 0, 1);
                    request.execute(mObexSession);
                    PhonebookPullRequest processor =
                            new PhonebookPullRequest(mPbapClientStateMachine.getContext(),
                                    mAccount);
                    processor.setResults(request.getList());
                    processor.onPullComplete();
                    HashMap<String, Integer> callCounter = new HashMap<>();
                    downloadCallLog(MCH_PATH, callCounter);
                    downloadCallLog(ICH_PATH, callCounter);
                    downloadCallLog(OCH_PATH, callCounter);
                } catch (IOException e) {
                    Log.w(TAG, "DOWNLOAD_CONTACTS Failure" + e.toString());
                }
                break;
            ...
        }
        ...
    }
    ...
    /* Connect an OBEX session over the already connected socket.  First establish an OBEX Transport
     * abstraction, then establish a Bluetooth Authenticator, and finally issue the connect call */
    private boolean connectObexSession() {
        boolean connectionSuccessful = false;

        try {
            if (DBG) {
                Log.v(TAG, "Start Obex Client Session");
            }
            BluetoothObexTransport transport = new BluetoothObexTransport(mSocket);
            mObexSession = new ClientSession(transport);
            mObexSession.setAuthenticator(mAuth);

            HeaderSet connectionRequest = new HeaderSet();
            connectionRequest.setHeader(HeaderSet.TARGET, PBAP_TARGET);

            if (mPseRec != null) {
                if (DBG) {
                    Log.d(TAG, "Remote PbapSupportedFeatures " + mPseRec.getSupportedFeatures());
                }

                ObexAppParameters oap = new ObexAppParameters();

                if (mPseRec.getProfileVersion() >= PBAP_V1_2) {
                    oap.add(BluetoothPbapRequest.OAP_TAGID_PBAP_SUPPORTED_FEATURES,
                            PBAP_SUPPORTED_FEATURE);
                }

                oap.addToHeaderSet(connectionRequest);
            }
            HeaderSet connectionResponse = mObexSession.connect(connectionRequest);

            connectionSuccessful =
                    (connectionResponse.getResponseCode() == ResponseCodes.OBEX_HTTP_OK);
            if (DBG) {
                Log.d(TAG, "Success = " + Boolean.toString(connectionSuccessful));
            }
        } catch (IOException e) {
            Log.w(TAG, "CONNECT Failure " + e.toString());
            closeSocket();
        }
        return connectionSuccessful;
    }
    ...
    void downloadCallLog(String path, HashMap<String, Integer> callCounter) {
        try {
            BluetoothPbapRequestPullPhoneBook request =
                    new BluetoothPbapRequestPullPhoneBook(path, mAccount, 0, VCARD_TYPE_30, 0, 0);
            request.execute(mObexSession);
            CallLogPullRequest processor =
                    new CallLogPullRequest(mPbapClientStateMachine.getContext(), path,
                        callCounter, mAccount);
            processor.setResults(request.getList());
            processor.onPullComplete();
        } catch (IOException e) {
            Log.w(TAG, "Download call log failure");
        }
    }
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\BluetoothPbapRequest.java
```java
abstract class BluetoothPbapRequest {
    ...
    public void execute(ClientSession session) throws IOException {
        ...
        ...
    }
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\BluetoothPbapRequestPullPhoneBook.java
```java
final class BluetoothPbapRequestPullPhoneBook extends BluetoothPbapRequest {
    ...
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PullRequest.java
```java
public abstract class PullRequest {
    public String path;
    protected List<VCardEntry> mEntries;

    public abstract void onPullComplete();

    @Override
    public String toString() {
        return "PullRequest: { path=" + path + " }";
    }

    public void setResults(List<VCardEntry> results) {
        mEntries = results;
    }
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PhonebookPullRequest.java
```java
public class PhonebookPullRequest extends PullRequest {
    ...
    @Override
    public void onPullComplete() {
        ...
        ...
    }
    ...
}
```
\packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\CallLogPullRequest.java
```java
public class CallLogPullRequest extends PullRequest {
    ...
    @Override
    public void onPullComplete() {
        ...
        ...
    }
    ...
}
```
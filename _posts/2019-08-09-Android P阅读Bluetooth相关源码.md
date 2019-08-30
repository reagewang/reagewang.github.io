---
layout: post
title: "Android P阅读Bluetooth相关源码"
tags: [bluetooth,source code,android p]
comments: true
---

# 蓝牙服务启动位置
> /android/frameworks/base/services/java/com/android/server/SystemService.java

```java
    /**
     * Starts a miscellaneous grab bag of stuff that has yet to be refactored
     * and organized.
     */
    private void startOtherServices() {
        ...
        // Skip Bluetooth if we have an emulator kernel
        // TODO: Use a more reliable check to see if this product should
        // support Bluetooth - see bug 988521
        if (isEmulator) {
            Slog.i(TAG, "No Bluetooth Service (emulator)");
        } else if (mFactoryTestMode == FactoryTest.FACTORY_TEST_LOW_LEVEL) {
            Slog.i(TAG, "No Bluetooth Service (factory test)");
        } else if (!context.getPackageManager().hasSystemFeature
                    (PackageManager.FEATURE_BLUETOOTH)) {
            Slog.i(TAG, "No Bluetooth Service (Bluetooth Hardware Not Present)");
        } else {
            traceBeginAndSlog("StartBluetoothService");
            mSystemServiceManager.startService(BluetoothService.class);
            traceEnd();
        }
        ...
    }
```

## package com.android.server 查找BluetoothService.class路径
> /frameworks/base/services/core/java/com/android/server/BluetoothService.java
> /frameworks/base/services/core/java/com/android/server/BluetoothManagerService.java

## package android.bluetooth
> /frameworks/base/core/java/android/bluetooth/BluetoothAdapter.java

## package android.bluetooth的aidl接口
> /system/bt/binder/android/bluetooth/IBluetoothManager.aidl

## package com.android.settingslib.bluetooth
> /frameworks/base/packages/SettingsLib/src/com/android/settingslib/bluetooth

## package com.android.bluetooth
> /packages/apps/Bluetooth

## 蓝牙同步联系人和通话记录目录以及相关类
> \packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient
> \packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PbapClientService.java
> \packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PbapClientStateMachine.java
> \packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PbapClientConnectionHandler.java
> \packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\CallLogPullRequest.java
> \packages\apps\Bluetooth\src\com\android\bluetooth\pbapclient\PhonebookPullRequest.java
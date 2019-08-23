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

## 查找BluetoothService.class路径
> /android/frameworks/base/services/core/java/com/android/server/BluetoothService.java
> /android/frameworks/base/services/core/java/com/android/server/BluetoothManagerService.java

## 查找BluetoothAdapter.java路径
> /android/frameworks/base/core/java/android/bluetooth/BluetoothAdapter.java

## 查找BluetoothManagerService继承的aidl接口
> /android/system/bt/binder/android/bluetooth/IBluetoothManager.aidl
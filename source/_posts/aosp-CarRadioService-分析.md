---
title: CarRadioService 分析
date: 2019-03-04 21:14:42
categories: "Android源码"
tags:
    - Android
    - Radio
    - CarService
---

# CarRadioService 分析

需求需要自己做个车机版 Radio app，需要 framework 提供接口，并且需要定义在 CarService 中，所以这里对 CarService 的流程做了个简单地总结，以及如何在 framework 层定义接口，最后是在模拟器中如何调试接口。

<!-- more -->

## 启动流程

步骤 0-27：

CarService.java

```java
// 1
@Override
public void onCreate() {
    mVehicle = getVehicle(null /* Any Vehicle HAL interface name */);
    // 1
    mICarImpl = new ICarImpl(this, mVehicle, SystemInterface.getDefault(this),
            mCanBusErrorNotifier);

    // 5
    mICarImpl.init();
}
```

ICarImpl.java

```java
// 2
public ICarImpl(Context serviceContext, IVehicle vehicle, SystemInterface systemInterface,
        CanBusErrorNotifier errorNotifier) {
    mHal = new VehicleHal(vehicle);
    mCarRadioService = new CarRadioService(serviceContext, mHal.getRadioHal());

    // Be careful with order. Service depending on other service should be inited later.
    List<CarServiceBase> allServices = new ArrayList<>(Arrays.asList(
            mCarRadioService
    ));
    mAllServices = allServices.toArray(new CarServiceBase[0]);
}

// 6
public void init() {
    // 7
    mHal.init();
    // 11
    for (CarServiceBase service : mAllServices) {
        service.init();
    }
}
```

VehicleHal.java

```java
// 3
public VehicleHal(IVehicle vehicle) {
    mRadioHal = new RadioHalService(this);
    mAllServices.addAll(Arrays.asList(mRadioHal));
    mHalClient = new HalClient(vehicle, mHandlerThread.getLooper(), this /*IVehicleCallback*/);
}

// 8
public void init() {
    Set<VehiclePropConfig> properties;
    properties = new HashSet<>(mHalClient.getAllPropConfigs());

    for (HalServiceBase service: mAllServices) {
        Collection<VehiclePropConfig> taken = service.takeSupportedProperties(properties);
        synchronized (this) {
            for (VehiclePropConfig p: taken) {
                mPropertyHandlers.append(p.prop, service);
            }
        }
        properties.removeAll(taken);
        // 9
        service.init();
    }
}

// 12 下面走回调流程

// public class VehicleHal extends IVehicleCallback.Stub
// 这个类接受hal层的callback，如果属性值更新了，就会向framework通知，然后在这个地方接收。

@Override
public void onPropertyEvent(ArrayList<VehiclePropValue> propValues) {
    for (VehiclePropValue v : propValues) {
        HalServiceBase service = mPropertyHandlers.get(v.prop);
        mServicesToDispatch.add(service);
    }
    for (HalServiceBase s : mServicesToDispatch) {
        // 13
        s.handleHalEvents(s.getDispatchList());
        s.getDispatchList().clear();
    }
    mServicesToDispatch.clear();
}

```

HalClient.java

```java
// 4
HalClient(IVehicle vehicle, Looper looper, IVehicleCallback callback) {
    mVehicle = vehicle;
    Handler handler = new CallbackHandler(looper, callback);
    mInternalCallback = new VehicleCallback(handler);
}
```

RadioHalService.java

```java
// 10
@Override
public synchronized void init() {
}

// 14
@Override
public void handleHalEvents(List<VehiclePropValue> values) {
    RadioHalService.RadioListener radioListener;
    radioListener = mListener;

    for (VehiclePropValue v : values) {
        // 15
        CarRadioEvent radioEvent = createCarRadioEvent(v);
        // 16
        radioListener.onEvent(radioEvent);
    }
}

// 16
private CarRadioEvent createCarRadioEvent(VehiclePropValue v) {
    switch (v.prop) {
        case VehicleProperty.RADIO_PRESET:
            int vecSize = v.value.int32Values.size();

            Integer intValues[] = new Integer[4];
            v.value.int32Values.toArray(intValues);

            CarRadioPreset preset =
                new CarRadioPreset(intValues[0], intValues[1], intValues[2], intValues[3]);
            CarRadioEvent event = new CarRadioEvent(CarRadioEvent.RADIO_PRESET, preset);
            return event;
    }
}

// 17
public synchronized void registerListener(RadioListener listener) {
    mListener = listener;

    // Subscribe to all radio properties.
    mHal.subscribeProperty(this, VehicleProperty.RADIO_PRESET);
}
```

CarRadioService.java

```java
// 12
@Override
public synchronized void init() {
}
```

CarRadioManager.java

```java
        // 27
public synchronized void registerListener(CarRadioEventListener listener)
        throws CarNotConnectedException {
        mListener = listener;
        // 19
        mListenerToService = new CarRadioEventListenerToService(this);
        // 18
        mService.registerListener(mListenerToService);

}

// 20
private static class CarRadioEventListenerToService extends ICarRadioEventListener.Stub {
    @Override
    public void onEvent(CarRadioEvent event) {
        CarRadioManager manager = mManager.get();
        // 21
        manager.handleEvent(event);
    }
}

// 22
private void handleEvent(CarRadioEvent event) {
    mHandler.sendMessage(mHandler.obtainMessage(MSG_RADIO_EVENT, event));
}

// 23
private static final class EventCallbackHandler extends Handler {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case MSG_RADIO_EVENT:
                CarRadioManager mgr = mMgr.get();
                mgr.dispatchEventToClient((CarRadioEvent) msg.obj);
    }
}

// 24
private void dispatchEventToClient(CarRadioEvent event) {
    CarRadioEventListener listener;
    // 25
    listener = mListener;
    listener.onEvent(event);

}
```

CarRadioEventListener.java

```java
// 26
public interface CarRadioEventListener {
    void onEvent(final CarRadioEvent event);
}
```

## App 调用接口

### 初始化 Car 及 CarRadioManager

```java
public Car car;
public CarRadioManager carRadioManager;

public void onCreate() {
    car = Car.createCar(this, new ServiceConnection() {
        @Override
        public void onServiceConnected(ComponentName name, IBinder service) {
            try {
                carRadioManager = (CarRadioManager) car.getCarManager(Car.RADIO_SERVICE);
            } catch (CarNotConnectedException e) {
                e.printStackTrace();
            }
        }

        @Override
        public void onServiceDisconnected(ComponentName name) {
            carRadioManager = null;
        }
    });
    car.connect();
}
```

### 接口定义 (framework层定义)

CarRadioManager.java

```java
public boolean setPreset(CarRadioPreset preset) throws IllegalArgumentException,
        CarNotConnectedException {
    return mService.setPreset(preset);
}
```

RadioHalService.java

```java
public boolean setRadioPreset(CarRadioPreset preset) {
    mHal.set(VehicleProperty.RADIO_PRESET).to(new int[] {
            preset.getPresetNumber(),
            preset.getBand(),
            preset.getChannel(),
            preset.getSubChannel()});
    return true;
}
``

VehicleHal.java

```java
@CheckResult
VehiclePropValueSetter set(int propId) {
    return new VehiclePropValueSetter(mHalClient, propId, NO_AREA);
}

void to(int[] values) throws PropertyTimeoutException {
    for (int value : values) {
        mPropValue.value.int32Values.add(value);
    }
    submit();
}

void submit() throws PropertyTimeoutException {
    HalClient client =  mClient.get();
    client.setValue(mPropValue);
}
```

HalClient.java

```java
public void setValue(VehiclePropValue propValue) throws PropertyTimeoutException {
    int status = invokeRetriable(() -> {
        try {
            return mVehicle.set(propValue);
        } catch (RemoteException e) {
            Log.e(CarLog.TAG_HAL, "Failed to set value", e);
            return StatusCode.TRY_AGAIN;
        }
    }, WAIT_CAP_FOR_RETRIABLE_RESULT_MS, SLEEP_BETWEEN_RETRIABLE_INVOKES_MS);

    if (StatusCode.INVALID_ARG == status) {
        throw new IllegalArgumentException(
                String.format("Failed to set value for: 0x%x, areaId: 0x%x",
                        propValue.prop, propValue.areaId));
    }

    if (StatusCode.TRY_AGAIN == status) {
        throw new PropertyTimeoutException(propValue.prop);
    }

    if (StatusCode.OK != status) {
        throw new IllegalStateException(
                String.format("Failed to set property: 0x%x, areaId: 0x%x, "
                        + "code: %d", propValue.prop, propValue.areaId, status));
    }
}
```

hardware:
<!-- android.hardware.automotive.vehicle.V2_0.IVehicle -->

## 模拟器如何做调试

1. 编译 Car 版本的 harware 源码，及 CarService Apk
注：Genymotion pixel xl 对应的是 marlin 版本， 编译出来是64位不能用，所以需要编译 x86 版本，为32位的

2. 将编译生成的三个库文件导入模拟器相应位置。

Z:\SourceCode\android-8.0.0_r1\out\target\product\generic_x86\system\vendor\etc\init
android.hardware.automotive.vehicle@2.0-service.rc

Z:\SourceCode\android-8.0.0_r1\out\target\product\generic_x86\system\vendor\bin\hw
android.hardware.automotive.vehicle@2.0-service

Z:\SourceCode\android-8.0.0_r1\out\target\product\generic_x86\system\lib
android.hardware.automotive.vehicle@2.0.so

3. 安装 CarService apk

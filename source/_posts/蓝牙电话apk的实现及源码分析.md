---
title: 蓝牙电话apk的实现及源码分析
date: 2019-03-20 20:50:51
categories: "Android"
tags:
     - Bluetooth
     - Android
     - 蓝牙
---

之前介绍了[蓝牙基本功能实现（开启，扫描，配对，连接等）](https://mcl-123.github.io/2019/03/16/%E8%93%9D%E7%89%99%E5%9F%BA%E6%9C%AC%E5%8A%9F%E8%83%BD%E5%AE%9E%E7%8E%B0/), 在这里继续介绍蓝牙电话 apk 的实现，及相关源码的分析。

<!-- more -->

# 蓝牙电话apk实现

只需要两步：

1. 注册 BluetoothHeadsetClientCall 相关广播监听来电/接通/挂断的状态，获取蓝牙电话的相关信息，比如号码、设备信息等。

```java
BroadcastReceiver broadcastReceiver = new BroadcastReceiver(){
      @Override
      public void onReceive(Context arg0, Intent intent) {
        // hide掉了
        if(intent.getAction().equals(BluetoothHeadsetClient.ACTION_CONNECTION_STATE_CHANGED)){// 连接远程设备广播 
             // 0:disconneted 1:connecting 2:connected 3:disconnecting
             int state = intent.getIntExtra(BluetoothProfile.EXTRA_STATE, -1);
             Log.e(TAG,"ACTION_CONNECTION_STATE_CHANGED:: state = " + String.valueOf(state) + "\n"); 
        } else if(intent.getAction().equals(BluetoothHeadsetClient.ACTION_CALL_CHANGED)){// 接通状态广播：来电、去电、挂断等
             Log.e(TAG," action = BluetoothHeadsetClient.ACTION_CALL_CHANGED");
             Object state = intent.getExtra(BluetoothHeadsetClient.EXTRA_CALL);// 注：getExtra()返回Object类型
             BluetoothHeadsetClientCall ss = (BluetoothHeadsetClientCall)state;// 类型转换
             Log.e(TAG," Object = " + String.valueOf(state));
             Log.e(TAG,"ss.mId = " + String.valueOf(ss.getId()));
             /*state:: CALL_STATE_ACTIVE =0; CALL_STATE_HELD = 1; CALL_STATE_DIALING =2*/
             Log.e(TAG,"ss.mState = " + String.valueOf(ss.getState()));// 获取来电/去电/接通/挂断等状态
             Log.e(TAG,"ss.mNumber = " + String.valueOf(ss.getNumber()));// 获取来电/去电的电话号码
             Log.e(TAG," ss.mOutgoing = " + String.valueOf(ss.isOutgoing()));
             Log.e(TAG,"ss.mMultiParty = " + String.valueOf(ss.isMultiParty()));
        } else if(intent.getAction().equals(BluetoothHeadsetClient.ACTION_AUDIO_STATE_CHANGED)){// 音频流接通或断开广播
             Log.e(TAG," action = BluetoothHeadsetClient.ACTION_AUDIO_STATE_CHANGED");
             // state: 0:audio disconnected; 1:conneting; 2:connected
             int state = intent.getIntExtra(BluetoothProfile.EXTRA_STATE, -1); 
             Log.e(TAG,"ACTION_AUDIO_STATE_CHANGED:: state = " + String.valueOf(state));  
        }
    }
};

public void Register(){
    IntentFilter filter = new IntentFilter();
    filter.addAction(BluetoothHeadsetClient.ACTION_CONNECTION_STATE_CHANGED);
    filter.addAction(BluetoothHeadsetClient.ACTION_CALL_CHANGED);
    filter.addAction(BluetoothHeadsetClient.ACTION_AUDIO_STATE_CHANGED);
    registerReceiver(broadcastReceiver, filter);
}
```

2. 通过 BluetoothAdapter 获取并且初始化 BluetoothHeadsetClient 对象，然后就可以调用 api：
    dial()/acceptCall()/rejectCall()/terminateCall()方法进行拨号/接通/拒接的操作了。

```java
bluetoothAdapter.getProfileProxy(MainActivity.this, new BluetoothProfile.ServiceListener() {
    @Override
    public void onServiceConnected(int profile, BluetoothProfile proxy) {
        // 连接后，拿到蓝牙电话客户端的代理 mHeadsetClient 
        // Api 接口： IBluetoothHeadsetClient.aidl
        mHeadsetClient = (BluetoothHeadsetClient) proxy;
    }
    @Override
    public void onServiceDisconnected(int profile) {
    }
}, BluetoothProfile.HEADSET);

// 给110打电话
mHeadsetClient.dial(mDevice, "110")
```

# 蓝牙电话 Api 相关源码分析

路径1: frameworks\base\core\java\android\bluetooth\  

蓝牙相关接口,蓝牙各种功能的发起点。

路径2:packages\apps\Bluetooth\src\com\android\bluetooth\  

独立的Bluetooth.apk,里面包含蓝牙相关的各种服务,是java层和C/C++层的桥梁。

路径3: packages\apps\Bluetooth\jni\

调用底层C/C++实现各种蓝牙功能,并且反馈给java层。

## 蓝牙协议

- a2dp: 和蓝牙耳机,音频有关,比如听歌等。

- avrcp: 音频/视频通过连接的蓝牙控制,比如放歌时控制暂停等。

- gatt：低功耗BLE有关,比如蓝牙按键。

- hdp: 蓝牙医疗有关

- hfp和hfpclient : 蓝牙通话有关,比如蓝牙通话的相关操作

- hid: 蓝牙键盘键盘/鼠标

- map: 同步蓝牙短信相关

- opp: 蓝牙传输,比如传输文件等

- pan: 个人局域网

- pbap: 同步电话本,比如联系人/通话记录等

- sap : 蓝牙通话,主要和SIM卡相关

- sdp: 蓝牙服务发现/获取相关

这12个包分别实现了12中蓝牙功能,大多数以服务的形式存在,运行在 Bluetooth.apk 中。不仅如此,还具有以下特点:

1,每一个服务相互独立,互相毫无任何影响, 继承自 ProfileService,由 AdapterService 服务统一管理。

2,每一个服务在路径1中都存在对应的客户端类,通过Binder进行跨进程通信。

3,每一个服务在路径3中都存在对应的C/C++类,通过JNI机制互相调用。

4,每一个服务的启动,对应的Binder以及JNI机制的调用原理,方法,流程几乎都是一样的。

## 通过蓝牙获取通讯录及通话记录

路径：android-8.0.0_r1\frameworks\base\core\java\android\bluetooth

Pbap 协议：通过 Pbap(Phone Book Access Profile) 协议同步联系人/通话记录

BluetoothPbapClient.java:

该类是 @hide，可以通过修改源码重新编译，或者通过反射的方式获取此类的对象，，但应该是尚不成熟的，所以在此仅做简单学习用吧。

注：使用反射时，我的一加6手机发现该类的构造函数不存在，应该是被阉割了。在 Hikey970 板子上测试是ok的。

```java
// Create a BluetoothPbapClient proxy object.
BluetoothPbapClient(Context context, ServiceListener l) {
    mContext = context;
    mServiceListener = l;
    mAdapter = BluetoothAdapter.getDefaultAdapter();
    IBluetoothManager mgr = mAdapter.getBluetoothManager();
    if (mgr != null) {
        try {
            mgr.registerStateChangeCallback(mBluetoothStateChangeCallback);
        } catch (RemoteException e) {
            Log.e(TAG, "", e);
        }
    }
    doBind();
}

// Upon successful connection to remote PBAP server the Client will attempt to automatically download the users phonebook and call log
public boolean connect(BluetoothDevice device) {
    final IBluetoothPbapClient service = mService;
    if (service != null && isEnabled() && isValidDevice(device)) {
        try {
            return service.connect(device);
        } catch (RemoteException e) {
            Log.e(TAG, Log.getStackTraceString(new Throwable()));
            return false;
        }
    }
    return false;
}
```

## 蓝牙电话API调用

主要类：

BluetoothHeadsetClient.java 主要负责蓝牙通话的相关动作,比如接听等等

BluetoothHeadsetClientCall.java 主要负责蓝牙通话的状态,比如是来电还是去电等等。

HeadsetClientHalConstants.java 类里面只是定义了一些 int/boolean 类型的值。

HeadsetClientService.java 从名字就知道它是一个服务,里面还有一个BluetoothHeadsetClientBinder内部类,该内部类主要负责和 BluetoothHeadsetClient 进行跨进程通信。另外, HeadsetClientService 也是 BluetoothHeadsetClientBinder 和 HeadsetClientStateMachine 之间的桥梁。

HeadsetClientStateMachine 是一个状态机,即管理连接的状态也是通话时 java 和 C/C++ 之间的桥梁,通过 JNI 机制和 com_android_bluetooth_hfpclient 里面的方法互相调用。

com_android_bluetooth_hfpclient  蓝牙通话动作,拨号/接听/挂断/拒接 实际的执行者。

### 打电话

```java
// 设置蓝牙电话客户端监听
bluetoothAdapter.getProfileProxy(MainActivity.this, new BluetoothProfile.ServiceListener() {
    @Override
    public void onServiceConnected(int profile, BluetoothProfile proxy) {
        // 连接后，拿到蓝牙电话客户端的代理 mHeadsetClient 
        // Api 接口： IBluetoothHeadsetClient.aidl
        mHeadsetClient = (BluetoothHeadsetClient) proxy;
    }
    @Override
    public void onServiceDisconnected(int profile) {
    }
}, BluetoothProfile.HEADSET);

// 给110打电话
mHeadsetClient.dial(mDevice, "110")

BluetoothHeadsetClient.java:

public BluetoothHeadsetClientCall dial(BluetoothDevice device, String number) {
    final IBluetoothHeadsetClient service = mService;
    return service.dial(device, number);
}

// 当我们获取得到一个BluetoothHeadsetClient时，会调用 BluetoothHeadsetClient 的构造函数，它会执行一个doBind()
BluetoothHeadsetClient(Context context, ServiceListener l) {
    doBind();
}

boolean doBind() {
    Intent intent = new Intent(IBluetoothHeadsetClient.class.getName());
    ComponentName comp = intent.resolveSystemService(mContext.getPackageManager(), 0);
    intent.setComponent(comp);
    if (comp == null || !mContext.bindServiceAsUser(intent, mConnection, 0, mContext.getUser())) {
        return false;
    }
    return true;
}

private final ServiceConnection mConnection = new ServiceConnection() {
    @Override
    public void onServiceConnected(ComponentName className, IBinder service) {
        // private volatile IBluetoothHeadsetClient mService;
        mService = IBluetoothHeadsetClient.Stub.asInterface(Binder.allowBlocking(service));
        mServiceListener.onServiceConnected(BluetoothProfile.HEADSET_CLIENT,
                BluetoothHeadsetClient.this);
    }

    @Override
    public void onServiceDisconnected(ComponentName className) {
    }
};

// 获得一个IBluetoothHeadsetClient 对象时，系统会去绑定一个服务，并获取这个服务的代理，而这个服务才是真正实现打电话的地方

// 该服务的实现在立即实现： HeadsetClientService
HeadsetClientService.java:

BluetoothHeadsetClientCall dial(BluetoothDevice device, String number) {
    // 安卓中的状态机
    HeadsetClientStateMachine sm = getStateMachine(device);

    int connectionState = sm.getConnectionState(device);

    BluetoothHeadsetClientCall call = new BluetoothHeadsetClientCall(
        device, HeadsetClientStateMachine.HF_ORIGINATED_CALL_ID,
        BluetoothHeadsetClientCall.CALL_STATE_DIALING, number, false  /* multiparty */,
        true  /* outgoing */);
    Message msg = sm.obtainMessage(HeadsetClientStateMachine.DIAL_NUMBER);
    msg.obj = call;
    sm.sendMessage(msg);
    return call;
}

HeadsetClientStateMachine.java:

public synchronized boolean processMessage(Message message) {
    switch (message.what) {
        case DIAL_NUMBER:
            // Add the call as an outgoing call.
            BluetoothHeadsetClientCall c = (BluetoothHeadsetClientCall) message.obj;
            mCalls.put(HF_ORIGINATED_CALL_ID, c);

            // 最终实现打电话的方法是 dialNative()， 一个本地方法
            // 接听拒接等同理
            if (NativeInterface.dialNative(getByteAddress(mCurrentDevice), c.getNumber())) {
                addQueuedAction(DIAL_NUMBER, c.getNumber());
                // Start looping on calling current calls.
                sendMessage(QUERY_CURRENT_CALLS);
            }
            break;
    }
    return HANDLED;
}
```

### 接听电话

如果有来电,服务端手机接通电话或者对方挂断电话,我们怎么知道呢？这些都是 com_android_bluetooth_hfpclient.cpp 通过 JNI 机制调用通话状态机的方法 sendCallChangedIntent,将电话的状态(包含在 BluetoothHeadsetClientCall.java 中)通过广播发送出来,第三方apk监听状态就可以进行相应的操作了。

HeadsetClientStateMachine.java:

```java
private void sendCallChangedIntent(BluetoothHeadsetClientCall c) {
    Intent intent = new Intent(BluetoothHeadsetClient.ACTION_CALL_CHANGED);
    intent.addFlags(Intent.FLAG_RECEIVER_FOREGROUND);
    intent.putExtra(BluetoothHeadsetClient.EXTRA_CALL, c);
    mService.sendBroadcast(intent, ProfileService.BLUETOOTH_PERM);
}
```

## Car 模块拨打电话

Android 7.0 增加车载新特性，在Car模块中实现了蓝牙电话功能

路径： android-8.0.0_r1\packages\apps\Car\Dialer\src\com\android\car\dialer

```java
DialerFragment.java：

callButton.setOnClickListener((unusedView) -> {
    getUiCallManager().safePlaceCall(mNumber.toString(), false);
});

UiCallManager.java:

public void safePlaceCall(String number, boolean bluetoothRequired) {
    placeCall(number);
}

TelecomUiCallManager.java:

@Override
public void placeCall(String number) {
    Uri uri = Uri.fromParts("tel", number, null);
    mTelecomManager.placeCall(uri, null);
}
```

TelecomManager.java:

```java
public void placeCall(Uri address, Bundle extras) {
    ITelecomService service = getTelecomService();
    service.placeCall(address, extras == null ? new Bundle() : extras,
            mContext.getOpPackageName());
}

private ITelecomService getTelecomService() {
    return ITelecomService.Stub.asInterface(ServiceManager.getService(Context.TELECOM_SERVICE));
}
```

TelecomService.java:

```java
@Override
public IBinder onBind(Intent intent) {
    synchronized (getTelecomSystem().getLock()) {
        return getTelecomSystem().getTelecomServiceImpl().getBinder();
    }
}
```

TelecomServiceImpl.java:

```java
@Override
public void placeCall(Uri handle, Bundle extras, String callingPackage) {
    final UserHandle userHandle = Binder.getCallingUserHandle();
    final Intent intent = new Intent(Intent.ACTION_CALL, handle);
    if (extras != null) {
        extras.setDefusable(true);
        intent.putExtras(extras);
    }
    mUserCallIntentProcessorFactory.create(mContext, userHandle)
            .processIntent(
                    intent, callingPackage, isSelfManaged ||
                            (hasCallAppOp && hasCallPermission));
}
```

UserCallIntentProcessor.java:

```java
public void processIntent(Intent intent, String callingPackageName,
            boolean canCallNonEmergency) {
    processOutgoingCallIntent(intent, callingPackageName, canCallNonEmergency);
}

private void processOutgoingCallIntent(Intent intent, String callingPackageName,
        boolean canCallNonEmergency) {
    intent.putExtra(CallIntentProcessor.KEY_INITIATING_USER, mUserHandle);
    sendBroadcastToReceiver(intent);
}
```

PrimaryCallReceiver.java:

```java
@Override
public void onReceive(Context context, Intent intent) {
    getTelecomSystem().getCallIntentProcessor().processIntent(intent);
}

@Override
public TelecomSystem getTelecomSystem() {
    return TelecomSystem.getInstance();
}
```

CallIntentProcessor.java:

```java
public void processIntent(Intent intent) {
    processOutgoingCallIntent(mContext, mCallsManager, intent);
}

static void processOutgoingCallIntent(
            Context context,
            CallsManager callsManager,
            Intent intent) {
    // Send to CallsManager to ensure the InCallUI gets kicked off before the broadcast returns
    Call call = callsManager
            .startOutgoingCall(handle, phoneAccountHandle, clientExtras, initiatingUser,
                    intent);
    sendNewOutgoingCallIntent(context, call, callsManager, intent);
}

static void sendNewOutgoingCallIntent(Context context, Call call, CallsManager callsManager,
            Intent intent) {
    // Asynchronous calls should not usually be made inside a BroadcastReceiver because once
    // onReceive is complete, the BroadcastReceiver's process runs the risk of getting
    // killed if memory is scarce. However, this is OK here because the entire Telecom
    // process will be running throughout the duration of the phone call and should never
    // be killed.
    NewOutgoingCallIntentBroadcaster broadcaster = new NewOutgoingCallIntentBroadcaster(
            context, callsManager, call, intent, callsManager.getPhoneNumberUtilsAdapter(),
            isPrivilegedDialer);
    final int result = broadcaster.processIntent();
}
```

NewOutgoingCallBroadcastIntentReceiver.java:

```java
public int processIntent() {
    Intent intent = mIntent;
    String action = intent.getAction();
    final Uri handle = intent.getData();

    if (handle == null) {
        Log.w(this, "Empty handle obtained from the call intent.");
        return DisconnectCause.INVALID_NUMBER;
    }

    boolean isVoicemailNumber = PhoneAccount.SCHEME_VOICEMAIL.equals(handle.getScheme());
    if (isVoicemailNumber) {
        if (Intent.ACTION_CALL.equals(action)
                || Intent.ACTION_CALL_PRIVILEGED.equals(action)) {
            // Voicemail calls will be handled directly by the telephony connection manager
            Log.i(this, "Placing call immediately instead of waiting for "
                    + " OutgoingCallBroadcastReceiver: %s", intent);

            // Since we are not going to go through "Outgoing call broadcast", make sure
            // we mark it as ready.
            mCall.setNewOutgoingCallIntentBroadcastIsDone();

            boolean speakerphoneOn = mIntent.getBooleanExtra(
                    TelecomManager.EXTRA_START_CALL_WITH_SPEAKERPHONE, false);
            mCallsManager.placeOutgoingCall(mCall, handle, null, speakerphoneOn,
                    VideoProfile.STATE_AUDIO_ONLY);

            return DisconnectCause.NOT_DISCONNECTED;
        } else {
            Log.i(this, "Unhandled intent %s. Ignoring and not placing call.", intent);
            return DisconnectCause.OUTGOING_CANCELED;
        }
    }

    String number = mPhoneNumberUtilsAdapter.getNumberFromIntent(intent, mContext);
    if (TextUtils.isEmpty(number)) {
        Log.w(this, "Empty number obtained from the call intent.");
        return DisconnectCause.NO_PHONE_NUMBER_SUPPLIED;
    }

    boolean isUriNumber = mPhoneNumberUtilsAdapter.isUriNumber(number);
    if (!isUriNumber) {
        number = mPhoneNumberUtilsAdapter.convertKeypadLettersToDigits(number);
        number = mPhoneNumberUtilsAdapter.stripSeparators(number);
    }

    final boolean isPotentialEmergencyNumber = isPotentialEmergencyNumber(number);
    Log.v(this, "isPotentialEmergencyNumber = %s", isPotentialEmergencyNumber);

    rewriteCallIntentAction(intent, isPotentialEmergencyNumber);
    action = intent.getAction();
    // True for certain types of numbers that are not intended to be intercepted or modified
    // by third parties (e.g. emergency numbers).
    boolean callImmediately = false;

    if (Intent.ACTION_CALL.equals(action)) {
        if (isPotentialEmergencyNumber) {
            if (!mIsDefaultOrSystemPhoneApp) {
                Log.w(this, "Cannot call potential emergency number %s with CALL Intent %s "
                        + "unless caller is system or default dialer.", number, intent);
                launchSystemDialer(intent.getData());
                return DisconnectCause.OUTGOING_CANCELED;
            } else {
                callImmediately = true;
            }
        }
    } else if (Intent.ACTION_CALL_EMERGENCY.equals(action)) {
        if (!isPotentialEmergencyNumber) {
            Log.w(this, "Cannot call non-potential-emergency number %s with EMERGENCY_CALL "
                    + "Intent %s.", number, intent);
            return DisconnectCause.OUTGOING_CANCELED;
        }
        callImmediately = true;
    } else {
        Log.w(this, "Unhandled Intent %s. Ignoring and not placing call.", intent);
        return DisconnectCause.INVALID_NUMBER;
    }

    if (callImmediately) {
        Log.i(this, "Placing call immediately instead of waiting for "
                + " OutgoingCallBroadcastReceiver: %s", intent);
        String scheme = isUriNumber ? PhoneAccount.SCHEME_SIP : PhoneAccount.SCHEME_TEL;
        boolean speakerphoneOn = mIntent.getBooleanExtra(
                TelecomManager.EXTRA_START_CALL_WITH_SPEAKERPHONE, false);
        int videoState = mIntent.getIntExtra(
                TelecomManager.EXTRA_START_CALL_WITH_VIDEO_STATE,
                VideoProfile.STATE_AUDIO_ONLY);
        mCallsManager.placeOutgoingCall(mCall, Uri.fromParts(scheme, number, null), null,
                speakerphoneOn, videoState);

        // Don't return but instead continue and send the ACTION_NEW_OUTGOING_CALL broadcast
        // so that third parties can still inspect (but not intercept) the outgoing call. When
        // the broadcast finally reaches the OutgoingCallBroadcastReceiver, we'll know not to
        // initiate the call again because of the presence of the EXTRA_ALREADY_CALLED extra.
    }

    UserHandle targetUser = mCall.getInitiatingUser();
    Log.i(this, "Sending NewOutgoingCallBroadcast for %s to %s", mCall, targetUser);
    broadcastIntent(intent, number, !callImmediately, targetUser);
    return DisconnectCause.NOT_DISCONNECTED;
}
```

CallsManager.java:

```java
Call startOutgoingCall(Uri handle, PhoneAccountHandle phoneAccountHandle, Bundle extras,
            UserHandle initiatingUser, Intent originalIntent) {
    // Ensure new calls related to self-managed calls/connections are set as such.  This
    // will be overridden when the actual connection is returned in startCreateConnection,
    // however doing this now ensures the logs and any other logic will treat this call as
    // self-managed from the moment it is created.
    if (account != null) {
        call.setIsSelfManaged(account.isSelfManaged());
        if (call.isSelfManaged()) {
            // Self-managed calls will ALWAYS use voip audio mode.
            call.setIsVoipAudioMode(true);
        }
    }

    call.setInitiatingUser(initiatingUser);
    isReusedCall = false;
}
```

Call.java:

```java
void startCreateConnection(PhoneAccountRegistrar phoneAccountRegistrar) {
    mCreateConnectionProcessor = new CreateConnectionProcessor(this, mRepository, this,
            phoneAccountRegistrar, mContext);
    mCreateConnectionProcessor.process();
}
```

CreateConnectionProcessor.java:

```java
public void process() {
    attemptNextPhoneAccount();
}

private void attemptNextPhoneAccount() {
    mService.createConnection(mCall, this);
}
```
最终还是调用 NativeInterface.dialNative(), 同上

## Car 模块接听电话

android-8.0.0_r1\packages\apps\Car\Dialer\src\com\android\car\dialer\telecom\embedded

```java
TelecomUiCallManager.java:

@Override
public void answerCall(UiCall uiCall) {
    Call telecomCall = mCallList.getTelecomCall(uiCall);
    if (telecomCall != null) {
        telecomCall.answer(0);
    }
}

Call.java:

public void answer(int videoState) {
    mConnectionService.answer(this, videoState);
}

ConnectionServiceWrapper.java:

void answer(Call call, int videoState) {
    final String callId = mCallIdMapper.getCallId(call);
    if (VideoProfile.isAudioOnly(videoState)) {
        mServiceInterface.answer(callId, Log.getExternalSession());
    } else {
        mServiceInterface.answerVideo(callId, videoState, Log.getExternalSession());
    }
}

ConnectionService.java:

public void answer(String callId, Session.Info sessionInfo) {
    SomeArgs args = SomeArgs.obtain();
    args.arg1 = callId;
    args.arg2 = Log.createSubsession();
    mHandler.obtainMessage(MSG_ANSWER, args).sendToTarget();
}

private final Handler mHandler = new Handler(Looper.getMainLooper()) {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case MSG_ANSWER: {
                SomeArgs args = (SomeArgs) msg.obj;
                answer((String) args.arg1);
                break;
            }
        }
    }
};

private void answer(String callId) {
    findConnectionForAction(callId, "answer").onAnswer();
}

private Connection findConnectionForAction(String callId, String action) {
    if (mConnectionById.containsKey(callId)) {
        return mConnectionById.get(callId);
    }
    return getNullConnection();
}

InCallAdapter.java:

// base\telecomm\java\android\telecom\InCallAdapter.java

// Instructs Telecom to reject the specified call.
public void answerCall(String callId, int videoState) {
    mAdapter.answerCall(callId, videoState);
}

InCallAdapter.java:

// packages\services\Telecomm\src\com\android\server\telecom\InCallAdapter.java

public void answerCall(String callId, int videoState) {
    Call call = mCallIdMapper.getCall(callId);
    mCallsManager.answerCall(call, videoState);
}

CallsManager.java:

// Instructs Telecom to answer the specified call
public void answerCall(Call call, int videoState) {
    call.answer(videoState);
}

Call.java:

// packages\services\Telecomm\src\com\android\server\telecom\Call.java

public void answer(int videoState) {
    // Check to verify that the call is still in the ringing state. A call can change states
    // between the time the user hits 'answer' and Telecom receives the command.
    if (isRinging("answer")) {
        if (!isVideoCallingSupported() && VideoProfile.isVideo(videoState)) {
            videoState = VideoProfile.STATE_AUDIO_ONLY;
        }
        // At this point, we are asking the connection service to answer but we don't assume
        // that it will work. Instead, we wait until confirmation from the connectino service
        // that the call is in a non-STATE_RINGING state before changing the UI. See
        // {@link ConnectionServiceAdapter#setActive} and other set* methods.
        mConnectionService.answer(this, videoState);
    }
}
```

最终还是调用 NativeInterface.handleCallActionNative(), 同上

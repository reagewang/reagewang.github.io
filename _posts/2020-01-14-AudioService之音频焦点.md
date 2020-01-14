---
layout: post
title: "AudioService之音频焦点"
tags: [audioservice,audiofocus]
comments: true
---

在分析AudioFocus机制之前,我们首先先来理解一下这个机制

* 首先,AudioFocus机制只是android建议我们去执行的一个规则,并不是一定要去遵循的.你设计的音频播放功能可以不采用这套机制.
* AudioFocus机制用来协调多个音频播放实例在播放时的复杂交互.

在同一时间,只能有一个音频播放实例拥有焦点,每个实例开始播放前,必须向AudioService申请获取AudioFocus,只有申请成功才允许开始回放.在回放实例播放结束后,要求释放AudioFocus.在音频实例播放的过程中,AudioFocus有可能被其他实例抢走,这时,被抢走AudioFocus的实例需要根据情况采取暂停、静音或降低音量的操作,以突出拥有AudioFocus的实例的播放.当AudioFocus被还回来时,回放实例可以恢复被抢走之前的状态,继续播放.

> `注意：关于被抢走AudioFocus的音频实例需要根据情况采取暂停、静音或降低音量的操作,这些操作是需要我们自己去实现的,并不是AudioService强制管理的,这就是我们刚刚所说的,AudioFocus机制只是android建议我们去执行的一个规则,并不是一定要去遵循的.`

* 通话作为手机的首要功能,同时也是一种音频的播放过程,所以从来电铃声开始到通话结束这个过程,Telephony相关的模块也会申请AudioFocus, 并且它的优先级是最高的.Telephony可以从所有人手中抢走AudioFocus,但是任何人无法从它手中将其夺回.

OK,了解了AudioFocus机制之后,我们就要开始分析它了,先来看下AudioFocus的使用案例：
```java

public void play(){
    //在开始播放前,先申请AudioFocus,注意传入的参数 
    int result = mAudioManager.requestAudioFocus(mAudioFocusListener, AudioManager.STREAM_MUSIC,AudioManager.AUDIOFOCUS_GAIN); 
    //只有成功申请到AudioFocus之后才能开始播放 
    if (result == AudioManager.AUDIOFOCUS_REQUEST_GRANTED) 
        mMediaPlayer.start(); 
    else//申请失败,如果系统没有问题,这一定是正在通话过程中,所以,还是不要播放了
        showMessageCannotStartPlaybackDuringACall();
} 
 
public void stop() { 
    //停止播放时,需要释放AudioFocus
    mAudioManager.abandonAudioFocus(mAudioFocusListener); 
}
 
private onAudioFocusChangeListener mAudioFocusListener = newOnAudioFocusChangeListener(){ 
    //当AudioFocus发生变化时,这个函数将会被调用.其中参数focusChange指示发生了什么变化
    public void onAudioFocusChange(int focusChange){
        switch( focusChange) { 
            /*AudioFocus被长期夺走,需要中止播放,并释放AudioFocus,这种情况对应于
抢走AudioFocus的申请者使用了AUDIOFOCUS_GAIN*/
            case AUDIOFOCUS_LOSS: 
                stop();
                break; 
            /*AudioFocus被临时夺走,不久就会被归还,只需要暂停,AudioFocus被归还后
再恢复播放 .这对应于抢走AudioFocus的申请者使用了AUDIOFOCUS_GAIN_TRANSIENT*/
            case AUDIOFOCUS_LOSS_TRANSIENT: 
                saveCurrentPlayingState(); 
                    pause();
                    break; 
            /*AudioFocus被临时夺走,允许不暂停,所以降低音量 ,这对应于抢走AudioFocus的
回放实例使用了AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK*/
            case AUDIOFOCUS_LOSS_TRANSIENT_CAN_DUCK:
                saveCurrentPlayingState();
                setVolume(getVolume()/2); 
                break; 
            /*AudioFocus被归还,这时需要恢复被夺走前的播放状态*/
            case AUDIOFOCUS_GAIN:
                restorePlayingState();
                break; 
        }
    }
};
```
点评：在开始播放前,应用首先要申请AudioFocus.申请AudioFocus时传入了3个参数.

* mAudioFocusListener：AudioFocus变化通知回调.

当AudioFocus被其他AudioFocus使用者抢走或归还时将通过这个回调对象进行通知.

* AudioManager.STREAM_MUSIC：这个参数表明了申请者即将使用哪种流类型进行播放.
* AudioManager.AUDIOFOCUS_GAIN： 这个参数指明了申请者将拥有这个AudioFocus多长时间.例子中传入的AUDIOFOCUS_GAIN表明了申请者将长期占用这个AudioFocus.

另外还有两个可能的取值,它们是AUDIOFOCUS_GAIN_TRANSIENT和AUDIOFOCUS_GAIN_TRANSIENT_MAY_DUCK.这两个取值的含义都表明申请者将暂时占用AudioFocus,不同的是,后者还指示了即将被申请者抢走AudioFocus的使用者不需要暂停,只要降低一下音量就可以了.

在停止播放时,需要调用abandonAudioFocus释放AudioFocus,会将其归还给之前被抢走AudioFocus的那个使用者.

我们逐步分析：

## 申请AudioFocus

申请AudioFocus,也就是mAudioManager.requestAudioFocus(mAudioFocusListener, AudioManager.STREAM_MUSIC,AudioManager.AUDIOFOCUS_GAIN),看下源码：
```java
public int requestAudioFocus(OnAudioFocusChangeListener l, int streamType, int durationHint) {
    PlayerBase.deprecateStreamTypeForPlayback(streamType,
            "AudioManager", "requestAudioFocus()");
    int status = AUDIOFOCUS_REQUEST_FAILED;
    try {
        status = requestAudioFocus(l,
                new AudioAttributes.Builder()
                        .setInternalLegacyStreamType(streamType).build(),
                durationHint,
                0 /* flags, legacy behavior */);
    } catch (IllegalArgumentException e) {
    }
 
    return status;
}
 
public int requestAudioFocus(OnAudioFocusChangeListener l,
        @NonNull AudioAttributes requestAttributes,
        int durationHint,
        int flags) throws IllegalArgumentException {
    if (flags != (flags & AUDIOFOCUS_FLAGS_APPS)) {
        throw new IllegalArgumentException("Invalid flags 0x"
                + Integer.toHexString(flags).toUpperCase());
    }
    return requestAudioFocus(l, requestAttributes, durationHint,
            flags & AUDIOFOCUS_FLAGS_APPS,
            null /* no AudioPolicy*/);
}
 
public int requestAudioFocus(OnAudioFocusChangeListener l,
        @NonNull AudioAttributes requestAttributes,
        int durationHint,
        int flags,
        AudioPolicy ap) throws IllegalArgumentException {
    ............
    ............
    ............
    /*使用Builder模式,将请求焦点的这个动作封装成AudioFocusRequest,
注意setOnAudioFocusChangeListenerInt()是用来设置AudioFocusRequest的mFocusListener和mListenerHandler,
mFocusListener就是我们传来进来的OnAudioFocusChangeListener,
mListenerHandler指的是一个Handler,此时为null*/
    final AudioFocusRequest afr = new AudioFocusRequest.Builder(durationHint)
            .setOnAudioFocusChangeListenerInt(l, null /* no Handler for this legacy API */)
            .setAudioAttributes(requestAttributes)
            .setAcceptsDelayedFocusGain((flags & AUDIOFOCUS_FLAG_DELAY_OK)
                    == AUDIOFOCUS_FLAG_DELAY_OK)
            .setWillPauseWhenDucked((flags & AUDIOFOCUS_FLAG_PAUSES_ON_DUCKABLE_LOSS)
                    == AUDIOFOCUS_FLAG_PAUSES_ON_DUCKABLE_LOSS)
            .setLocksFocus((flags & AUDIOFOCUS_FLAG_LOCK) == AUDIOFOCUS_FLAG_LOCK)
            .build();
    return requestAudioFocus(afr, ap);
}
 
public int requestAudioFocus(@NonNull AudioFocusRequest afr, @Nullable AudioPolicy ap) {
    ............
    ............
    ............
    //注册AudioFocusListener
    registerAudioFocusRequest(afr);
    final IAudioService service = getService();
    final int status;
    int sdk;
    try {
        sdk = getContext().getApplicationInfo().targetSdkVersion;
    } catch (NullPointerException e) {
        // some tests don't have a Context
        sdk = Build.VERSION.SDK_INT;
    }
    try {
        //将由AudioService来执行请求
        status = service.requestAudioFocus(afr.getAudioAttributes(),
                afr.getFocusGain(), mICallBack,
                mAudioFocusDispatcher,
                getIdForAudioFocusListener(afr.getOnAudioFocusChangeListener()),
                getContext().getOpPackageName() /* package name */, afr.getFlags(),
                ap != null ? ap.cb() : null,
                sdk);
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
    return status;
}
```
代码很多,总结起来就是,显示将请求焦点的信息封装成AudioFocusRequest然后调用registerAudioFocusListener()完成AudioFocusListener的注册,调用AudioService的requestAudioFocus()函数将申请焦点的请求交由AudioService来执行.

先来看registerAudioFocusListener():
```java
public void registerAudioFocusRequest(@NonNull AudioFocusRequest afr) {
    final Handler h = afr.getOnAudioFocusChangeListenerHandler();
    //获取到的h为null,将AudioFocusRequest和Handler封装成FocusRequestInfo
    final FocusRequestInfo fri = new FocusRequestInfo(afr, (h == null) ? null :
        new ServiceEventHandlerDelegate(h).getHandler());
    final String key = getIdForAudioFocusListener(afr.getOnAudioFocusChangeListener());
    //key为AudioFocusListener的字符串型Id,value为FocusRequestInfo 
    mAudioFocusIdListenerMap.put(key, fri);
}
```
AudioFocusChangeListener原来是被注册进AudioManager的一个Map中而不是AudioService中. 注意, 这个Map的KEY是getIdForAudioFocusListener分配的一个字符串型的Id.

这样看来,AudioManager这一侧一定有一个代理负责接受AudioService的回调并从这个Map中通过id 将回调转发给相应的Listener;

这个代理是谁呢？就是mAudioFocusDispatcher,它作为参数传递给了AudioService;

当音频焦点的拥有者发生变化时,AudioService会回调mAudioFocusDispatcher的dispatchAudioFocusChange()方法：
```java

private final IAudioFocusDispatcher mAudioFocusDispatcher = new IAudioFocusDispatcher.Stub() {
    @Override
    public void dispatchAudioFocusChange(int focusChange, String id) {
        final FocusRequestInfo fri = findFocusRequestInfo(id);
        if (fri != null)  {
            //listener就是我们之前传递进来的mAudioFocusChangeListener 
            final OnAudioFocusChangeListener listener =
                    fri.mRequest.getOnAudioFocusChangeListener();
            if (listener != null) {
                //fri.mHandler == null,所以使用ServiceEventHandlerDelegate的Handler
                final Handler h = (fri.mHandler == null) ?
                        mServiceEventHandlerDelegate.getHandler() : fri.mHandler;
                final Message m = h.obtainMessage(
                        MSSG_FOCUS_CHANGE/*what*/, focusChange/*arg1*/, 0/*arg2 ignored*/,
                        id/*obj*/);
                /*将MSSG_FOCUS_CHANGE消息发送给ServiceEventHandlerDelegate的Handler去处理,
并携带者"音频焦点新的拥有者将拥有焦点多长时间"的标志*/
                h.sendMessage(m);
            }
        }
    }
};
```
看出,mAudioFocusDispatcher的实现非常轻量级,直接把focusChange和listener的id发送给了一个Handler去处理;

我们来看下这个Handler ,也就是 mServiceEventHandlerDelegate.getHandler()：
```java
mHandler = new Handler(looper) {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case MSSG_FOCUS_CHANGE: {
                final FocusRequestInfo fri = findFocusRequestInfo((String)msg.obj);
                if (fri != null)  {
                    final OnAudioFocusChangeListener listener =
                            fri.mRequest.getOnAudioFocusChangeListener();
                    if (listener != null) {
                        //调用OnAudioFocusChangeListener.onAudioFocusChange(),通知使用者AudioFocus的归属发生了变化
                        listener.onAudioFocusChange(msg.arg1);
                    }
                }
            }
            break;
        }
    }
}
```
registerAudioFocusListener分析结束了,再看AudioService.requestAudioFocus():
```java
public int requestAudioFocus(AudioAttributes aa, int durationHint, IBinder cb,
        IAudioFocusDispatcher fd, String clientId, String callingPackageName, int flags,
        IAudioPolicyCallback pcb, int sdk) {
    //先弄清楚传递进来几个参数.
    //durationHint：AudioManager.AUDIOFOCUS_GAIN
    //fd:我们刚刚分析过的AudioService回调回放实例的中介
    //clientId：用于唯一标识一个AudioFocusChangeListener
    //callingPackageName:回放实例所在的包名
    ...........
    ...........
    ...........
    //调用mMediaFocusControl.requestAudioFocus()
    return mMediaFocusControl.requestAudioFocus(aa, durationHint, cb, fd,
            clientId, callingPackageName, flags, sdk);
}
```
调用了调用mMediaFocusControl.requestAudioFocus(),继续看：
```java
protected int requestAudioFocus(AudioAttributes aa, int focusChangeHint, IBinder cb,
        IAudioFocusDispatcher fd, String clientId, String callingPackageName, int flags,int sdk) {
   
    // we need a valid binder callback for clients
    if (!cb.pingBinder()) {
        Log.e(TAG, " AudioFocus DOA client for requestAudioFocus(), aborting.");
        return AudioManager.AUDIOFOCUS_REQUEST_FAILED;
    }
 
    if (mAppOps.noteOp(AppOpsManager.OP_TAKE_AUDIO_FOCUS, Binder.getCallingUid(),
            callingPackageName) != AppOpsManager.MODE_ALLOWED) {
        return AudioManager.AUDIOFOCUS_REQUEST_FAILED;
    }
 
    synchronized(mAudioFocusLock) {
        //mFocusStack就是AudioFocus机制所基于的栈,栈中元素太多就拒绝焦点申请
        if (mFocusStack.size() > MAX_STACK_SIZE) {
            return AudioManager.AUDIOFOCUS_REQUEST_FAILED;
        }
 
        //来电铃声或者通话
        boolean enteringRingOrCall = !mRingOrCallActive
                & (AudioSystem.IN_VOICE_COMM_FOCUS_ID.compareTo(clientId) == 0);
        if (enteringRingOrCall) { mRingOrCallActive = true; }
 
        final AudioFocusInfo afiForExtPolicy;
        if (mFocusPolicy != null) {
            afiForExtPolicy = new AudioFocusInfo(aa, Binder.getCallingUid(),
                    clientId, callingPackageName, focusChangeHint, 0 /*lossReceived*/,flags, sdk);
        } else {
            afiForExtPolicy = null;
        }
 
        //焦点延迟,暂时理解为不延迟,我们只看focusGrantDelayed = false的情况
        boolean focusGrantDelayed = false;
 
        //检查一下当前情况下AudioService是否能够让申请者获取AudioFocus;
        //前面说过,如果通话占用了AudioFocus,任何人都不能够再申请AudioFocus
        if (!canReassignAudioFocus()) {
            if ((flags & AudioManager.AUDIOFOCUS_FLAG_DELAY_OK) == 0) {
                final int result = AudioManager.AUDIOFOCUS_REQUEST_FAILED;
                notifyExtFocusPolicyFocusRequest_syncAf(afiForExtPolicy, result, fd, cb);
                return result;
            } else {
                focusGrantDelayed = true;
            }
        }
 
        // external focus policy: delay request for focus gain?
        final int resultWithExtPolicy = AudioManager.AUDIOFOCUS_REQUEST_DELAYED;
        if (notifyExtFocusPolicyFocusRequest_syncAf(
                afiForExtPolicy, resultWithExtPolicy, fd, cb)) {
            // stop handling focus request here as it is handled by external audio focus policy
            return resultWithExtPolicy;
        }
 
        //AudioFocusDeathHandler用来监控其生命状态,当申请者意外退出后可以代其完成abandonAudioFocus操作
        AudioFocusDeathHandler afdh = new AudioFocusDeathHandler(cb);
 
        try {
            cb.linkToDeath(afdh, 0);
        } catch (RemoteException e) {
            return AudioManager.AUDIOFOCUS_REQUEST_FAILED;
        }
 
        //mFocusStack就是AudioFocus机制所基于的栈,栈中的元素类型为FocusRequester,它保存了一个回放实例的所有信息.
        //这里先处理一下特殊情况,如果申请者已经拥有了AudioFocus,那怎么办呢？
        if (!mFocusStack.empty() && mFocusStack.peek().hasSameClient(clientId)) {
            final FocusRequester fr = mFocusStack.peek();
            //申请者已经位于栈顶,也就是拥有了AudioFocus
            if (fr.getGainRequest() == focusChangeHint && fr.getGrantFlags() == flags) {
                //持有方式也没有发生变化,那就直接返回成功
                cb.unlinkToDeath(afdh, 0);
                notifyExtPolicyFocusGrant_syncAf(fr.toAudioFocusInfo(),
                        AudioManager.AUDIOFOCUS_REQUEST_GRANTED);
                return AudioManager.AUDIOFOCUS_REQUEST_GRANTED;
            }
 
            if (!focusGrantDelayed) {
                //改变了持有方式,则把这个回放线程从栈上先拿下来,后面再重新加进去,相当于重新申请
                mFocusStack.pop();
                fr.release();
            }
        }
 
 
        /*由于申请者可能曾经调用过requestAudioFocus,但是目前被别人夺走了,
        所以它现在应该在栈的某个位置上,先把它从栈中删除,注意第三个参数的意思是不通知AudioFocus的变化*/
        removeFocusStackEntry(clientId, false /* signal */, false /*notifyFocusFollowers*/);
 
        //创建新的FocusRequester对象,保存新的焦点申请者的信息
        final FocusRequester nfr = new FocusRequester(aa, focusChangeHint, flags, fd, cb,
                clientId, afdh, callingPackageName, Binder.getCallingUid(), this, sdk);
        if (focusGrantDelayed) {
            final int requestResult = pushBelowLockedFocusOwners(nfr);
            if (requestResult != AudioManager.AUDIOFOCUS_REQUEST_FAILED) {
                notifyExtPolicyFocusGrant_syncAf(nfr.toAudioFocusInfo(), requestResult);
            }
            return requestResult;
        } else {
            //通知位于栈顶,也就是当前拥有AudioFocus的回放实例,它的AudioFocus要被夺走了
            if (!mFocusStack.empty()) {
                //追溯该方法,会发现该方法调用fd.dispatchAudioFocusChange,也就是fd就是IAudioFocusDispatcher
                propagateFocusLossFromGain_syncAf(focusChangeHint, nfr);
            }
 
            //将新建的FocusRequester放到栈顶,使其拥有AudioFocus
            mFocusStack.push(nfr);
            nfr.handleFocusGainFromRequest(AudioManager.AUDIOFOCUS_REQUEST_GRANTED);
        }
        notifyExtPolicyFocusGrant_syncAf(nfr.toAudioFocusInfo(),
                AudioManager.AUDIOFOCUS_REQUEST_GRANTED);
 
        if (ENFORCE_MUTING_FOR_RING_OR_CALL & enteringRingOrCall) {
            runAudioCheckerForRingOrCallAsync(true/*enteringRingOrCall*/);
        }
    }
 
    //告诉申请者它请求焦点成功
    return AudioManager.AUDIOFOCUS_REQUEST_GRANTED;
}
```
申请焦点的流程已经结束了, 我们来总结一下：
* 通过canReassignAudioFocus()判断当前是否在通话中,如果在通话过程中, 则直接拒绝申请.
* 对申请者进行linkToDeath,使得在申请者意外退出后可以代其完成abandonAudioFocus操作.
* 对于已经持有AudioFocus的情况,如果没有改变持有方式,则不作任何处理,直接返回申请成功,否则将其从栈顶删除,暂时将栈顶让给下一个回 放实例.
* 通过回调告知此时处于栈顶的回放实例,它的AudioFocus将被夺走.
* 将申请者的信息加入栈顶,成为新的拥有AudioFocus的回放实例.

## 释放AudioFocus
释放音频焦点的代码就不贴了.
## 总结下AudioFocus的工作原理：
`AudioFocus的实现基础是一个栈.栈顶的使用者拥有AudioFocus.在申请AudioFocus成功时,申请者被放置在栈顶,同时,通知之前在栈顶的使用者,告诉它我抢走了AudioFocus.当释放AudioFocus时,使用者将从栈中被移除,因此AudioFocus将被返还给新的栈顶.`
> 有一个问题？如果栈中有4个元素,当栈底的元素被移除,即调用abandonAudioFocus释放AudioFocus后,栈顶的元素会收到回调通知吗？

答案是否.`因为栈底元素被移除影响不到栈顶元素,所以不会对AudioFocus的拥有者产生影响,只有栈顶的元素释放AudioFocus后才会影响到栈中的其他元素,并且栈顶元素只会影响到它后面的第一个元素.`


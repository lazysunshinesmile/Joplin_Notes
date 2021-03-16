Android 9.0灭屏流程

Android 9.0灭屏流程

当power键灭屏时，会在PhoneWindowManager中处理按键事件后，调用到PMS的gotoSleep()进行灭屏处理，下面直接看看PhoneWindowManger中对Power键灭屏的处理以及和PMS的交互。在按power后，PWS中如下：

```


case KeyEvent.KEYCODE_POWER: {
    
    cancelPendingAccessibilityShortcutAction();
    result &= ~ACTION_PASS_TO_USER;
    isWakeKey = false; 
    if (down) {
        
        interceptPowerKeyDown(event, interactive);
    } else 
        
        interceptPowerKeyUp(event, interactive, canceled);
    }
    break;
}


```

在处理Power键抬起事件时，开始了灭屏流程：

```
private void powerPress(long eventTime, boolean interactive, int count) {
    if (mScreenOnEarly && !mScreenOnFully) {
        Slog.i(TAG, "Suppressed redundant power key press while "
                + "already in the process of turning the screen on.");
        return;
    }
    if (count == 2) {
       ......
    } else if (interactive && !mBeganFromNonInteractive) {
        switch (mShortPressOnPowerBehavior) {
            
            case SHORT_PRESS_POWER_GO_TO_SLEEP:
                goToSleep(eventTime, PowerManager.GO_TO_SLEEP_REASON_POWER_BUTTON, 0);
                break;
            
            case SHORT_PRESS_POWER_REALLY_GO_TO_SLEEP:
                goToSleep(eventTime, PowerManager.GO_TO_SLEEP_REASON_POWER_BUTTON,
                        PowerManager.GO_TO_SLEEP_FLAG_NO_DOZE);
                break;
                } else {
                    shortPressPowerGoHome();
                }
                break;
            }
        }
    }
}


```

在这里调用了goToSleep()方法，该方法如下：

```
private void goToSleep(long eventTime, int reason, int flags) {
    mRequestedOrGoingToSleep = true;
    mPowerManager.goToSleep(eventTime, reason, flags);
}


```

最终，PhoneWindowManager中调用了PowerManager的goToSleep()方法来灭屏。我们进入到PowerManager.goToSleep()方法：

```


public void goToSleep(long time, int reason, int flags) {
    try {
        mService.goToSleep(time, reason, flags);
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
}


```

可以看到，在PowerManger中开始向下调用到了PoweManagerService(以下简称PMS)中的goToSleep()中。  
我们进入PMS中，就需要详细分析其中的方法了，先来看看goToSleep()方法：

```

@Override 
public void goToSleep(long eventTime, int reason, int flags) {
    if (eventTime > SystemClock.uptimeMillis()) {
        throw new IllegalArgumentException("event time must not be in the future");
    }
    
    mContext.enforceCallingOrSelfPermission(
            android.Manifest.permission.DEVICE_POWER, null);

    final int uid = Binder.getCallingUid();
    final long ident = Binder.clearCallingIdentity();
    try {
        
        goToSleepInternal(eventTime, reason, flags, uid);
    } finally {
        Binder.restoreCallingIdentity(ident);
    }
}


```

这个方法的参数和PowerManager,PhoneWindowManager中的同名方法对应，需要注意的是第二个参数和第三个参数；  
第二个参数：表示灭屏原因，在PowerManager中定义了一些常量值来表示；  
第三个参数：是一个标识，用来表示是否直接进入灭屏，一般的灭屏流程，都会先进入Doze状态，然后才会进入Sleep状态，如果将flag设置为1，则将会直接进入Sleep状态，在goToSleep()方法中，检查权限之后，开始调用了goToSleepInternal()方法，该方法如下：

```
private void goToSleepInternal(long eventTime, int reason, int flags, int uid) {
    synchronized (mLock) {
        if (goToSleepNoUpdateLocked(eventTime, reason, flags, uid)) {
            updatePowerStateLocked();
        }
    }
}

```

这个方法逻辑很简单，首先是调用了goToSleepNoUpdateLocked()方法，并根据该方法返回值来决定是否调用updatePowerStateLocked()方法。

一般来说，goToSleepNoUpdateLocked()都会返回true，现在看看该方法：

```


@SuppressWarnings("deprecation")
private boolean goToSleepNoUpdateLocked(long eventTime, int reason, int flags, int uid) {

    if (eventTime < mLastWakeTime
            || mWakefulness == WAKEFULNESS_ASLEEP
            || mWakefulness == WAKEFULNESS_DOZING
            || !mBootCompleted || !mSystemReady) {
        return false;
    }
    try {
        switch (reason) {
            case PowerManager.GO_TO_SLEEP_REASON_DEVICE_ADMIN:
                Slog.i(TAG, "Going to sleep due to device administration policy "
                        + "(uid " + uid +")...");
                break;
            case PowerManager.GO_TO_SLEEP_REASON_TIMEOUT:
                Slog.i(TAG, "Going to sleep due to screen timeout (uid " + uid +")...");
                break;
            case PowerManager.GO_TO_SLEEP_REASON_LID_SWITCH:
                Slog.i(TAG, "Going to sleep due to lid switch (uid " + uid +")...");
                break;
            case PowerManager.GO_TO_SLEEP_REASON_POWER_BUTTON:
                Slog.i(TAG, "Going to sleep due to power button (uid " + uid +")...");
                break;
            case PowerManager.GO_TO_SLEEP_REASON_SLEEP_BUTTON:
                Slog.i(TAG, "Going to sleep due to sleep button (uid " + uid +")...");
                break;
            case PowerManager.GO_TO_SLEEP_REASON_HDMI:
                Slog.i(TAG, "Going to sleep due to HDMI standby (uid " + uid +")...");
                break;
            case PowerManager.GO_TO_SLEEP_REASON_ACCESSIBILITY:
                Slog.i(TAG, "Going to sleep by an accessibility service request (uid "
                        + uid +")...");
                break;
            default:
                Slog.i(TAG, "Going to sleep by application request (uid " + uid +")...");
                reason = PowerManager.GO_TO_SLEEP_REASON_APPLICATION;
                break;
        }
        
        mLastSleepTime = eventTime;
        
        mSandmanSummoned = true;
        
        setWakefulnessLocked(WAKEFULNESS_DOZING, reason);

        
        
        int numWakeLocksCleared = 0;
        final int numWakeLocks = mWakeLocks.size();
        for (int i = 0; i < numWakeLocks; i++) {
            final WakeLock wakeLock = mWakeLocks.get(i);
            switch (wakeLock.mFlags & PowerManager.WAKE_LOCK_LEVEL_MASK) {
                case PowerManager.FULL_WAKE_LOCK:
                case PowerManager.SCREEN_BRIGHT_WAKE_LOCK:
                case PowerManager.SCREEN_DIM_WAKE_LOCK:
                    numWakeLocksCleared += 1;
                    break;
            }
        }
        
        
        if ((flags & PowerManager.GO_TO_SLEEP_FLAG_NO_DOZE) != 0) {
            
            reallyGoToSleepNoUpdateLocked(eventTime, uid);
        }
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_POWER);
    }
    return true;
}


```

在这个方法中：  
首先，是判断调用该方法的原因并打印log，然后，通过setWakefulnessLocked()将当前wakefulness设置为Doze状态；最后，通过flag判断，如果flag为1,则调用reallyGoToSleepNoUpdateLocked()方法直接进入Sleep状态。  
因此，系统其他模块在调用PM.goToSleep()灭屏时，在除指定flag为PowerManager.GO\_TO\_SLEEP\_FLAG\_NO_DOZE的情况外，都会首先进入Doze，再由Doze进入Sleep。

setWakefulnessLocked()方法用来设置wakefulness值，同时将会调用Notifier中wakefulness相关的逻辑，这里再来看下：

```
@VisibleForTesting
void setWakefulnessLocked(int wakefulness, int reason) {
    if (mWakefulness != wakefulness) {
        
        mWakefulness = wakefulness;
        mWakefulnessChanging = true;
        
        mDirty |= DIRTY_WAKEFULNESS;
        if (mNotifier != null) {
            
            mNotifier.onWakefulnessChangeStarted(wakefulness, reason);
        }
    }
}


```

首先，改变当前mWakefulness值，将mWakefulnessChanging标记为true，将mWakefulness值标志为DIRTY_WAKEFULNESS，然后通过Notifier进行改变wakefulness之前的一些处理.  
我们跟着执行流程来进行分析，Notifier是PMS模块中用于进行“通知”的一个组件类，比如发送亮灭屏广播就是它来负责，具体详细的分析后面会分析到。这里针对于灭屏场景，再来看下其中的逻辑：

```
public void onWakefulnessChangeStarted(final int wakefulness, int reason) {
    
    final boolean interactive = PowerManagerInternal.isInteractive(wakefulness);
    
    
    
    if (mInteractive != interactive) {
        
        if (mInteractiveChanging) {
            handleLateInteractiveChange();
        }
        
        mInteractive = interactive;
        mInteractiveChangeReason = reason;
        mInteractiveChanging = true;
        
        handleEarlyInteractiveChange();
    }
}


```

在这个方法中，首先根据wakefulness值判断了系统当前的交互状态，如果是处于Awake状态和Dream状态，则表示可交互；如果处于Doze和Asleep状态，则表示不可交互；  
由于在setWakefulnessLocked()中已经设置了wakefulness为DOZE状态，因此此时处于不可交互状态，接下来开始执行handleEarlyInteractiveChange()方法：

```
private void handleEarlyInteractiveChange() {
    synchronized (mLock) {
        
        if (mInteractive) {
            
            mHandler.post(new Runnable() {
                @Override
                public void run() {
                    
                    mPolicy.startedWakingUp();
                }
            });
            
            mPendingInteractiveState = INTERACTIVE_STATE_AWAKE;
            mPendingWakeUpBroadcast = true;
            updatePendingBroadcastLocked();
        } else {
            final int why = translateOffReason(mInteractiveChangeReason);
            mHandler.post(new Runnable() {
                @Override
                public void run() {
                    
                    mPolicy.startedGoingToSleep(why);
                }
            });
        }
    }
}


```

在这个方法中，将调用mPolicy.startedGoingToSleep(why)进行锁屏流程(Keyguard的绘制)。

回到PMS中，在处理完setWakefulnessLocked()方法后，由于没有PowerManager.GO\_TO\_SLEEP\_FLAG\_NO_DOZE，所以不会立即执行reallyGoToSleepNoUpdateLocked()方法，此时goToSleepNoUpdateLocked()方法完毕并返回true。

之后开始执行updatePowerStateLocked()方法了，它是整个PMS的核心，后面会详细的分析 , 在这里我们只看其灭屏时的一些处理。

在updatePowerStateLocked()方法中，和灭屏直接相关的有如下部分：

```

boolean displayBecameReady = updateDisplayPowerStateLocked(dirtyPhase2);

updateDreamLocked(dirtyPhase2, displayBecameReady);

finishWakefulnessChangeIfNeededLocked();

updateSuspendBlockerLocked();


```

- updateDisplayPowerStateLocked()将会向DisplayPowerController请求新的屏幕状态，完成屏幕的更新；
    
- updateDreamLocked()方法用来更新屏保信息，除此之外还有一个任务——调用reallyGoToSleep()方法进入休眠，即由DOZE状态进入Sleep状态。
    
- finishWakefulnessChangeIfNeededLocked()方法用来做最后的收尾工作，当然，在这里会调用到Notifier中进行收尾。
    
- updateSuspendBlockerLocked()方法将用来更新SuspendBlocker锁，会根据当前的WakeLock类型以及屏幕状态来决定是否需要申请SuspendBlocker锁。
    

在updateDreamLocked()中更新屏保状态时，如果此时处于Doze状态且没有进行屏保，则将调用reallyGoToSleepNoUpdateLocked()方法，将wakefulness值设置为了Sleep,部分代码如下：

```
else if (wakefulness == WAKEFULNESS_DOZING) {
                if (isDreaming) {
                    return; 
                }

                
                reallyGoToSleepNoUpdateLocked(SystemClock.uptimeMillis(), Process.SYSTEM_UID);
                
    			updatePowerStateLocked();
            }


```

再来看看该方法：

```
private boolean reallyGoToSleepNoUpdateLocked(long eventTime, int uid) {
    if (eventTime < mLastWakeTime || mWakefulness == WAKEFULNESS_ASLEEP
            || !mBootCompleted || !mSystemReady) {
        return false;
    }

    try {
        
        setWakefulnessLocked(WAKEFULNESS_ASLEEP, 
                  PowerManager.GO_TO_SLEEP_REASON_TIMEOUT);
    } finally {
        Trace.traceEnd(Trace.TRACE_TAG_POWER);
    }
    return true;
}


```
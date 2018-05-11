---
title: Android之输入系统
date: 2018-02-05 20:31:47
tags:
---

Author JOY.
<!-- excerpt -->

### android 子系统主要由以下几个模块组成：   
EventHub：事件的传入是从EventHub开始的，EventHub是事件的抽象结构，维护着系统设备的运行情况，并维护一个所有输入设备的文件描述符   
Input Reader: 负责从硬件获取输入，转换成事件（Event), 并分发给Input Dispatcher.      
Input Dispatcher: 将Input Reader传送过来的Events 分发给合适的窗口，并监控ANR。   
Input Manager Service：负责Input Reader 和 Input Dispatchor的创建，并提供Policy 用于Events的预处理。   
Window Manager Service：管理Input Manager 与 View（Window) 以及 ActivityManager 之间的通信。
View and Activity：接收按键并处理。   
ActivityManager Service：ANR 处理

### getevent
输入设备插拔或者使用时，数据会实时显示出来

### 查看外接输入设备
cat /proc/bus/input/devices

## 源码分析

### android/app/ActivityThread.java
```Java   
final void handleResumeActivity(IBinder token, boolean clearHide, boolean isForward, boolean reallyResume) {
  wm.addView(decor, l);
}
```
### android/view/WindowManagerImpl.java
```Java
public void addView(@NonNull View view, @NonNull ViewGroup.LayoutParams params) {
    applyDefaultToken(params);
    mGlobal.addView(view, params, mDisplay, mParentWindow);
}
```
### android/view/WindowManagerGlobal.java
```Java
public void addView(View view, ViewGroup.LayoutParams params, Display display, Window parentWindow) {
  mViews.add(view);
  mRoots.add(root);
  mParams.add(wparams);
  root.setView(view, wparams, panelParentView);
}
```
### android/view/ViewRootImpl.java
```Java
public void setView(View view, WindowManager.LayoutParams attrs, View panelParentView) {
  mWindowSession.addToDisplay(mWindow, mSeq, mWindowAttributes,
                              getHostVisibility(), mDisplay.getDisplayId(),
                              mAttachInfo.mContentInsets, mAttachInfo.mStableInsets, mInputChannel);
}
```
### com/android/server/wm/WindowManagerService.java
```Java
public int addWindow(Session session, IWindow client, int seq, WindowManager.LayoutParams attrs, int viewVisibility, int displayId, Rect outContentInsets, Rect outStableInsets, InputChannel outInputChannel) {
  InputChannel[] inputChannels = InputChannel.openInputChannelPair(name);
  mInputManager.registerInputChannel(win.mInputChannel, win.mInputWindowHandle);
}
```

### frameworks/native/libs/input/InputTransport.cpp
在Linux中，完全可以把这一对socket当成pipe返回的文件描述符一样使用，唯一的区别就是这一对文件描述符中的任何一个都可读和可写。
返回一对连接好的sorcket。
```cpp

status_t InputChannel::openInputChannelPair(const String8& name,
        sp<InputChannel>& outServerChannel, sp<InputChannel>& outClientChannel) {
    int sockets[2];
    if (socketpair(AF_UNIX, SOCK_SEQPACKET, 0, sockets)) {
        status_t result = -errno;
        ALOGE("channel '%s' ~ Could not create socket pair.  errno=%d",
                name.string(), errno);
        outServerChannel.clear();
        outClientChannel.clear();
        return result;
    }

    int bufferSize = SOCKET_BUFFER_SIZE;
    setsockopt(sockets[0], SOL_SOCKET, SO_SNDBUF, &bufferSize, sizeof(bufferSize));
    setsockopt(sockets[0], SOL_SOCKET, SO_RCVBUF, &bufferSize, sizeof(bufferSize));
    setsockopt(sockets[1], SOL_SOCKET, SO_SNDBUF, &bufferSize, sizeof(bufferSize));
    setsockopt(sockets[1], SOL_SOCKET, SO_RCVBUF, &bufferSize, sizeof(bufferSize));

    String8 serverChannelName = name;

    serverChannelName.append(" (server)");
    outServerChannel = new InputChannel(serverChannelName, sockets[0]);

    String8 clientChannelName = name;
    clientChannelName.append(" (client)");
    outClientChannel = new InputChannel(clientChannelName, sockets[1]);
    return OK;
}

status_t InputPublisher::publishKeyEvent() {
   return mChannel->sendMessage(&msg);
}

status_t InputPublisher::publishMotionEvent() {
   return mChannel->sendMessage(&msg);
}

status_t InputChannel::sendMessage(const InputMessage* msg) {
  nWrite = ::send(mFd, msg, msgLength, MSG_DONTWAIT | MSG_NOSIGNAL);
}
```
### com/android/server/input/InputManagerService.java
```Java
public void registerInputChannel(InputChannel inputChannel,
        InputWindowHandle inputWindowHandle) {
    nativeRegisterInputChannel(mPtr, inputChannel, inputWindowHandle, false);
}
```
### frameworks/base/services/core/jni/com_android_server_input_InputManagerService.cpp
```cpp
  status_t status = im->registerInputChannel(
            env, inputChannel, inputWindowHandle, monitor);
```

### frameworks/native/services/inputflinger/InputDispatcher.cpp
```cpp
status_t InputDispatcher::registerInputChannel(const sp<InputChannel>& inputChannel, const sp<InputWindowHandle>& inputWindowHandle, bool monitor) {
  sp<Connection> connection = new Connection(inputChannel, inputWindowHandle, monitor);
  int fd = inputChannel->getFd();
  mConnectionsByFd.add(fd, connection);
  mLooper->addFd(fd, 0, ALOOPER_EVENT_INPUT, handleReceiveCallback, this);
}      

void InputDispatcher::startDispatchCycleLocked(nsecs_t currentTime,
const sp<Connection>& connection) {
  case EventEntry::TYPE_KEY: {
              KeyEntry* keyEntry = static_cast<KeyEntry*>(eventEntry);

              // Publish the key event.
              status = connection->inputPublisher.publishKeyEvent(dispatchEntry->seq,
                      keyEntry->deviceId, keyEntry->source,
                      dispatchEntry->resolvedAction, dispatchEntry->resolvedFlags,
                      keyEntry->keyCode, keyEntry->scanCode,
                      keyEntry->metaState, keyEntry->repeatCount, keyEntry->downTime,
                      keyEntry->eventTime);
              break;
          }
          case EventEntry::TYPE_MOTION: {
             connection->inputPublisher.publishMotionEvent(dispatchEntry->seq,
                      motionEntry->deviceId, motionEntry->source,
                      dispatchEntry->resolvedAction, dispatchEntry->resolvedFlags,
                      motionEntry->edgeFlags, motionEntry->metaState, motionEntry->buttonState,
                      xOffset, yOffset,
                      motionEntry->xPrecision, motionEntry->yPrecision,
                      motionEntry->downTime, motionEntry->eventTime,
                      motionEntry->pointerCount, motionEntry->pointerProperties,
                      usingCoords);
              break;
          }                   
}
```
### android/view/ViewRootImpl.java
``` Java
mInputEventReceiver = new WindowInputEventReceiver(mInputChannel,
                          Looper.myLooper());
```

### android/view/InputEventReceiver.java
```Java
public InputEventReceiver(InputChannel inputChannel, Looper looper) {
       if (inputChannel == null) {
           throw new IllegalArgumentException("inputChannel must not be null");
       }
       if (looper == null) {
           throw new IllegalArgumentException("looper must not be null");
       }

       mInputChannel = inputChannel;
       mMessageQueue = looper.getQueue();
       mReceiverPtr = nativeInit(new WeakReference<InputEventReceiver>(this),
               inputChannel, mMessageQueue);

       mCloseGuard.open("dispose");
   }
```

### frameworks/base/core/jni/android_view_InputEventReceiver.cpp
```cpp
void NativeInputEventReceiver::setFdEvents(int events) {
    if (mFdEvents != events) {
        mFdEvents = events;
        int fd = mInputConsumer.getChannel()->getFd();
        if (events) {
            mMessageQueue->getLooper()->addFd(fd, 0, events, this, NULL);
        } else {
            mMessageQueue->getLooper()->removeFd(fd);
        }
    }
}
```

### system/core/libutils/Looper.cpp
```cpp
int Looper::pollInner(int timeoutMillis) {
  int callbackResult = response.request.callback->handleEvent(fd, events, data);
}
```

### frameworks/base/core/jni/android_view_InputEventReceiver.cpp
``` cpp
int NativeInputEventReceiver::handleEvent(int receiveFd, int events, void* data) {
  status_t status = consumeEvents(env, false /*consumeBatches*/, -1, NULL);
}

status_t NativeInputEventReceiver::consumeEvents() {
  env->CallVoidMethod(receiverObj.get(),
                        gInputEventReceiverClassInfo.dispatchInputEvent, seq, inputEventObj);
}
```

### android/view/InputEventReceiver.java
```java
private void dispatchInputEvent(int seq, InputEvent event) {
    mSeqMap.put(event.getSequenceNumber(), seq);
    onInputEvent(event);
}
```

### android/view/ViewRootImpl.java
```Java
final class WindowInputEventReceiver extends InputEventReceiver {
        public WindowInputEventReceiver(InputChannel inputChannel, Looper looper) {
            super(inputChannel, looper);
        }

        @Override
        public void onInputEvent(InputEvent event) {
            /// M: record current key event and motion event to dump input event info for
            /// ANR analysis. @{
            if (event instanceof KeyEvent) {
                mCurrentKeyEvent = (KeyEvent) event;
                mKeyEventStartTime = System.currentTimeMillis();
                mKeyEventStatus = INPUT_DISPATCH_STATE_STARTED;
            } else {
                mCurrentMotion = (MotionEvent) event;
                mMotionEventStartTime = System.currentTimeMillis();
                mMotionEventStatus = INPUT_DISPATCH_STATE_STARTED;
            }
            /// @}
            enqueueInputEvent(event, this, 0, true);
        }
}    

void enqueueInputEvent() {
  if (processImmediately) {
    doProcessInputEvents();
  } else {
    scheduleProcessInputEvents();
  }
}  

void doProcessInputEvents() {
  deliverInputEvent(q);
}

private void deliverInputEvent(QueuedInputEvent q) {
  stage.deliver(q);
}

final class ViewPostImeInputStage extends InputStage {
  protected int onProcess(QueuedInputEvent q) {
    return processKeyEvent(q);
  }

  private int processKeyEvent(QueuedInputEvent q) {
    mView.dispatchKeyEvent(event)
  }
}
```

### com/android/internal/policy/impl/PhoneWindow.java
```Java
private final class DecorView extends FrameLayout implements RootViewSurfaceTaker
{
  public boolean dispatchKeyEvent(KeyEvent event)
  		{
  			final int keyCode = event.getKeyCode();
  			final int action = event.getAction();
  			final boolean isDown = action == KeyEvent.ACTION_DOWN;

  			if (isDown && (event.getRepeatCount() == 0))
  			{
  				// First handle chording of panel key: if a panel key is held
  				// but not released, try to execute a shortcut in it.
  				if ((mPanelChordingKey > 0) && (mPanelChordingKey != keyCode))
  				{
  					boolean handled = dispatchKeyShortcutEvent(event);
  					if (handled)
  					{
  						return true;
  					}
  				}

  				// If a panel is open, perform a shortcut on it without the
  				// chorded panel key
  				if ((mPreparedPanel != null) && mPreparedPanel.isOpen)
  				{
  					if (performPanelShortcut(mPreparedPanel, keyCode, event, 0))
  					{
  						return true;
  					}
  				}
  			}

  			if (!isDestroyed())
  			{
  				final Callback cb = getCallback();
  				if (DBG_MOTION)
  				{
  					Log.d(TAG, "dispatchKeyEvent = " + event + ", cb = " + cb + ", mFeatureId = " + mFeatureId);
  				}
  				final boolean handled = cb != null && mFeatureId < 0 ? cb.dispatchKeyEvent(event) : super.dispatchKeyEvent(event);
  				if (DBG_MOTION)
  				{
  					Log.d(TAG, "dispatchKeyEvent = " + event + ", handled = " + handled);
  				}
  				if (handled)
  				{
  					return true;
  				}
  			} else
  			{
  				Log.d(TAG, "Dropping key event due to destroyed state, event = " + event);
  			}

  			return isDown ? PhoneWindow.this.onKeyDown(mFeatureId, event.getKeyCode(), event) : PhoneWindow.this.onKeyUp(mFeatureId, event.getKeyCode(), event);
  		}
}

```

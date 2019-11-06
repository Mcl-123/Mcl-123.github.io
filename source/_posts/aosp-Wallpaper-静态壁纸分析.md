---
title: Android Wallpaper 静态壁纸分析
date: 2019-01-28 21:58:49
categories: "Android源码"
tags:
    - Android
    - Wallpaper
    - 源码
---

壁纸更新是一个壁纸服务，每换一张壁纸，就是将该图片写入壁纸文件，再启动一个壁纸服务读取该壁纸文件显示出来的过程

<!-- more -->

# 平台

Android 5.1

# 源代码路径

/frameworks/base/packages/SystemUI/src/com/android/systemui/

# 具体实现

## 显示流程

ImageWallpaper.java：

```java
/**
 * Default built-in wallpaper that simply shows a static image.
 */
@SuppressWarnings({"UnusedDeclaration"})
// 继承WallpaperService，随SystemUI 进程启动而启动
public class ImageWallpaper extends WallpaperService {
	// 壁纸加载的真正实现在 Engine中
	class DrawableEngine extends Engine {
		@Override
        public void onCreate(SurfaceHolder surfaceHolder) {
            updateSurfaceSize(surfaceHolder, getDefaultDisplayInfo(), false /* forDraw */);
        }
    }
}
```

updateSurfaceSize()：
```java
void updateSurfaceSize(SurfaceHolder surfaceHolder) {
    Point p = getDefaultDisplaySize();

    // Load background image dimensions, if we haven't saved them yet
    if (mBackgroundWidth <= 0 || mBackgroundHeight <= 0) {
        // Need to load the image to get dimensions
        mWallpaperManager.forgetLoadedWallpaper();
        // 加载壁纸图片
        updateWallpaperLocked();
        if (mBackgroundWidth <= 0 || mBackgroundHeight <= 0) {
            // Default to the display size if we can't find the dimensions
            mBackgroundWidth = p.x;
            mBackgroundHeight = p.y;
        }
    }

    // Force the wallpaper to cover the screen in both dimensions
    int surfaceWidth = Math.max(p.x, mBackgroundWidth);
    int surfaceHeight = Math.max(p.y, mBackgroundHeight);

    // If the surface dimensions haven't changed, then just return
    final Rect frame = surfaceHolder.getSurfaceFrame();
    if (frame != null) {
        final int dw = frame.width();
        final int dh = frame.height();
        if (surfaceWidth == dw && surfaceHeight == dh) {
            return;
        }
    }

    if (FIXED_SIZED_SURFACE) {
        // 图片尺寸大则用图片尺寸，屏幕尺寸大则用屏幕尺寸
        // 图片尺寸大时，滑动桌面时壁纸随之而动
        // Used a fixed size surface, because we are special.  We can do
        // this because we know the current design of window animations doesn't
        // cause this to break.
        surfaceHolder.setFixedSize(surfaceWidth, surfaceHeight);
    } else {
        surfaceHolder.setSizeFromLayout();
    }
}
```

updateWallpaperLocked():
```java
private void updateWallpaperLocked() {
    Throwable exception = null;
    try {
        mBackground = null;
        mBackgroundWidth = -1;
        mBackgroundHeight = -1;
        mBackground = mWallpaperManager.getBitmap();
        mBackgroundWidth = mBackground.getWidth();
        mBackgroundHeight = mBackground.getHeight();
    } catch (RuntimeException e) {
        exception = e;
    } catch (OutOfMemoryError e) {
        exception = e;
    }

    if (exception != null) {
        mBackground = null;
        mBackgroundWidth = -1;
        mBackgroundHeight = -1;
        // Note that if we do fail at this, and the default wallpaper can't
        // be loaded, we will go into a cycle.  Don't do a build where the
        // default wallpaper can't be loaded.
        Log.w(TAG, "Unable to load wallpaper!", exception);
        try {
            mWallpaperManager.clear();
        } catch (IOException ex) {
            // now we're really screwed.
            Log.w(TAG, "Unable reset to default wallpaper!", ex);
        }
    }
}
```

getBitmap():
```java
public Bitmap getBitmap() {
    return getBitmapAsUser(mContext.getUserId());
}

public Bitmap getBitmapAsUser(int userId) {
    return sGlobals.peekWallpaperBitmap(mContext, true, FLAG_SYSTEM, userId);
}
```

peekWallpaperBitmap():
```java
public Bitmap peekWallpaperBitmap(Context context, boolean returnDefault,
        @SetWallpaperFlags int which, int userId) {
    synchronized (this) {
        if (mCachedWallpaper != null && mCachedWallpaperUserId == userId) {
            return mCachedWallpaper;
        }
        mCachedWallpaper = null;
        mCachedWallpaperUserId = 0;
        try {
            mCachedWallpaper = getCurrentWallpaperLocked(userId);
            mCachedWallpaperUserId = userId;
        } catch (OutOfMemoryError e) {
            Log.w(TAG, "No memory load current wallpaper", e);
        }
        if (mCachedWallpaper != null) {
            return mCachedWallpaper;
        }
    }
    if (returnDefault) {
        // 如果上面没有得到壁纸资源就在这里取得默认壁纸即路径com.android.internal.R.drawable.default_wallpaper并 把图片写入/data/system/users/{userid}/wallpaper
        Bitmap defaultWallpaper = mDefaultWallpaper;
        if (defaultWallpaper == null) {
            defaultWallpaper = getDefaultWallpaper(context, which);
            synchronized (this) {
                mDefaultWallpaper = defaultWallpaper;
            }
        }
        return defaultWallpaper;
    }
    return null;
}
```

getCurrentWallpaperLocked()：

// 从路径data//system/users/{userid}/wallpaper取得当前用户壁纸，如果手机不是第一次启动这个一般能取到壁纸资源

```java
private Bitmap getCurrentWallpaperLocked(int userId) {
    if (mService == null) {
        Log.w(TAG, "WallpaperService not running");
        return null;
    }

    try {
        Bundle params = new Bundle();
        ParcelFileDescriptor fd = mService.getWallpaper(this, FLAG_SYSTEM,
                params, userId);
        if (fd != null) {
            try {
                BitmapFactory.Options options = new BitmapFactory.Options();
                return BitmapFactory.decodeFileDescriptor(
                        fd.getFileDescriptor(), null, options);
            } catch (OutOfMemoryError e) {
                Log.w(TAG, "Can't decode file", e);
            } finally {
                IoUtils.closeQuietly(fd);
            }
        }
    } catch (RemoteException e) {
        throw e.rethrowFromSystemServer();
    }
    return null;
}
```

drawFrame():

得到壁纸图片后开始绘制：

```java
void drawFrame() {
    try {
        int newRotation = ((WindowManager) getSystemService(WINDOW_SERVICE)).
                getDefaultDisplay().getRotation();

        // Sometimes a wallpaper is not large enough to cover the screen in one dimension.
        // Call updateSurfaceSize -- it will only actually do the update if the dimensions
        // should change
        if (newRotation != mLastRotation) {
            // Update surface size (if necessary)
            updateSurfaceSize(getSurfaceHolder());
        }
        SurfaceHolder sh = getSurfaceHolder();
        final Rect frame = sh.getSurfaceFrame();
        final int dw = frame.width();
        final int dh = frame.height();
        boolean surfaceDimensionsChanged = dw != mLastSurfaceWidth
                || dh != mLastSurfaceHeight;

        boolean redrawNeeded = surfaceDimensionsChanged || newRotation != mLastRotation;
        if (!redrawNeeded && !mOffsetsChanged) {
            return;
        }
        mLastRotation = newRotation;

        // Load bitmap if it is not yet loaded or if it was loaded at a different size
        if (mBackground == null || surfaceDimensionsChanged) {
            mWallpaperManager.forgetLoadedWallpaper();
            updateWallpaperLocked();
            if (mBackground == null) {
                return;
            }
        }

        // Center the scaled image
        mScale = Math.max(1f, Math.max(dw / (float) mBackground.getWidth(),
                dh / (float) mBackground.getHeight()));
        final int availw = dw - (int) (mBackground.getWidth() * mScale);
        final int availh = dh - (int) (mBackground.getHeight() * mScale);
        int xPixels = availw / 2;
        int yPixels = availh / 2;

        // Adjust the image for xOffset/yOffset values. If window manager is handling offsets,
        // mXOffset and mYOffset are set to 0.5f by default and therefore xPixels and yPixels
        // will remain unchanged
        final int availwUnscaled = dw - mBackground.getWidth();
        final int availhUnscaled = dh - mBackground.getHeight();
        if (availwUnscaled < 0)
            xPixels += (int) (availwUnscaled * (mXOffset - .5f) + .5f);
        if (availhUnscaled < 0)
            yPixels += (int) (availhUnscaled * (mYOffset - .5f) + .5f);

        mOffsetsChanged = false;
        mRedrawNeeded = false;
        if (surfaceDimensionsChanged) {
            mLastSurfaceWidth = dw;
            mLastSurfaceHeight = dh;
        }
        if (!redrawNeeded && xPixels == mLastXTranslation && yPixels == mLastYTranslation) {
            return;
        }
        mLastXTranslation = xPixels;
        mLastYTranslation = yPixels;

        // 是否支持硬件加速
        if (mIsHwAccelerated) {
            if (!drawWallpaperWithOpenGL(sh, availw, availh, xPixels, yPixels)) {
                drawWallpaperWithCanvas(sh, availw, availh, xPixels, yPixels);
            }
        } else {
            drawWallpaperWithCanvas(sh, availw, availh, xPixels, yPixels);
        }
    } finally {
        if (FIXED_SIZED_SURFACE && !mIsHwAccelerated) {
            // If the surface is fixed-size, we should only need to
            // draw it once and then we'll let the window manager
            // position it appropriately.  As such, we no longer needed
            // the loaded bitmap.  Yay!
            // hw-accelerated renderer retains bitmap for faster rotation
            mBackground = null;
            mWallpaperManager.forgetLoadedWallpaper();
        }
    }
}
```

## 设置流程

### 添加权限

```
<uses-permission android:name="android.permission.SET_WALLPAPER"/>
```

### 设置壁纸

```
WallpaperManager wallpaperManager = WallpaperManager.getInstance(this);
try {
wallpaperManager.setStream(InputStream,null,true,WallpaperManager.FLAG_LOCK);
} catch (IOException e) {
   e.printStackTrace();
}
```

### 分析设置过程

setStream():

```
try {
    //sGlobals.mService即WallpaperManagerService
    ParcelFileDescriptor fd = sGlobals.mService.setWallpaper(null,
            mContext.getOpPackageName(), visibleCropHint, allowBackup,
            result, which, completion, UserHandle.myUserId());
    if (fd != null) {
        FileOutputStream fos = null;
        try {
            // 拷贝壁纸图片到壁纸目录
            fos = new ParcelFileDescriptor.AutoCloseOutputStream(fd);
            copyStreamToWallpaperFile(bitmapData, fos);
            fos.close();
            completion.waitForCompletion();
        } finally {
            IoUtils.closeQuietly(fos);
        }
    }
} catch (RemoteException e) {
    throw e.rethrowFromSystemServer();
}
```

WallpaperManagerService.java 

setWallpaper():

```
public ParcelFileDescriptor setWallpaper(String name, String callingPackage,
                                            Rect cropHint, boolean allowBackup, Bundle extras, int which,
                                            IWallpaperManagerCallback completion, int userId) {

    checkPermission(android.Manifest.permission.SET_WALLPAPER);
    if ((which & (FLAG_LOCK|FLAG_SYSTEM)) == 0) {
        final String msg = "Must specify a valid wallpaper category to set";
        Slog.e(TAG, msg);
        throw new IllegalArgumentException(msg);
    }

    if (which == FLAG_SYSTEM && mLockWallpaperMap.get(userId) == null) {
        if (DEBUG) {
            Slog.i(TAG, "Migrating system->lock to preserve");
        }
        migrateSystemToLockWallpaperLocked(userId);
    }

    ParcelFileDescriptor pfd = updateWallpaperBitmapLocked(name, wallpaper, extras);
}
```

onEvent():

监听壁纸图片变化的回调

```
public void onEvent(int event, String path) {

    if (sysWallpaperChanged || lockWallpaperChanged) {
        notifyCallbacksLocked(wallpaper);
    }

    if (sysWallpaperChanged) {
        //桌面壁纸变化，那么bind ImageWallpaper，ImageWallpaper是负责显示静态桌面壁纸的
        // If this was the system wallpaper, rebind...
        bindWallpaperComponentLocked(mImageWallpaper, true,
                false, wallpaper, null);
        notifyColorsWhich |= FLAG_SYSTEM;
    }

    if (lockWallpaperChanged
            || (wallpaper.whichPending & FLAG_LOCK) != 0) {
        if (DEBUG) {
            Slog.i(TAG, "Lock-relevant wallpaper changed");
        }
        // either a lock-only wallpaper commit or a system+lock event.
        // if it's system-plus-lock we need to wipe the lock bookkeeping;
        // we're falling back to displaying the system wallpaper there.
        //如果参数which是system+lock，也就是同时设置锁屏和桌面壁纸，那么remove锁屏壁纸，因为已经是同一张壁纸了
        if (!lockWallpaperChanged) {
            mLockWallpaperMap.remove(wallpaper.userId);
        }
        // and in any case, tell keyguard about it
        notifyLockWallpaperChanged();
        notifyColorsWhich |= FLAG_LOCK;
    }

}
```

#### 锁屏壁纸更新

notifyLockWallpaperChanged():

```
void notifyLockWallpaperChanged() {
    final IWallpaperManagerCallback cb = mKeyguardListener;
    if (cb != null) {
        try {
            cb.onWallpaperChanged();
        } catch (RemoteException e) {
            // Oh well it went away; no big deal
        }
    }
}
```

LockscreenWallpaper.java:

```
public LockscreenWallpaper(Context ctx, PhoneStatusBar bar, Handler h) {
    try {
        //在这里给mKeyguardListener赋值的
        mService.setLockWallpaperCallback(this);
    } catch (RemoteException e) {
        Log.e(TAG, "System dead?" + e);
    }
}
```

LockscreenWallpaper.java onWallpaperChanged:

```
public void onWallpaperChanged() {
    // Called on Binder thread.
    mH.removeCallbacks(this);
    mH.post(this);
}
```

run():

实现了 runnable 接口，异步获取壁纸图片，更新 statusbar

```
public void run() {
    // Called in response to onWallpaperChanged on the main thread.
    mLoader = new AsyncTask<Void, Void, LoaderResult>() {
        @Override
        protected LoaderResult doInBackground(Void... params) {
            return loadBitmap(currentUser, selectedUser);
        }
        @Override
        protected void onPostExecute(LoaderResult result) {
            super.onPostExecute(result);
            if (isCancelled()) {
                return;
            }
            if (result.success) {
                mCached = true;
                mCache = result.bitmap;
                mUpdateMonitor.setHasLockscreenWallpaper(result.bitmap != null);
                //通知StatsuBar更新壁纸
                mBar.updateMediaMetaData(
                        true /* metaDataChanged */, true /* allowEnterAnimation */);
            }
            mLoader = null;
        }
    }.executeOnExecutor(AsyncTask.THREAD_POOL_EXECUTOR);
}
```

#### 更新桌面壁纸

ImageWallpaper.java bindWallpaperComponentLocked():

```
 boolean bindWallpaperComponentLocked(ComponentName componentName, boolean force,
                                        boolean fromUser, WallpaperData wallpaper, IRemoteCallback reply) {
    Intent intent = new Intent(WallpaperService.SERVICE_INTERFACE);
    WallpaperConnection newConn = new WallpaperConnection(wi, wallpaper);
    //componentName就是ImageWallpaper
    intent.setComponent(componentName);
    intent.putExtra(Intent.EXTRA_CLIENT_LABEL,
            com.android.internal.R.string.wallpaper_binding_label);
    intent.putExtra(Intent.EXTRA_CLIENT_INTENT, PendingIntent.getActivityAsUser(
            mContext, 0,
            Intent.createChooser(new Intent(Intent.ACTION_SET_WALLPAPER),
                    mContext.getText(com.android.internal.R.string.chooser_wallpaper)),
            0, null, new UserHandle(serviceUserId)));
}
```

WallpaperConnection.java onServiceConnected():

ImageWallpaper 继承自 service，bindService，所以看 conn 的 onServiceConnected()

```
public void onServiceConnected(ComponentName name, IBinder service) {
    synchronized (mLock) {
        if (mWallpaper.connection == this) {
            mService = IWallpaperService.Stub.asInterface(service);
            attachServiceLocked(this, mWallpaper);
            // XXX should probably do saveSettingsLocked() later
            // when we have an engine, but I'm not sure about
            // locking there and anyway we always need to be able to
            // recover if there is something wrong.
            saveSettingsLocked(mWallpaper.userId);
            FgThread.getHandler().removeCallbacks(mResetRunnable);
        }
    }
}
```

attachServcieLocked():

```
void attachServiceLocked(WallpaperConnection conn, WallpaperData wallpaper) {
    try {
        conn.mService.attach(conn, conn.mToken,
                TYPE_WALLPAPER, false,
                wallpaper.width, wallpaper.height, wallpaper.padding);
    } catch (RemoteException e) {
        Slog.w(TAG, "Failed attaching wallpaper; clearing", e);
        if (!wallpaper.wallpaperUpdating) {
            bindWallpaperComponentLocked(null, false, false, wallpaper, null);
        }
    }
}
```

IWallpaperService.Stub attach():

```
public void attach(IWallpaperConnection conn, IBinder windowToken,
        int windowType, boolean isPreview, int reqWidth, int reqHeight, Rect padding) {
    new IWallpaperEngineWrapper(mTarget, conn, windowToken,
            windowType, isPreview, reqWidth, reqHeight, padding);
}
```

IWallpaperEngineWrapper IWallpaperEngineWrapper()

```
Message msg = mCaller.obtainMessage(DO_ATTACH);
mCaller.sendMessage(msg);
```

```
case DO_ATTACH: {
                    try {
                        mConnection.attachEngine(this);
                    } catch (RemoteException e) {
                        Log.w(TAG, "Wallpaper host disappeared", e);
                        return;
                    }
                    Engine engine = onCreateEngine();
                    mEngine = engine;
                    mActiveEngines.add(engine);
                    engine.attach(this);
                    return;
                }
```

ImageWallpaper.java onCreateEngine():

```
public Engine onCreateEngine() {
    mEngine = new DrawableEngine();
    return mEngine;
}
```

Engine.java attach():

```
void attach(IWallpaperEngineWrapper wrapper) {
      onCreate(mSurfaceHolder);
      mInitializing = false;
      mReportedVisible = false;
      updateSurface(false, false, false);
 }
```

ImageWallpaper.java onCreate():

```
public void onCreate(SurfaceHolder surfaceHolder) {
    if (DEBUG) {
        Log.d(TAG, "onCreate");
    }
    super.onCreate(surfaceHolder);
    mDefaultDisplay = getSystemService(WindowManager.class).getDefaultDisplay();
    setOffsetNotificationsEnabled(false);
    updateSurfaceSize(surfaceHolder, getDefaultDisplayInfo(), false /* forDraw */);
}
```

updateSurfaceSize():

见流程

# Demo
[WallpaperDemo](https://github.com/Mcl-123/WallpaperDemo)
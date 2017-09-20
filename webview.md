## cookie 设置
```java

  public synchronized void fillCookie() {
        if (!SDKManager.getInstance().getHostInfoExtractor().isLogged()) {
            return;
        }
        CookieSyncManager.createInstance(SDKManager.getInstance().getApplicationContext());
        final HostInfoExtractor extractor = SDKManager.getInstance().getHostInfoExtractor();
        String[] hosts = HostWhiteListManager.get().getAll();
        for (String host : hosts) {
           
        }
        flush();
    }

    public synchronized void clearCookie() {
        CookieSyncManager.createInstance(SDKManager.getInstance().getApplicationContext());
        String[] hosts = HostWhiteListManager.get().getAll();
        for (String host : hosts) {
           
        }
        flush();
    }

    public void setCookie(String url, String cookieStr) {
        try {
            CookieManager cookieManager = CookieManager.getInstance();
            cookieManager.setAcceptCookie(true);
//domain前加个点才能覆盖已有的key，CookieTest来自CTS 源码
            cookieManager.setCookie(url, cookieStr + ";domain=." + url);
        } catch (Exception e) {
        }
    }

    private void flush() {
        try {
            CookieSyncManager.getInstance().sync();
            if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
                CookieManager.getInstance().flush();
            }
        } catch (Exception e) {
        }
    }

```

## webview 输入法 input标签时 自动滚动
```java

import android.app.Activity;
import android.graphics.Rect;
import android.os.Build;
import android.view.View;
import android.view.ViewTreeObserver;
import android.widget.FrameLayout;

//when the application uses full screen theme and the keyboard is shown the content not scrollable!
//with this util it will be scrollable once again
//http://stackoverflow.com/questions/7417123/android-how-to-adjust-layout-in-full-screen-mode-when-softkeyboard-is-visible
public class AndroidBug5497Workaround {


    private static AndroidBug5497Workaround mInstance = null;
    private View mChildOfContent;
    private int usableHeightPrevious;
    private FrameLayout.LayoutParams frameLayoutParams;
    private ViewTreeObserver.OnGlobalLayoutListener _globalListener;

    // For more information, see https://code.google.com/p/android/issues/detail?id=5497
    // To use this class, simply invoke assistActivity() on an Activity that already has its content view set.

    public static AndroidBug5497Workaround getInstance(Activity activity) {
        if (mInstance == null) {
            synchronized (AndroidBug5497Workaround.class) {
                mInstance = new AndroidBug5497Workaround(activity);
            }
        }
        return mInstance;
    }

    private AndroidBug5497Workaround(Activity activity) {
        FrameLayout content = (FrameLayout) activity.findViewById(android.R.id.content);
        mChildOfContent = content.getChildAt(0);
        frameLayoutParams = (FrameLayout.LayoutParams) mChildOfContent.getLayoutParams();
        _globalListener = () -> possiblyResizeChildOfContent();
    }

    public void subscribe() {
        mChildOfContent.getViewTreeObserver().addOnGlobalLayoutListener(_globalListener);
    }

    public void unsubscribe() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            mChildOfContent.getViewTreeObserver().removeOnGlobalLayoutListener(_globalListener);
        } else {
            mChildOfContent.getViewTreeObserver().removeGlobalOnLayoutListener(_globalListener);
        }
    }

    private void possiblyResizeChildOfContent() {
        int usableHeightNow = computeUsableHeight();
        if (usableHeightNow != usableHeightPrevious) {
            int usableHeightSansKeyboard = mChildOfContent.getRootView().getHeight();
            int heightDifference = usableHeightSansKeyboard - usableHeightNow;
            if (heightDifference > (usableHeightSansKeyboard / 4)) {
                // keyboard probably just became visible
                frameLayoutParams.height = usableHeightSansKeyboard - heightDifference;
            } else {
                // keyboard probably just became hidden
                frameLayoutParams.height = usableHeightSansKeyboard;
            }
            mChildOfContent.requestLayout();
            usableHeightPrevious = usableHeightNow;
        }
    }

    private int computeUsableHeight() {
        Rect r = new Rect();
        mChildOfContent.getWindowVisibleDisplayFrame(r);
        return (r.bottom - r.top);
    }
}
```

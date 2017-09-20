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

    private final Activity mActivity;
    private View mActivityContentView;
    private int usableHeightPrevious;
    private FrameLayout.LayoutParams mContentLayoutParams;
    private ViewTreeObserver.OnGlobalLayoutListener _globalListener;

    // For more information, see https://code.google.com/p/android/issues/detail?id=5497

    public AndroidBug5497Workaround(Activity activity) {
        _globalListener = () -> possiblyResizeChildOfContent();
        mActivity = activity;
        initIfNeed();
    }

    public void subscribe() {
        initIfNeed();
        mActivityContentView.getViewTreeObserver().addOnGlobalLayoutListener(_globalListener);
    }

    public void unsubscribe() {
        initIfNeed();
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.JELLY_BEAN) {
            mActivityContentView.getViewTreeObserver().removeOnGlobalLayoutListener(_globalListener);
        } else {
            mActivityContentView.getViewTreeObserver().removeGlobalOnLayoutListener(_globalListener);
        }
    }

    private void initIfNeed() {
        if (mActivityContentView == null) {
            FrameLayout content = (FrameLayout) mActivity.findViewById(android.R.id.content);
            if (content.getChildCount() > 0) {
                mActivityContentView = content.getChildAt(0);
                mContentLayoutParams = (FrameLayout.LayoutParams) mActivityContentView.getLayoutParams();
            }
        }
    }

    private void possiblyResizeChildOfContent() {
        int usableHeightNow = computeUsableHeight();
        if (usableHeightNow != usableHeightPrevious) {
            int usableHeightSansKeyboard = mActivityContentView.getRootView().getHeight();
            int heightDifference = usableHeightSansKeyboard - usableHeightNow;
            boolean keyboardVisible = heightDifference > (usableHeightSansKeyboard / 4);
            if (keyboardVisible) {
            //mActivityContentView y 可能为负数，会造成一块空白区域。所以 -getY()
                mContentLayoutParams.height = (int) (usableHeightSansKeyboard - mActivityContentView.getY() - heightDifference);
     
            } else {
                mContentLayoutParams.height = usableHeightSansKeyboard;
            }
            mActivityContentView.requestLayout();
            usableHeightPrevious = usableHeightNow;
        }

    }

    private int computeUsableHeight() {
        Rect r = new Rect();
        mActivityContentView.getWindowVisibleDisplayFrame(r);
        return (r.bottom - r.top);
    }
}
```

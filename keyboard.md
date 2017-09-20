## 自定义布局解决键盘弹出挡住输入框的问题
```java
import android.content.Context;
import android.graphics.Rect;
import android.util.AttributeSet;
import android.view.Display;
import android.view.View;
import android.view.WindowManager;
import android.widget.RelativeLayout;

public class InputMethodAdjustRelativeLayout extends RelativeLayout {
    public interface OnAdjustStatusChanged {
        void onAdjust();

        void onResume();
    }

    private int width;
    private boolean sizeChanged = false; //变化的标志
    private int height;
    private int screenHeight; //屏幕高度
    private View mTargetView; //键盘不遮盖的view
    private OnAdjustStatusChanged mOnAdjustStatusChangedListener;

    public void setOnAdjustStatusChanged(OnAdjustStatusChanged listener) {
        this.mOnAdjustStatusChangedListener = listener;
    }

    public InputMethodAdjustRelativeLayout(Context paramContext,
                                           AttributeSet paramAttributeSet) {
        super(paramContext, paramAttributeSet);
        WindowManager wm = (WindowManager) paramContext.getSystemService(Context.WINDOW_SERVICE);
        Display localDisplay = wm.getDefaultDisplay();
        this.screenHeight = localDisplay.getHeight();
    }

    public InputMethodAdjustRelativeLayout(Context paramContext,
                                           AttributeSet paramAttributeSet, int paramInt) {
        super(paramContext, paramAttributeSet, paramInt);
    }

    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        this.width = widthMeasureSpec;
        this.height = heightMeasureSpec;
        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
    }

    @Override
    public void onSizeChanged(int w, int h, int oldw, int oldh) {
        //监听不为空、宽度不变、当前高度与历史高度不为0
        if ((w == oldw) && (oldw != 0) && (oldh != 0)) {
            if ((h >= oldh) || (Math.abs(h - oldh) <= 1 * this.screenHeight / 5)) {
                if ((h <= oldh)
                        || (Math.abs(h - oldh) <= 1 * this.screenHeight / 5))
                    return;
                this.sizeChanged = false;
            } else {
                this.sizeChanged = true;
            }
            adJustKeyBoard(this.sizeChanged, oldh, h);
            measure(this.width - w + getWidth(), this.height
                    - h + getHeight());
        }
    }

    public void attachTargetView(View view) {
        this.mTargetView = view;
    }

    private Rect rect = new Rect();

    private void adJustKeyBoard(boolean sizeChanged, int w, int h) {
        if (sizeChanged) {
            //获取root在窗体的可视区域
            getWindowVisibleDisplayFrame(rect);
            //获取root在窗体的不可视区域高度(被其他View遮挡的区域高度)
            int rootInvisibleHeight = getRootView().getHeight() - rect.bottom;
            //若不可视区域高度大于100，则键盘显示
            if (rootInvisibleHeight > 100) {
                int[] location = new int[2];
                //获取scrollToView在窗体的坐标
                mTargetView.getLocationInWindow(location);
                //计算root滚动高度，使mTargetView在可见区域的底部
                int scrollHeight = (location[1] + mTargetView.getHeight()) - rect.bottom;
                if (scrollHeight > 0) {
                    if (mOnAdjustStatusChangedListener != null) {
                        mOnAdjustStatusChangedListener.onAdjust();
                    }
                    setPadding(0, -scrollHeight, 0, 0);
                }
            }
        } else {
            if (mOnAdjustStatusChangedListener != null) {
                mOnAdjustStatusChangedListener.onResume();
            }
            setPadding(0, 0, 0, 0);
        }
    }
}
```

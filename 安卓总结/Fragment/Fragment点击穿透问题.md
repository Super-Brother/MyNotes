```
package com.keytop.fcaam.findcar.fragment;

import android.app.Fragment;
import android.os.Bundle;
import android.support.annotation.Nullable;
import android.view.MotionEvent;
import android.view.View;

/**
 * BUG修正:thx http://www.itdadao.com/articles/c15a700976p0.html
 * 当Fragment栈中有多个add Fragment时，点击最上层Fragment时的空白处，如果对应的下层Fragment中存在按钮或其他事件，那么奇妙的事情就发生了，会穿透点击到下方的事件，不可否认，这是我们不愿意看到的。
 * 究其原因：Fragment的本质就是一个View布局的管理器，当Fragment attach到Activity时，其实就是把Fragment#onCreateView()返回的View，替换掉(如果是用replace)FragmentTransaction#replace中指定的View，或者添加到(如果是add)FragmentTransaction#add()中指定的ViewGroup里面。
 * 当我们以层叠方式显示多个Fragment时，通常的做法就是弄一个FrameLayout，然后每次把Fragment add到此布局。因此，这时Activity的页面布局树实际上就是一个FrameLayout里面包含几个View。
 * 所以，当点击上面Fragment的空白区域时，如果事件没被吃掉，就会向下传递。
 * Created by fengwenhua on 2017/5/18.
 */

public class BaseFragment extends Fragment implements View.OnTouchListener{

    @Override
    public void onViewCreated(View view, @Nullable Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        view.setOnTouchListener(this);
    }

    /**
     * Called when a touch event is dispatched to a view. This allows listeners to
     * get a chance to respond before the target view.
     *
     * @param v     The view the touch event has been dispatched to.
     * @param event The MotionEvent object containing full information about
     *              the event.
     * @return True if the listener has consumed the event, false otherwise.
     */
    @Override
    public boolean onTouch(View v, MotionEvent event) {
        return true;//消费掉点击事件,防止跑到下一层去
    }
}
```


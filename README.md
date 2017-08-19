使用seekbar实现滑动验证条
====

效果演示
---------

![image.png](http://upload-images.jianshu.io/upload_images/3537898-377e73e7d1a171ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

布局简介
---------

#### XML布局
```
    <SeekBar
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_margin="10dp"
        android:max="100"
        android:maxHeight="45dp"
        android:minHeight="45dp"
        android:progress="0"
        android:clickable="false"
        android:progressDrawable="@drawable/bg_forgotpassword_seekbar"
        android:thumb="@drawable/bg_seekbar_thumb"
        android:id="@+id/sb_progress"
        android:thumbOffset="-1dp"
        android:padding="1dp"
        />
```

#### 属性简介

1. android:max是总共的容量，这里设100就行。

2. android:progressDrawable是只设置进度框的背景，就是整个条的背景，比如图中的没滑动的时候是灰色，滑动的地方是绿色。

3. android:thumb这个属性是设置滑块的样式，比如图中的没滑时是>>，滑到最右变成勾。默认的样式是一个圆。

4. android:thumbOffset这个是滑块的起始位置，怎么说呢，你可以当图中第一条的滑块是android:thumbOffset = “0dp”，如果你设置成正数，这个滑块的位置会向左移动，设成负数会向右移动。我这里设成-1是以为是0的时候会挡住左边的边框，这样不好看，我设成-1为了让滑块向右移动一点。


#### 滑动条样式

android:progressDrawable="@drawable/bg_forgotpassword_seekbar"

```
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <!--seekBar背景-->
    <item android:id="@android:id/background">
        <!--形状-->
        <shape android:shape="rectangle">
            <!--大小-->
            <size android:height="29dp" />
            <!--圆角-->
            <corners android:radius="2dp" />
            <!--背景-->
            <solid android:color="#E7EAE9" />
            <!--边框-->
            <stroke
                android:width="1dp"
                android:color="#C3C5C4" />
        </shape>
    </item>
    <!--seekBar的进度条-->
    <item android:id="@android:id/progress">
        <clip>
            <shape>
                <corners android:radius="2dp" />
                <solid android:color="#7AC23C" />
                <stroke
                    android:width="1dp"
                    android:color="#C3C5C4" />
            </shape>
        </clip>
    </item>
</layer-list>
```
#### 滑块样式

```
<selector xmlns:android="http://schemas.android.com/apk/res/android">
    <!-- 按下状态-->
    <item android:drawable="@mipmap/seekbar_thumb" android:state_pressed="true" />

    <!-- 有焦点状态 -->
    <item android:drawable="@mipmap/seekbar_thumb" android:state_focused="true" />

    <!-- 普通状态 -->
    <item android:drawable="@mipmap/seekbar_thumb" />
</selector>
```

自定义seekbar
---------

#### 明明已经完成了滑动的效果，为什么需要自定义seekbar

>> 你以为这样就完啦？那你太天真了，你会发现如果你按上面的步骤做，最后会有一个很蛋疼的效果：
你不滑动滑块，只点击滑动条中间，滑块会马上到中间。
>> 也就是说我们想做的效果是只滑动而不能点击，仅仅做成这样是没办法实现这个需求。
那怎么办？我在网上找了很多文章，大多都是不能滑也不能点，而我要的是能滑不能点。难道SeekBar没戏啦？我想了想，最后我用事件分发来解决。

#### VerificationSeekBar 

```
sbProgress.setOnSeekBarChangeListener(new SeekBar.OnSeekBarChangeListener() {
            @Override
            public void onProgressChanged(SeekBar seekBar, int progress, boolean fromUser) {
  
            }

            @Override
            public void onStartTrackingTouch(SeekBar seekBar) {
         
            }

            @Override
            public void onStopTrackingTouch(SeekBar seekBar) {
                if (seekBar.getProgress() != seekBar.getMax()) {
                    seekBar.setProgress(0);
                }else {
                    sbProgress.setEnabled(false);
                    setData();
                }
            }
        });
```

最后实现的效果是这样：

![image.png](http://upload-images.jianshu.io/upload_images/3537898-377e73e7d1a171ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

你可以自己加文字在中间，这个我就不在demo里弄了。

### 四、剩下的处理

你以为这样就完啦？那你太天真了，你会发现如果你按上面的步骤做，最后会有一个很蛋疼的效果：
你不滑动滑块，只点击滑动条中间，滑块会马上到中间。
也就是说我们想做的效果是只滑动而不能点击，仅仅做成这样是没办法实现这个需求。

那怎么办？我在网上找了很多文章，大多都是不能滑也不能点，而我要的是能滑不能点。难道SeekBar没戏啦？我想了想，**最后我用事件分发来解决**。

既然是事件分发，我这里也不想写事件分发的内容，以后我们写一篇专门关于事件分发的文章，这里如果有小伙伴不了解事件分发的话，自己先去google一下。
既然是事件分发，那我们就需要自定义seekbar啦，其实很简单。我先贴代码，然后再讲解。

#####（1）代码君：

```
public class VerificationSeekBar extends SeekBar{
    //这两个值为用算法使用的2空间复杂度
    private int index = 150;
    private boolean k = true;

    public VerificationSeekBar(Context context) {
        super(context);
    }

    public VerificationSeekBar(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public VerificationSeekBar(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
    }


    @Override
    public boolean dispatchTouchEvent(MotionEvent event) {
        int x = (int) event.getX();
        if (event.getAction() == MotionEvent.ACTION_DOWN){
            k = true;
            if (x - index > 20) {
                k = false;
                return true;
            }
        }
        if (event.getAction() == MotionEvent.ACTION_MOVE){
            if (!k){
                return true;
            }
        }
        return super.dispatchTouchEvent(event);
    }
}
```

#### dispatchTouchEvent中的算法

>> 为了方便讲解，我把其它的内容删掉，就留关键方法，没错，就是dispatchTouchEvent。但是如果我不说，可能dispatchTouchEvent里面的代码你会看得蒙。
>> 先说说我的思想：简单来说就是你点击的地方要在滑块的范围，才分发事件，不然retrun true不分发事件。所以有了x - index > 20，这里的index =
150是我滑块的大概宽度，所以要你点击的地方在我滑块的宽度的20像素直接我才分发事件。所以x - index > 20的话不分发。
>> int x = (int) event.getX(); 获取点击时的坐标，注意，是相对于view左上角的，而不是相对屏幕的。
我这里分别按顺序判断了event.getAction() == MotionEvent.ACTION_DOWN和event.getAction() == MotionEvent.ACTION_MOVE，注意，是按顺序。为什么要按顺序呢？首先你自己测试你会发现，点击seekbar时ACTION_DOWN和ACTION_MOVE都会执行，所以你不能光判定按下，还要判断滑动。那为什么不一起判断而要按顺序判定呢？因为一起判断的话你可以试试，你会发现根本就滑不了。
而学过事件分发的都知道事件先执行ACTION_DOWN再执行ACTION_MOVE，所以先判断点击的地方是否在滑块+20像素的范围内，如果不在，定义一个布尔值k记录不在，然后执行 if (!k){return true;}

说明文档
---------

http://www.jianshu.com/p/777904acb056

不足之处
---------

>> 该方法虽然能方便的解决滑动解锁的问题，但是也有不足之处，那就是这个做法对尺寸的要求非常的细致。
>> 比如maxHeight和minHeight两个属性，如果偏大或偏小都不能达到完美包裹滑块的视觉效果，再比如自定义中的允许点击有效的范围是150也是根据滑块的宽度来设置的。



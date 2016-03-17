## 添加桌面悬浮控件
- 主要两个对象：
  - `WindowManager.WindowBase(复写其initLayoutParams方法)`
  - `view`    
- 主要操作过程
  - windowBase得到 layoutParamas，可以修改x，y坐标；
  - windowManager.show（view, layoutParamas，null）
  - windowManager.update(view，layoutParamas）

  - mWindowBase.remove()

- 代码

```java
// 被添加额view
RelativeLayout mGameRelativeLayout = (RelativeLayout) View.inflate(mContext.getApplicationContext(), R.layout.game_relative_layout, null);

// 主控对象WindowBase，并复写其得到initLayoutParams的方法
new WindowBase(mContext.getApplicationContext()) {
  @Override
  public WindowManager.LayoutParams initLayoutParams() {
      WindowManager.LayoutParams lp = new WindowManager.LayoutParams();
      lp.width = WindowManager.LayoutParams.WRAP_CONTENT;
      lp.height = WindowManager.LayoutParams.WRAP_CONTENT;
//                lp.verticalMargin = UIUtils.dip2Px(mContext.getApplicationContext(), 1);
      lp.gravity = Gravity.LEFT | Gravity.CENTER_VERTICAL;
      lp.format = PixelFormat.RGBA_8888;
      lp.type = WindowManager.LayoutParams.TYPE_PHONE;
//                lp.flags = WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE | WindowManager.LayoutParams.FLAG_NOT_TOUCHABLE;
      lp.flags = WindowManager.LayoutParams.FLAG_NOT_TOUCH_MODAL
              | WindowManager.LayoutParams.FLAG_NOT_FOCUSABLE;
      lp.packageName = mContext.getApplicationContext().getPackageName();
      return lp;
  }
```

- 控制位置代码    

```java
//windowBase得到 layoutParamas，可以修改x，y坐标；
windowManager.show（view, layoutParamas，null）
windowManager.update(view，layoutParamas）

// windowBase有个成员变量 WindowManager.LayoutParams
layoutParams.update（x，y）
```



## 手绘view
#### 代码添加extView
- new 一个view，创建ViewGroup.Params，设置LayoutParams

```java
TextView aimTextView = new TextView(this);
ViewGroup.LayoutParams layoutParams = new
       ViewGroup.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT,
       ViewGroup.LayoutParams.WRAP_CONTENT);
aimTextView.setLayoutParams(layoutParams);
aimTextView.setTextColor(Color.BLUE);
aimTextView.addView(result);
return aimTextView;
```


#### SurfaceView(就是画笔画)绘制

```
通过它的holder来处理、画图， this.getHolder()
获得canvas     holder.lockCanvas()
提交绘制结果  holder.unLockCanvasAndPost(canvas)
               canvas的绘制放在下面说
为holder增加回调     holder.addCallBack(SurfaceHolder.Callback)
实现SurfaceHolder.Callback的3个抽象方法：
     surfaceChanged    //在surface的大小发生改变时激发
     surfaceCreated      //在创建时激发，一般在这里调用画图的线程。
     surfaceDestroyed  //销毁时激发，一般在这里将画图的线程停止、释放。
```


#### Canvas绘制

1. 自定义view复写drawable（Canvas canvas)
2. SurfaceView在lockCanvas（）和unLockCanvasAndPost（canvas）之间
3. 代码创建一个Canvas
  1. 创建一个bitmap（指定canvas区域大小）`Bitmap.createBitmap(...)`
    ```
    Bitmap.createBitmap(200, 100, Config.ARGB_8888)(宽、高、flag）
    ```
  2. 创建canvas，传入bitmap（作画区域）
    ```
    new Canvas(bitmap)
    ```

  3. 利用canvas绘图去
    1. Canvas的方法
      ```
      clipRect（new Rect())      剪裁出一个新的绘画区域
      save()
      restore()
      drawRect()    drawaBitmap(bitmap, 20, 20, paint)
      drawBitmap()
      drawText()
      ```
    2. 画笔的方法
      ```
      Typeface font = Typeface.create("宋体", Typeface.BOLD_ITALIC);
      paint.setStyle(Paint.Style.STROKE);
      paint.setTypeface()
      paint.setTextScaleX(0.6f);    压缩字体，大于1拉伸，小于1压缩
      ```

#### android原生图形

  1. shape = [`oval`,`rectangle`]
  2. corners(设置圆角)
    - radius：设置四角半径
    - topLeftRadius：左上角半径
    - topRightRadius：右上角半径
  3. gradient（渐变）
    - startColor：
    - centerColor：
    - endColor：
    - angle：设置旋转角度（逆时针）
    - type：[`sweep`,`radial`'默认矩形']
    - centerX：渐变起始位置（0~1）之间
    - centerY：
    - gradientRadius：渐变type为radial时，设置渐变圆的半径
  4. padding间隔
    - left：
    - right：
  5. size大小
    width：
    height：
  6. solid填充
    color：
  7. stroke描边
    - width
    - color
    - dashWidth：虚线自身宽度
    - dashGap：虚线之间间距

```java
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval" >

    <solid android:color="#f85959" />
    <corners android:radius="3dp" />

</shape>

```

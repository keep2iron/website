---
title: 大话自定义view(1)
date: 2018-08-19 20:33:00
tags: Android
---

<h5>出师不利</h5>

小艾从新兵训练营毕业，在训练营中学习不少编程方面的知识，学着编写了不少Android方面的小程序，他踌躇满志一心想要报考*Android调查兵团*。终于当投递了无数次申请了之后，调查兵团准许了小艾的申请，并给与了他一次面试的机会。

到了要进行面试的日子，考官发放了一个题目，上面写道：“**请编写一个如下图的自定义控件**”

![pic1](https://i.imgur.com/X1ZyPpl.png)

<!-- more -->

小艾心想糟了，身上这一身本事都是CV老师教的(PS:老夫敲代码就是一梭子.png)，自己写这自定义控件还真的是不太行阿~。小艾心灰意冷的答完了题目，垂头丧一些气的回到了自己的房间，痛定思痛心想：这自定义控件劳资一定要攻破它，说着小艾就打开了aoogle搜索了一下这自定义控件的一些知识。

<h5>奇怪的Constructor</h5>小艾：哦，这第一步原来是要继承一个View或者一个ViewGroup​，我这个自定义控件是一个圆形进度条，那么我就起名叫**CircleProgressView**吧！
```java
public class CircleProgress extends View{
   public CircleProgress(Context context) {
     this(context, null);
   }

   //@Nullable是android扩展包的注解，标识这个变量是可以为null的
   public CircleProgress(Context context, @Nullable AttributeSet attrs) {
     this(context, attrs, 0);
   }

   public CircleProgress(Context context, @Nullable AttributeSet attrs, int defStyleAttr) {
   }
}
```

可是这自定义View为啥要重写了这三个方法呢？小艾这是产生了一个疑惑，不过正好今天是一个星期六，大表哥正好来小艾家玩，心里正犯嘀咕。大表哥看了一眼代码，瞅了一眼犯嘀咕的小艾。

大表哥："小艾呀，我知道你心中为啥这么困惑。首先第一个方法*CircleProgress(Context)*是保证了我们能够**直接创建一个自定义控件**因此才有了第一个构造方法。第二个方法的带了一个**AttributeSet**，这个方法是为了Android中使用**Pull解析**xml时，解析之后的属性便全部都放入了这个对象中，简单点说就是为了**提供给xml解析时调用**。第三个方法嘛，说来安卓真的是为我们开发者操碎了心，第三个参数用于解析主题文件时能够产生不同属性值，因此才有了这第三个参数。"
> 在第一种new出来的生成方式的时候，会调用一个参数的构造方法。

> 第二种使用xml声明的方式则会调用两个参数的构造方法

> 第三个参数的意义在于在当前主题中包含了style资源的一个属性值，适用于对于当前view的一个默认style。 需要特别指出的是三个参数的构造方法系统是不进行调用的，需要我们这里显式的进行调用，比如我们的Button通过调用，并传递了com.android.internal.R.attr.buttonStyle，而在4.4的主题文件是holo风格，然后定义了button的默认的几个属性

以Button为例下面，是Button的构造方法以及style文件中的声明。

```java
public Button(Context context) {
	this(context, null);
}

public Button(Context context, AttributeSet attrs) {
	this(context, attrs, com.android.internal.R.attr.buttonStyle);
}

public Button(Context context, AttributeSet attrs, int defStyle) {
	super(context, attrs, defStyle);
}
```

```xml
<style name="Theme.DeviceDefault" parent="Theme.Holo" >
	...
	<item name="buttonStyle">@android:style/Widget.DeviceDefault.Button</item>
	....
	</style>
	
	<style name="Widget.DeviceDefault.Button" parent="Widget.Holo.Button" >
	</style>
	
	<style name="Widget.Holo.Button" parent="Widget.Button">
	<item name="android:background">@android:drawable/btn_default_holo_dark</item>
	<item name="android:textAppearance">?android:attr/textAppearanceMedium</item>
	<item name="android:textColor">@android:color/primary_text_holo_dark</item>
	<item name="android:minHeight">48dip</item>
	<item name="android:minWidth">64dip</item>
</style>
```
<h5>懵逼的onMeasure</h5>大表哥：言归正传，以上就是构造函数的一些意义啦，这些东西多结合实战才会有进步~！当然了理解constructor的过程还是第一步，后面还有你好看的呢~！来看一下这个函数。
```java
@Override protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
	super.onMeasure(widthMeasureSpec, heightMeasureSpec);
	//获取测量的宽度和高度
	int widthSize = getMeasureSize(widthMeasureSpec);
	int heightSize = getMeasureSize(heightMeasureSpec);
	
	int size = widthSize > heightSize ? heightSize : widthSize;
	//必须调用这个方法，进行设置大小
	setMeasuredDimension(size,size);
}
	
private int getMeasureSize(int measureSpec) {
	int mode = MeasureSpec.getMode(measureSpec);
	int size = 0;
	
	switch(mode){
		case MeasureSpec.EXACTLY:
		case MeasureSpec.AT_MOST:
			size = MeasureSpec.getSize(measureSpec);
			break;
		case MeasureSpec.UNSPECIFIED:
			size = CommonUtil.dp2px(getContext(),50);
			break;
	}
	return size;
}

//决定了我们自己的控件相对与父控件的位置
@Override protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
	super.onLayout(changed, left, top, right, bottom);
}
```
小艾：哇，这个函数干啥玩意的咋越看越懵逼呢！！
大表哥：哈哈，懵逼吧~，不过我先跟你阐述几个概念，这几个概念你心里有个数，后面不会再查文档就行了。

**EXACTLY**

表示父视图希望子视图的大小应该是由specSize的值来决定的，系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

**AT_MOST**

表示子视图最多只能是specSize中指定的大小，开发人员应该尽可能小得去设置这个视图，并且保证不会超过specSize。系统默认会按照这个规则来设置子视图的大小，开发人员当然也可以按照自己的意愿设置成任意的大小。

**UNSPECIFIED**

表示开发人员可以将视图按照自己的意愿设置成任意的大小，没有任何限制。这种情况比较少见，不太会用到。
> 上面的一些表述参考了郭霖的[博客](http://blog.csdn.net/guolin_blog/article/details/16330267)
 
<h5>一些基本的操作</h5>
大表哥：今天主要是画这个进度是吧，既然要画东西（Canvas），起码我们是必须要有一个笔的，在Android中有个类叫**Paint**，你去官方文档看下它这个相关的api做一些demo基本上就能够上手啦~！当然我们这里再来说说获取了画笔之后，一些关于自定义控件的基本操作。有几点是要确定的
**1.绘制区域的大小（主要跟控件的大小有关）**
小艾：立方懵逼.png.......
大表哥：一般我觉得写自定义控件来说，获取他的大小基本上必不可少，那么在什么函数获取大小比较合适呢？我个人认为在onSizeChanged(int w, int h, int oldw, int oldh) 这个方法中进行赋值大小比较合适。虽然onDraw可以获取但是onDraw会可能会调用多次，书写一些初始化操作就不太合适了。
```java
@Override protected void onSizeChanged(int w, int h, int oldw, int oldh) {
	super.onSizeChanged(w, h, oldw, oldh);
	//初始化画笔的宽度
	int strokeSize = (int) (getMeasuredWidth() / 2 * 0.3f);
	int size = 10 + strokeSize;
	//初始化绘制区域
	mDrawRect = new RectF(getPaddingLeft() + size,getPaddingTop() + size,getWidth() -getPaddingLeft() - size,getHeight() - getPaddingBottom() - size);
	mPaint.setStrokeWidth(strokeSize);
}
```
<h5>干活啦！气氛搞起来~</h5>
大表哥：终于到了你应该最感兴趣的代码了，这里来说说绘制的流程

1. **canvas.save()**这里主要为了保存画布的状态，因为后面可能还需要针对画布进行旋转、平移、缩放等等操作。

2. **canvas.ratate/translate/scale**这里是针对画布进行平移缩放，当然了画布的坐标系也会同步修改。
3. **canvas.draw**这里就是干活的地方，发挥你灵魂画师的基本功力，在这里可以为所欲为的画你想画的东西。
4. **canvas.restore()**切记如果你之前**save过画布之后，一定要restore！！**,你也不想后面画着其他图形的时候犯嘀咕，咦？我这矩形咋飞上了天？？明明就是居中的阿~~。**主要就是恢复save时canvas的坐标系**。

下面就是一个代码实例，小艾你就跟着多练习啦，下午有个大保健等着我，溜了溜了
```java
@Override protected void onDraw(Canvas canvas) {
	//保存画布的当前状态
	canvas.save();
	//旋转画布逆时针90°，那么坐标系就发生了旋转了。
	canvas.rotate(-90,mDrawRect.centerX(),mDrawRect.centerY());
	//设置背景圆的颜色
	mPaint.setColor(mDefColor);
	canvas.drawCircle(mDrawRect.centerX(),mDrawRect.centerY(),mDrawRect.width() / 2,mPaint);
	//设置弧线的颜色
	mPaint.setColor(mColor);
	canvas.drawArc(mDrawRect,0,mProgress * 360,false,mPaint);
	//重新恢复画布
	canvas.restore();
}
```

小艾：留下了没技术的泪水.png~~~

自定义控件系列还在持续连载中!!!!coming soon

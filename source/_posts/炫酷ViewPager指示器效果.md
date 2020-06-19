---
title: 炫酷ViewPager指示器效果
date: 2017-07-31 18:12:34
tags: Simple Record
desc: 自定义控件
---

![](http://i.imgur.com/FRwSDei.gif)


# 写在前面的话

本文的源起是在有一天在网上看到的一个挺不错的一个效果而产生的一个想法，正好因为这段时间公司闲了下来，因此想着练习一下中定义view。

<!-- more -->

##### 本文以尽可能通俗的语言，让大家理解整个的绘制过程，尽量不粘贴代码(因为我认为思路往往比代码更重要)。还有就是可能对数学无感的人和不太友好。

## 一、这篇文章你将会学到什么?
> 1.学到一些自定义绘制中的一些技巧。
>  
> 2.学习Bezier的一些相关知识。
>  
> 3.利用面向对象更好的去解决一些复杂的问题。


## 二、准备

### 2.1 UI效果展示
### 原始效果图:
![](http://i.imgur.com/ybV4NBB.gif)

### 实际运行图:
![](http://i.imgur.com/FRwSDei.gif)

### 2.2 思考

>1.界面由 ViewPager + 自定义指示器
>
>2.ViewPager的间隔效果.
>
>3.小球能够和ViewPager联动不断变化

### 2.3 ViewPager效果实现
我们看到上面的是一个可以滑动的ViewPager，但是默认的ViewPager是一页只能显示一个Item的，因此经过多方查找，我找到了如下方法可以实现这个效果:

##### 2.3.1 ViewPager的布局
```xml
<android.support.v4.view.ViewPager
	android:layout_above="@+id/bezierIndicator"
	android:id="@+id/viewPager"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:layout_centerVertical="true"
	android:layout_gravity="bottom"
	android:clipToPadding="false"
	android:overScrollMode="never"
	android:paddingBottom="10dp"
	android:paddingEnd="@dimen/card_padding"
	android:paddingLeft="@dimen/card_padding"
	android:paddingRight="@dimen/card_padding"
	android:paddingStart="@dimen/card_padding"/>
```
##### android:clipToPadding="false" 

就是说控件的绘制区域是否在padding里面的，true的情况下绘制的区域不包括了Padding占据的那部分
![](http://i.imgur.com/uxdywV1.png)
蓝线的部分即为我们绘制的区域，因为设置了true，而且默认是true，而我们想要把绘制区域在padding中那么就要将这个属性设置为false了。

##### android:overScrollMode="never"
我们滑动的时候经常可以看到谷歌很多滑动的原生控件上都具有一个明显的特征(5.0以上)
![](http://i.imgur.com/DewB1bC.png)

有一个阴影对吧，这个效果默认是有的，这个效果的含义就是滑动的时候可以滑出区域外，有一个简单的回弹效果，如果不想要这个阴影，也就是这个回弹，那么可以将这个属性设置成never即可。

再来就是设置padding值。但是还有没结束。

##### 2.3.2 ViewPager的代码设置

        mViewPager = (ViewPager) findViewById(R.id.viewPager);
        mViewPager.setOffscreenPageLimit(3);
        mViewPager.setPageMargin(60);

1.设置viewpager缓存页数，因为默认ViewPager只加载一页，因此这里设置成三个，让其全部加载。
2.设置setPageMargin是为了控制每一页Page之间的大小。产生一个间隔的效果。这里和padding的不同在于Padding是设置了边界，也就是第一页左边的那个大小，因此这里是设置每一页之间的大小的。

经过了上面的配置，我们的ViewPager就可以完成了下面所示的效果
![](http://i.imgur.com/TDZ8ZEf.png)

### 2.4 BezierIndicator
上面我们一步一步实现了ViewPager的效果，接下来的重头戏就是如何去实现小球？

细心的我们可以发现这个球一共经历了5个状态，并且动画是*通过形状的变化加上画布的位移而产生*的
![](http://i.imgur.com/rHwii48.png)
![](http://i.imgur.com/987EBUH.gif)

下面我们来一步一步来进行Beizer的绘制工作

#### 2.4.1 预热一下
在开始画之前我们先来看一下这个Beizer的相关api，关于Bezier的数学原理在这里不会详细阐述(网络上有大量的说明，如果有兴趣可以自行查阅)。

在具体的讲解之前，我们来看看如果利用Bezier画圆。

下面这个图利用了PS中的钢笔工具进行了绘制，也就是说这个圆是由4条Bezier曲线组成的。
![](http://i.imgur.com/dMXbG4Q.png)

利用如下代码即可完成绘制
````
mPath.cubicTo(x1,y1,x2,y2,x3,y3);	//x1,x2都是控制点,x3是终点
````

![](http://i.imgur.com/i4jslVi.png)

P1 P2 P3 P4分别是圆上的4个端点,它们连线的圆点即为该点的控制点。M为控制点到圆点的距离,那么M值的大小直接影响了曲线变化！

代码如下:

    mPath.reset();

    mPath.reset();
    mPath.moveTo(p1.x, p1.y);
    mPath.cubicTo(p1.rightX, p1.rightY, p2.bottomX, p2.bottomY, p2.x, p2.y);
    mPath.cubicTo(p2.topX, p2.topY, p3.rightX, p3.rightY, p3.x, p3.y);
    mPath.cubicTo(p3.leftX, p3.leftY, p4.topX, p4.topY, p4.x, p4.y);
    mPath.cubicTo(p4.bottomX, p4.bottomY, p1.leftX, p1.leftY, p1.x, p1.y);

代码中，先将Path移动到p1点

	mPath.moveTo(p1.x, p1.y)
然后利用p1右边的控制点,p2下方的控制点，以及p2点的坐标即可画出p1到p2的曲线

	mPath.cubicTo(p1.rightX, p1.rightY, p2.bottomX, p2.bottomY, p2.x, p2.y);
其他的跟上面的写法是一样的，就不再赘述。

p1,p3的控制点由于是在水平方向上的，于是控制点的计算使用了如下的代码:

    this.x = x;
    this.y = y;
    this.m = m;

    leftX = x - m;
    rightX = x + m;

    leftY = y;
    rightY = y;
根据资料查询的结果: M =  *0.5522847498*时，曲线的绘制就是一个圆弧。
> [http://spencermortensen.com/articles/bezier-circle/](http://spencermortensen.com/articles/bezier-circle/)有一篇文章专门讲解了M点计算的原理。

## 三、开始
#### 3.1来面向对象一下
![](http://i.imgur.com/JpN6uGa.png)

首先将小球进行了抽象，抽取成一个单独的类*BerizerCircle*，然后画出小球的时候需要一些控制点的坐标。在这里我们分为了水平端点(*HorizontalPoint*)和垂直端点坐标(*VerticalPoint*)。

那么问题来了什么是水平端点和垂直端点呢？哈哈哈，其实这里我只是自己这么称呼的而已，不要介意。

水平端点即为在水平方向上具有控制点的点，对应了我们刚才图上所示的P1,P3的两个端点，那么垂直端点也就是P2、P4了

下面来看一下其中的**HorizontalPoint.java**的构造函数
	
    public HorizontalPoint(float x, float y, float m) {
	    this.x = x;
	    this.y = y;
	    this.m = m;
	
	    leftX = x - m;
	    rightX = x + m;
	
	    leftY = y;
	    rightY = y;
    }
这个构造函数的意思就是，通过设置端点的坐标(x,y)，以及端点到控制点的距离(M)，即可得到端点坐标和两个控制点的坐标。那么VerticalPoint类的构造函数的思路也就不用多说了。

那么构造端点方法可以通过如下的方式进行了

    p1 = new HorizontalPoint(0, R, M);
    p2 = new VerticalPoint(R, 0, M);
    p3 = new HorizontalPoint(0, -R, M);
    p4 = new VerticalPoint(-R, 0, M);

如果上述不是看的很懂那么再一个图来理解一下
![](http://i.imgur.com/XgOAb5N.png)

那么利用R,M我们可以把图中所有的端点和控制点进行了计算。

#### 3.2绘制流程

下面我们来看一下*BezierCircle*下面这个核心的函数，思路就是通过滑动的百分比，来控制图形的形态。

    /**
     * 通过ViewPager的百分比进行控制显示的状态
     *
     * @param fromPos
     * @param toPos
     * @param positionOffset Value from [0, 1) indicating the offset from the page at position.
     * @see android.support.v4.view.ViewPager.OnPageChangeListener
     */
    public void drawByPositionOffset(int fromPos, int toPos, float positionOffset) {
        boolean isTurnRight = toPos - fromPos > 0;

        if (positionOffset >= 0 && positionOffset <= 0.2f) {
            buildCircle1(positionOffset, isTurnRight);
        } else if (positionOffset <= 0.5f) {
            buildCircle2(positionOffset, isTurnRight);
        } else if (positionOffset <= 0.8f) {
            buildCircle3(positionOffset, isTurnRight);
        } else if (positionOffset <= 0.9f) {
            buildCircle4(positionOffset, isTurnRight);
        } else if (positionOffset <= 1.0f) {
            buildCircle5(positionOffset, isTurnRight);
        }
    }
![](http://i.imgur.com/Q0YEXmf.png)

上图就是一个小球变化的一个趋势图，整体的一个绘制思路和流程在上图可以进行了完整的体现。

那么下面我们来具体分析一下，小球在各个滑动区间中具体是如何变化的吧！

**3.2.1平移距离在(0,0.2]的范围内时**


![](http://i.imgur.com/ohP8VAU.gif)

p2点在这个状态的中从(R,0)变成了(9/5 *R,0),**positionOffset / 0.2f为这个区间(0,0.2]的变化率**。

> x = R + 4 / 5.0f * R * P,P是变化率也就是positionOffset / 0.2f
 
下面是代码：

    /**
     * 因为变化率是0 - 0.2f区间进行变化, 而p2点的x坐标则是从R - 4/5R之间进行变化,
     * 因此x = R + (4/5R * (变化率的百分比))
     * 变化率的百分比为 pos / 0.2f   0.2为区间值,pos在0 - 0.2f之间变化
     *
     * @param positionOffset
     */
    private void buildCircle1(float positionOffset) {   //0 < pos <=0.2f
        p1.setX(0);
        p3.setX(0);
        p4.setX(-R);

        p2.setX(R + 4 / 5.0f * R * positionOffset / 0.2f);
    }
如果对上面的公式或者注释你还是无法有一个直观的理解，可以结合下面的图来进行进一步的加深
![](http://i.imgur.com/GdUdPvK.png)

在buildCircle1中我们做的就是将P2点的坐标不断进行水平移动，从而让小球从状态1变化到了状态2了。

**3.2.2平移距离在(0.2,0.5]的范围内时**

![](http://i.imgur.com/mqA6dSV.gif)

在这个过程中我们需要将小球进行变化成一个椭圆
通过下图我们可以看到如果单纯将P2点垂直方向上的控制点的距离进行增大，图形的右边就更加的椭圆了，因为要对称，所以同时改变P2点和p4的垂直方向上M的距离，就可以让这两边的曲线更加接近椭圆。
![](http://i.imgur.com/REGqLHG.png)

但是还有一个问题，图形目前是不对称的，在上一个状态中我们移动了p2点的坐标R->9/5R，因此要让P2和p4对称，因此p1和p3也要同时进行移动，让其两点的x轴的坐标移动到新的中心点坐标。

**新的中心点在哪里？**

我们重新来看下坐标
P1(0,R)，P2(5/9R,0),P3(0，-R),P4（-R,0）
新的中心点的x轴的坐标为
> x = (p4.x + (p2.x - p4.x) / 2) * P

我们可以看到公式中有一个除2的操作，为什么要除2呢?
![](http://i.imgur.com/Q6tTjLZ.png)

**可以看到正方形，通过将右边的左边的点进行平移1个单位后**,中心点从(0,0)变化至(0.5,0)。通过这个公式p4.x + (p2.x - p4.x) / 2,这里我们将p2.x = 2，p4.x = -1，通过计算得出了0.5，因此这就是为什么要这么写的原因了。

下面是代码：

    /**
     * 在这个过程中需要将Circle变成一个椭圆
     * 这个里p2点的x是R + 4 / 5R,因此椭圆的长轴的距离为 R+ R + 4 / 5R = 14 / 5R
     * <p>
     * 然后 14 / 5R /2 为长轴一半 - R即为椭圆中心点的坐标
     *
     * @param positionOffset
     */
    private void buildCircle2(float positionOffset) {   //0.2 < pos <= 0.5f
        p2.setX(R + 4 / 5.f * R);

        float x = ((R + R + 4 / 5.f * R) / 2.f - R) * (positionOffset - 0.2f) / 0.3f;
        p1.setX(x);
        p3.setX(x);

        float m = M + M * 2 / 3.f * (positionOffset - 0.2f) / 0.3f;
        p4.setM(m);
        p2.setM(m);
    }

**3.2.3平移距离在(0.5,0.8]的范围内时**

![](http://i.imgur.com/3GKt8Rv.gif)

在这个过程中我们需要将椭圆变成如我们状态2那样子的有一头比较尖的圆形。

p2.x从9/5R 变化至 R

p4.x从-R 变化至 -9/5R

p1.x从((R +  9/5R) / 2.f - R)/2 变化至 0

p3.x从((R +  9/5R) / 2.f - R)/2 变化至 0

    /**
     * 在这个过程中需要将circle从一个椭圆变成左边较尖锐的椭圆
     *
     * @param positionOffset
     */
    private void buildCircle3(float positionOffset) {       //0.5 - 0.8f
        float x = ((R + R + 4 / 5.f * R) / 2.f - R) - ((R + R + 4 / 5.f * R) / 2.f - R) * (positionOffset - 0.5f) / 0.3f;
        p1.setX(x);
        p3.setX(x);

        float m = 5.f * M / 3 - 2.f * M / 3 * (positionOffset - 0.5f) / 0.3f;
        p2.setM(m);
        p4.setM(m);

        float p4X = -R - 4 / 5.f * R * (positionOffset - 0.5f) / 0.3f;
        p4.setX(p4X);

        float p2X = 9 / 5.f * R - 4 / 5.f * R * (positionOffset - 0.5f) / 0.3f;
        p2.setX(p2X);
    }

**3.2.4恢复成圆形(0.8,0.9]**

![](http://i.imgur.com/AZFm3Sd.gif)

p4.x 从-9/5R 变化至 -R。并且重置一下p1、p2、p3的坐标

    private void buildCircle4(float positionOffset) {       //0.8 - 1.0
        float p4X = -9 / 5.f * R + 4 / 5.f * R * (positionOffset - 0.8f) / 0.2f;
        p4.setX(p4X);

        p1.setX(0);
        p3.setX(0);
        p2.setX(R);
    }

**3.2.5让小球进行回弹起来吧[0.9,1.0]**

![](http://i.imgur.com/kwXQmEN.gif)

	double value = Math.sin(Math.toRadians((positionOffset - 0.9f) / 0.1f * 180));

	p4.setX((float) (-R +  R / 2.0f * value));

为社么这里使用了sin函数呢?
![](http://i.imgur.com/L0qETmy.jpg)

**在sin函数中x在[0，π/2]y轴的变化过程是[0,1]，x在[π/2，π]之间y轴的变化过程是[1,0]，这个y轴的变化过程正好满足我们这里回弹过程的变化率！！！因此使用sin作为这里控制回弹效果的产生是再合适不过的了。**


**3.2.6让小球进行平移**

前面我们都是通过改变p1,p2,p3,p4的坐标来改变小球的形状，然而还要进行添加位移，这样才能形成一个完整的动画，我在这里有两个思路
 
1.通过drawPath方法，不断地改变小球的x,y的位置来进行，然而这个的代价就是所有的坐标点都需要进行变化加上平移距离，得出最后的点的坐标
 
2.通过Canvas的translate方法，移动画布来达到我们这里平移动画的产生效果，显然，这一种方法我们不需要进行坐标转变即可完成动画效果，因为本控件也是采用了这一种方案。

![](http://i.imgur.com/cOCWL6Q.png)

    canvas.save();


    canvas.translate(mStartPoint.x
            + translateMoveSize * mPercent
            + (span + 2 * R) * mViewPagerPosition, mStartPoint.y);

    Path path = mBezierCircle.buildPath();
    mPaint.setColor(color);
    canvas.drawPath(path, mPaint);
    canvas.restore();

mStartPoint.x是计算的小球的起始位置。

	mStartPoint = new Point(span + R, h / 2);

translateMoveSize是小球移动的一个单位的距离，mPercent是移动的百分比，mViewPagerPosition当前Page的位置。mStartPoint.x 先让小球平移到初始位置,然后加上 (span + 2 * R) * mViewPagerPosition当前页滑动页的距离，然后加上滑动的百分比在滑动距离上的比值，计算出最后的平移距离。

span的计算是通过下面的公式进行等分的计算出来的距离。

	span = (getWidth() - (count * R * 2)) / (count + 1);

**3.2.7绑定ViewPager进行联动**

监听ViewPager的滑动事件自然是通过addOnPageChanageListener进行滑动的监听。在这里我们使用了如下的方法：

    @Override
    public void onPageScrolled(int position, float positionOffset, int positionOffsetPixels) {
        if (!isInitialise) return;

        if (!isMoveByTouch) {
            boolean right = position + positionOffset - mViewPagerPosition > 0;
            if (positionOffset != 0.0f) {
                //位移未完成
                startColor = roundColors[mViewPagerPosition];
                endColor = roundColors[(mViewPagerPosition + (right ? 1 : -1)) % roundColors.length];
                color = mBezierCircle.getCurrentColor(right ? positionOffset : 1 - positionOffset, startColor, endColor);
            }
            moveBezierCirclePercent(position, positionOffset);
        }
    }

在这里我们先来看一下ViewPager数值的变化规律:
这是从左向右滑动的时候,从第0页向第一页滑动position从0变化到1，positionOffset:0.0到0.999最后在完成翻页了之后变成了0.0.

![](http://i.imgur.com/h5iaGTj.png)

这是从右向左滑动的时候

![](http://i.imgur.com/p32wjpw.png)

	boolean right = position + positionOffset - mCurrentPosition > 0;
	if (positionOffset != 0.0f) {
	    //位移未完成
	    startColor = roundColors[mCurrentPosition];
	    endColor = roundColors[mCurrentPosition + (right ? 1 : -1)];
	    color = mBezierCircle.getCurrentColor(right ? positionOffset : 1 - positionOffset, startColor, endColor);
	}
	moveBezierCirclePercent(position, positionOffset);

这里是颜色的相关计算,mCurrentPosition是当前页的位置。

0到1页时,颜色值从a-b进行变化,变化率从0.0到0.9

1到0页时,颜色值从b-a进行变化,变化率从0.9到0.0,因此需要进行1 - positionOffset让其从0.0到0.9进行变化，才满足我们变化的颜色公式。

下面是颜色变化的计算函数

    public int getCurrentColor(float percent, int startColor, int endColor) {
        f[0][0] = (startColor & 0xff0000) >> 16;
        f[0][1] = (startColor & 0x00ff00) >> 8;
        f[0][2] = (startColor & 0x0000ff);

        f[1][0] = (endColor & 0xff0000) >> 16;
        f[1][1] = (endColor & 0x00ff00) >> 8;
        f[1][2] = (endColor & 0x0000ff);

        for (int i = 0; i < 3; i++) {
            result[i] = (int) (f[0][i] + (f[1][i] - f[0][i]) * percent);
        }

        return Color.rgb(result[0], result[1], result[2]);
    }

#### 3.3 点击产生的涟漪效果
实际的原理是通过属性动画进行改变画笔画圆的半径，然后通过设置画笔的粗细程度来完成这一效果的实现。

在onDraw方法画出点击的产生的圆

    private void drawTouchWave(Canvas canvas) {
        if (mTouchPoint.isShow()) {
            canvas.drawCircle(mTouchPoint.getCenterX(), mTouchPoint.getCenterY(), mTouchPoint.R, mTouchPoint.mPaint);
        }
    }

在onTouchEvent进行点击效果的触发

    private int getDistance(float x1, float y1, float x2, float y2) {
        return (int) Math.sqrt(Math.pow(x1 - x2, 2) + Math.pow(y1 - y2, 2));
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        super.onTouchEvent(event);

        switch (event.getActionMasked()) {
            case MotionEvent.ACTION_DOWN:
                for (int i = 0; i < mBitmapRects.length; i++) {
                    int distance = getDistance(mBitmapRects[i].centerX(), mBitmapRects[i].centerY(), event.getX(), event.getY());
                    if (distance <= R) {
                        mTouchPoint.setCenterX((int) mBitmapRects[i].centerX());
                        mTouchPoint.setCenterY((int) mBitmapRects[i].centerY());
                        mTouchPoint.setShow(true);
                        startWave();
                        startMoveBezierCircleByTouch(mCurrentPosition, i);
                        break;
                    }
                }
                break;
        }

        return true;
    }
通过两点之间的距离公式，判断是否在点击的区域范围内，然后通过startWave()方法进行显示点击的涟漪效果，通过startMoveBezierCircleByTouch方法进行从当前位置，跳转的指定的位置的平移变换。

    /**
     * 开启点击的水波纹
     */
    private void startWave() {
        ValueAnimator valueAnimator = ValueAnimator.ofFloat(R, 4.0f / 3 * R);
        valueAnimator.setDuration(400);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                float value = (float) animation.getAnimatedValue();
                float percent = (value - R) / (1.f / 3 * R);
                mTouchPoint.setStrokeWidth(TouchPoint.STROKE_WIDTH * (1 - percent));
                mTouchPoint.R = value;
                invalidate();
            }
        });
        valueAnimator.addListener(new AnimatorListenerAdapter() {
            @Override
            public void onAnimationEnd(Animator animation) {
                mTouchPoint.setShow(false);
                invalidate();
            }
        });
        valueAnimator.start();
    }

通过改变R的半径以及画笔的粗细程度，即可完成了这一效果的绘制过程

![](http://i.imgur.com/nBVpmbw.gif)

### 3.4 点击产生的位移
上面我们看到点击后通过属性动画完成涟漪效果的显示，**同样我们可以利用属性动画，让其模拟viewPager的参数的变化过程，这样之前的ViewPager函数就可以进行调用就行了**。

    private void startMoveBezierCircleByTouch(final int formPos, final int toPos) {
        if (formPos == toPos) return;
        final boolean isTurnRight = toPos - formPos > 0;
        translateMoveSize = Math.abs(toPos - formPos) * (span + 2 * R);
        setDurationScroller();

        //位移未完成
        startColor = roundColors[formPos];
        endColor = roundColors[toPos];

        mViewPager.setCurrentItem(toPos);
        ValueAnimator valueAnimator = ValueAnimator.ofInt(0, Math.abs(translateMoveSize));
        valueAnimator.setDuration(600);
        valueAnimator.addUpdateListener(new ValueAnimator.AnimatorUpdateListener() {
            @Override
            public void onAnimationUpdate(ValueAnimator animation) {
                isMoveByTouch = true;
                int value = (int) animation.getAnimatedValue();

                float percent = Math.abs(value * 1.0f) / Math.abs((toPos - formPos) * (span + 2 * R));
                color = mBezierCircle.getCurrentColor(percent, startColor, endColor);

                if (percent >= 1.0f) {
                    isMoveByTouch = false;
                    translateMoveSize = span + 2 * R;
                    mViewPagerPosition = toPos;
                    setOriginScroller();
                    moveBezierCirclePercent(toPos, 0.0f);
                } else {
                    moveBezierCirclePercent(isTurnRight ? formPos : toPos, isTurnRight ? percent : 1 - percent);
                }
            }
        });
        valueAnimator.start();
    }
因为ViewPager在0页到1页的过程中position是0，positionOffset是0-0.9直到当滑动完成后变成了0，1页到0页的过程中positionOffset是从0.9 - 0，position是0，因此这里就是模拟这样子的参数而编写代码。

## 四、总结

现在看一下之前的问题，看看心里有没有数

	do{
		1.ViewPager一页显示多个item？
		2.如何进行绘制一个不断变化的小球？
		3.如何让小球进行和ViewPager的进行绑定？
		4.如何让小球不断的进行变化？
		5.如何生成点击后的涟漪效果？
	}while(不懂);


![](http://i.imgur.com/dhsnvVB.png)

最后非常感谢***陈宇明***大佬的细心指导,文章的结构顺序、还有内容都一一进行指正和阅读，很多地方都是根据其建议进行修改的。

本篇的分析就到此为止了，后面我会抽时间不断的完善这个demo，如果你有好的建议，请加Bravh交流群和我一起交流/学习，我在等你哟~！

### 本项目地址
> https://github.com/keep2iron/BezierIndicator

##### 在编写代码的过程中也并不是一帆风顺的，再次感谢广大的博主和开源作者，Android社区的繁荣离不开你们辛勤的劳动。下面是参考的博客地址和项目地址
> http://blog.csdn.net/zanelove/article/details/50337623
> 
> http://www.jianshu.com/p/791d3a791ec2
> 
> https://github.com/Ulez/DropIndicator
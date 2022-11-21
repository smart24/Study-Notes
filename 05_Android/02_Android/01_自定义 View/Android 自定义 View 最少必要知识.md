## 1. 什么是自定义 View？  

### 1.1 定义  

在 Android 系统中，界面中所有能看到的元素都是 View。默认情况下，Android 系统为开发者提供了很多 View，比如用于展示文本信息的 TextView，用于展示图片的 ImageView 等等。但有时，这并不能满足开发者的需求，例如，开发者想要用一个饼状图来展示一组数据，这时如果用系统提供的 View 就不能实现了，只能通过自定义 View 来实现。那到底什么是自定义 View 呢？  

> 自定义 View 就是通过继承 View 或者 View 的子类，并在新的类里面实现相应的处理逻辑（重写相应的方法），以达到自己想要的效果。  


### 1.2 继承结构  

Android 中的所有 UI 元素都是 View 的子类：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf0102b6641eab~tplv-t2oaga2asx-image.image" width=1080 height=240 />  

PS：由于涉及的类太多，如果将所有涉及到的类全部加到类图里面，类图将十分大，所以此处只列出了 View 的直接子类。  


### 1.3 视图体系用到的设计模式  

Android View 体系如下：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf0103f4b5f06d~tplv-t2oaga2asx-image.image" width=640 height=280 />  

仔细观察，你会发现，Android View 的体系结构和设计模式中的组合模式的结构如出一辙：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf0102b6952935~tplv-t2oaga2asx-image.image" width=640 height=360 />  

Android View 体系结构中的 ViewGroup 对应于组合模式中抽象构件（Component 和 Composite），Android View 体系结构中的 View 对应于组合模式中的叶子构件（Leaf）：  

Android View 构件| Composite Pattern 构件
---|---
ViewGroup | Component、Composite
View | Leaf


## 2. 为什么要自定义 View？  

大多数情况下，开发者常常会因为下面四个原因去自定义 View：  

1. 让界面有特定的显示风格、效果；
2. 让控件具有特殊的交互方式；
3. 优化布局；
4. 封装；


### 2.1 让界面有特定的显示风格、效果  

默认情况下，Android 系统为开发者提供了很多控件，但有时，这并不能满足开发者的需求。例如，开发者想要用一个饼状图来展示一组数据，这时如果用系统提供的 View 就不能实现了，只能通过自定义 View 来实现。  

> If none of the prebuilt widgets or layouts meets your needs, you can create your own View subclass.  


### 2.2 让控件具有特殊的交互方式  

默认情况下，Android 系统为开发者提供的控件都有属于它们自己的特定的交互方式，但有时，控件的默认交互方式并不能满足开发者的需求。例如，开发者想要缩放 ImageView 中的图片内容，这时如果用系统提供的 ImageView 就不能实现了，只能通过自定义 ImageView 来实现。  


### 2.3 优化布局  

有时，有些布局如果用系统提供的控件实现起来相当复杂，需要各种嵌套，虽然最终也能实现了想要的效果，但性能极差，此时就可以通过自定义 View 来减少嵌套层级、优化布局。  


### 2.4 封装  

有些控件可能在多个地方使用，如大多数 App 里面的底部 Tab，像这样的经常被用到的控件就可以通过自定义 View 将它们封装起来，以便在多个地方使用。  


## 3. 如何自定义 View？  

在说「如何自定义 View？」之前，我们需要知道「自定义 View 都包括哪些内容」？  

自定义 View 包括三部分内容：  

1. 布局（Layout）
2. 绘制（Drawing）
3. 触摸反馈（Event Handling）

布局阶段：确定 View 的位置和尺寸。  
绘制阶段：绘制 View 的内容。  
触摸反馈：确定用户点击了哪里。  

其中布局阶段包括测量（measure）和布局（layout）两个过程，另外，布局阶段是为绘制和触摸反馈阶段做支持的，它并没有什么直接作用。正是因为在布局阶段确定了 View 的尺寸和位置，绘制阶段才知道往哪里绘制，触摸反馈阶段才知道用户点的是哪里。  

另外，由于触摸反馈是一个大的话题，限于篇幅，就不在这里讲解了，后面有机会的话，我会再补上一篇关于触摸反馈的文章。  


在自定义 View 和自定义 ViewGroup 中，布局和绘制流程虽然整体上都是一样的，但在细节方面，自定义 View 和自定义 ViewGroup 还是不一样的，所以，接下来分两类进行讨论：  

- 自定义 View 布局、绘制流程
- 自定义 ViewGroup 布局、绘制流程


### 3.1 自定义 View 布局、绘制流程  

「自定义 View 布局、绘制」主要包括三个阶段：  

1. 测量阶段（measure）
2. 布局阶段（layout）
3. 绘制阶段（draw）  


#### 3.1.1 自定义 View 测量阶段  

在 View 的测量阶段会执行两个方法（在测量阶段，View 的父 View 会通过调用 View 的 measure() 方法将父 View 对 View 尺寸要求传进来。紧接着 View 的 measure() 方法会做一些前置和优化工作，然后调用 View 的 onMeasure() 方法，并通过 onMeasure() 方法将父 View 对 View 的尺寸要求传入。在自定义 View 中，只有需要修改 View 的尺寸的时候才需要重写 onMeasure() 方法。在 onMeasure() 方法中根据业务需求进行相应的逻辑处理，并在最后通过调用 setMeasuredDimension() 方法告知父 View 自己的期望尺寸）：  

- measure()
- onMeasure()

**measure()** ： 调度方法，主要做一些前置和优化工作，并最终会调用 onMeasure() 方法执行实际的测量工作；  

**onMeasure()** ： 实际执行测量任务的方法，主要用与测量 View 尺寸和位置。在自定义 View 的 onMeasure() 方法中，View 根据自己的特性和父 View 对自己的尺寸要求算出自己的期望尺寸，并通过 setMeasuredDimension() 方法告知父 View 自己的期望尺寸。  

onMeasure() 计算 View 期望尺寸方法如下：  

1. 参考父 View 的对 View 的尺寸要求和实际业务需求计算出 View 的期望尺寸：  
        
    - 解析 widthMeasureSpec；
    - 解析 heightMeasureSpec；
    - 将「根据实际业务需求计算出 View 的尺寸」根据「父 View 的对 View 的尺寸要求」进行相应的修正得出 View 的期望尺寸（通过调用 resolveSize() 方法）；

2. 通过 setMeasuredDimension() 保存 View 的期望尺寸（实际上是通过 setMeasuredDimension() 告知父 View 自己的期望尺寸）;  

**注意：**  
多数情况下，这里的期望尺寸就是 View 的最终尺寸。不过最终 View 的期望尺寸和实际尺寸是不是一样还要看它的父 View 会不会同意。View 的父 View 最终会通过调用 View 的 layout() 方法告知 View 的实际尺寸，并且在 layout() 方法中 View 需要将这个实际尺寸保存下来，以便绘制阶段和触摸反馈阶段使用，这也是 View 需要在 layout() 方法中保存自己实际尺寸的原因——因为绘制阶段和触摸反馈阶段要使用啊！  


#### 3.1.2 自定义 View 布局阶段  

在 View 的布局阶段会执行两个方法（在布局阶段，View 的父 View 会通过调用 View 的 layout() 方法将 View 的实际尺寸（父 View 根据 View 的期望尺寸确定的 View 的实际尺寸）传给 View，View 需要在 layout() 方法中将自己的实际尺寸保存（通过调用 View 的 setFrame() 方法保存，在 setFrame() 方法中，又会通过调用 onSizeChanged() 方法告知开发者 View 的尺寸修改了）以便在绘制和触摸反馈阶段使用。保存 View 的实际尺寸之后，View 的 layout() 方法又会调用 View 的 onLayout() 方法，不过 View 的 onLayout() 方法是一个空实现，因为它没有子 View）：  

- layout()
- onLayout()

**layout()** ： 保存 View 的实际尺寸。调用 setFrame() 方法保存 View 的实际尺寸，调用 onSizeChanged() 通知开发者 View 的尺寸更改了，并最终会调用 onLayout() 方法让子 View 布局（如果有子 View 的话。因为自定义 View 中没有子 View，所以自定义 View 的 onLayout() 方法是一个空实现）；  

**onLayout()** ： 空实现，什么也不做，因为它没有子 View。如果是 ViewGroup 的话，在 onLayout() 方法中需要调用子 View 的 layout() 方法，将子 View 的实际尺寸传给它们，让子 View 保存自己的实际尺寸。因此，在自定义 View 中，不需重写此方法，在自定义 ViewGroup 中，需重写此方法。  

**注意：**  
layout() & onLayout() 并不是「调度」与「实际做事」的关系，layout() 和 onLayout() 均做事，只不过职责不同。  


#### 3.1.3 自定义 View 绘制阶段  

在 View 的绘制阶段会执行一个方法——draw()，draw() 是绘制阶段的总调度方法，在其中会调用绘制背景的方法 drawBackground()、绘制主体的方法 onDraw()、绘制子 View 的方法 dispatchDraw() 和 绘制前景的方法 onDrawForeground()：  

- draw()  

**draw()** ： 绘制阶段的总调度方法，在其中会调用绘制背景的方法 drawBackground()、绘制主体的方法 onDraw()、绘制子 View 的方法 dispatchDraw() 和 绘制前景的方法 onDrawForeground()；  

**drawBackground()** ： 绘制背景的方法，不能重写，只能通过 xml 布局文件或者 setBackground() 来设置或修改背景；  

**onDraw()** ： 绘制 View 主体内容的方法，通常情况下，在自定义 View 的时候，只用实现该方法即可；  

**dispatchDraw()** ： 绘制子 View 的方法。同 onLayout() 方法一样，在自定义 View 中它是空实现，什么也不做。但在自定义 ViewGroup 中，它会调用 ViewGroup.drawChild() 方法，在 ViewGroup.drawChild() 方法中又会调用每一个子 View 的 View.draw() 让子 View 进行自我绘制；  

**onDrawForeground()** ： 绘制 View 前景的方法，也就是说，想要在主体内容之上绘制东西的时候就可以在该方法中实现。  

**注意：**  
Android 里面的绘制都是按顺序的，先绘制的内容会被后绘制的盖住。如，你在重叠的位置「先画圆再画方」和「先画方再画圆」所呈现出来的结果是不同的，具体表现为下表：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf0103f4c76c31~tplv-t2oaga2asx-image.image" width=320 height=160 />  

<br/>  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf0103f4a8c299~tplv-t2oaga2asx-image.image" width=640 height=360 />  


#### 3.1.4 自定义 View 布局、绘制流程时序图  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf0103f49fee81~tplv-t2oaga2asx-image.image" width=960 height=1080 />  


### 3.2 自定义 ViewGroup 布局、绘制流程  

「自定义 ViewGroup 布局、绘制」主要包括三个阶段：  

1. 测量阶段（measure）
2. 布局阶段（layout）
3. 绘制阶段（draw）  


#### 3.2.1 自定义 ViewGroup 测量阶段  

同自定义 View 一样，在自定义 ViewGroup 的测量阶段会执行两个方法：  

- measure()
- onMeasure()  

**measure()** ： 调度方法，主要做一些前置和优化工作，并最终会调用 onMeasure() 方法执行实际的测量工作；  

**onMeasure()** ： 实际执行测量任务的方法，与自定义 View 不同，在自定义 ViewGroup 的 onMeasure() 方法中，ViewGroup 会递归调用子 View 的 measure() 方法，并通过 measure() 将 ViewGroup 对子 View 的尺寸要求（ViewGroup 会根据开发者对子 View 的尺寸要求、自己的父 View（ViewGroup 的父 View） 对自己的尺寸要求和自己的可用空间计算出自己对子 View 的尺寸要求）传入，对子 View 进行测量，并把测量结果临时保存，以便在布局阶段使用。测量出子 View 的实际尺寸之后，ViewGroup 会根据子 View 的实际尺寸计算出自己的期望尺寸，并通过 setMeasuredDimension() 方法告知父 View（ViewGroup 的父 View） 自己的期望尺寸。  

具体流程如下：  

1. 运行前，开发者在 xml 中写入对 ViewGroup 和 ViewGroup 子 View 的尺寸要求 layout_xxx；
2. ViewGroup 在自己的 onMeasure() 方法中，根据开发者在 xml 中写的对 ViewGroup 子 View 的尺寸要求、自己的父 View（ViewGroup 的父 View） 对自己的尺寸要求和自己的可用空间计算出自己对子 View 的尺寸要求，并调用每个子 View 的 measure() 将 ViewGroup 对子 View 的尺寸要求传入，测量子 View 尺寸；
3. ViewGroup 在子 View 计算出期望尺寸之后（在 ViewGroup 的 onMeasure() 方法中，ViewGroup 递归调用每个子 View 的 measure() 方法，子 View 在自己的 onMeasure() 方法中会通过调用 setMeasuredDimension() 方法告知父 View（ViewGroup） 自己的期望尺寸），得出子 View 的实际尺寸和位置，并暂时保存计算结果，以便布局阶段使用；
4. ViewGroup 根据子 View 的尺寸和位置计算自己的期望尺寸，并通过 setMeasuredDimension() 方法告知父 View 自己的期望尺寸。如果想要做的更好，可以在「 ViewGroup 根据子 View 的尺寸和位置计算出自己的期望尺寸」之后，再结合 ViewGroup 的父 View 对 ViewGroup 的尺寸要求进行修正（通过 resolveSize() 方法），这样得出的 ViewGroup 的期望尺寸更符合 ViewGroup 的父 View 对  ViewGroup 的尺寸要求。 


#### 3.2.2 自定义 ViewGroup 布局阶段  

同自定义 View 一样，在自定义 ViewGroup 的布局阶段会执行两个方法：  

- layout()
- onLayout()  

**layout()** ： 保存 ViewGroup 的实际尺寸。调用 setFrame() 方法保存 ViewGroup 的实际尺寸，调用 onSizeChanged() 通知开发者 ViewGroup 的尺寸更改了，并最终会调用 onLayout() 方法让子 View 布局；  

**onLayout()** ： ViewGroup 会递归调用每个子 View 的 layout() 方法，把测量阶段计算出的子 View 的实际尺寸和位置传给子 View，让子 View 保存自己的实际尺寸和位置。  


#### 3.2.3 自定义 ViewGroup 绘制阶段  

同自定义 View 一样，在自定义 ViewGroup 的绘制阶段会执行一个方法——draw()。draw() 是绘制阶段的总调度方法，在其中会调用绘制背景的方法 drawBackground()、绘制主体的方法 onDraw()、绘制子 View 的方法 dispatchDraw() 和 绘制前景的方法 onDrawForeground()：  

- draw()  

**draw()** ： 绘制阶段的总调度方法，在其中会调用绘制背景的方法 drawBackground()、绘制主体的方法 onDraw()、绘制子 View 的方法 dispatchDraw() 和 绘制前景的方法 onDrawForeground()；  

在 ViewGroup 中，你也可以重写绘制主体的方法 onDraw()、绘制子 View 的方法 dispatchDraw() 和 绘制前景的方法 onDrawForeground()。但大多数情况下，自定义 ViewGroup 是不需要重写任何绘制方法的。因为通常情况下，ViewGroup 的角色是容器，一个透明的容器，它只是用来盛放子 View 的。  


#### 3.2.4 自定义 ViewGroup 布局、绘制流程时序图  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf0103f48b0917~tplv-t2oaga2asx-image.image" width=960 height=1080 />  


### 3.3 自定义 View 步骤  

1. 自定义属性的声明与获取；
2. 重写测量阶段相关方法（onMeasure()）；
3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））；
4. 重写绘制阶段相关方法（onDraw() 绘制主体、dispatchDraw() 绘制子 View 和 onDrawForeground() 绘制前景）；
5. onTouchEvent()；
6. onInterceptTouchEvent()（仅 ViewGroup 有此方法）；


## 4. 实战演练  

### 4.1 自定义 View  

#### 4.1.1 自定义 View ——自定义 View 的绘制内容  

自定义 View，它的内容是「三个半径不同、颜色不同的同心圆」，效果图如下：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf010572139870~tplv-t2oaga2asx-image.image" width=360 height=640 />  

1. 自定义属性的声明与获取  

```
//1.1 在 xml 中自定义 View 属性
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!--CircleView-->
    <declare-styleable name="CircleView">
        <attr name="circle_radius" format="dimension" />
        <attr name="outer_circle_color" format="reference|color" />
        <attr name="middle_circle_color" format="reference|color" />
        <attr name="inner_circle_color" format="reference|color" />
    </declare-styleable>
</resources>

//1.2 在 View 构造函数中获取自定义 View 属性
TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
mRadius = typedArray.getDimension(R.styleable.CircleView_circle_radius, getResources().getDimension(R.dimen.avatar_size));
mOuterCircleColor = typedArray.getColor(R.styleable.CircleView_outer_circle_color, getResources().getColor(R.color.purple_500));
mMiddleCircleColor = typedArray.getColor(R.styleable.CircleView_middle_circle_color, getResources().getColor(R.color.purple_500));
mInnerCircleColor = typedArray.getColor(R.styleable.CircleView_inner_circle_color, getResources().getColor(R.color.purple_500));
typedArray.recycle();
```

2. 重写测量阶段相关方法（onMeasure()）  

由于不需要自定义 View 的尺寸，所以，不用重写该方法。  

3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））  

由于没有子 View 需要布局，所以，不用重写该方法。  

4. 重写绘制阶段相关方法（onDraw() 绘制主体、dispatchDraw() 绘制子 View 和 onDrawForeground() 绘制前景）  

```
//4. 重写 onDraw() 方法，自定义 View 内容
@Override
protected void onDraw(Canvas canvas) {
    mPaint.setColor(mOuterCircleColor);
    canvas.drawCircle(mRadius, mRadius, mRadius, mPaint);
    mPaint.setColor(mMiddleCircleColor);
    canvas.drawCircle(mRadius, mRadius, mRadius * 2/3, mPaint);
    mPaint.setColor(mInnerCircleColor);
    canvas.drawCircle(mRadius, mRadius, mRadius/3, mPaint);
}
```

5. onTouchEvent()  

由于 View 不需要和用户交互，所以，不用重写该方法。  

6. onInterceptTouchEvent()（仅 ViewGroup 有此方法）  

ViewGroup 的方法。  

完整代码如下：  

```
//1. 自定义属性的声明  
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!--CircleView-->
    <declare-styleable name="CircleView">
        <attr name="circle_radius" format="dimension" />
        <attr name="outer_circle_color" format="reference|color" />
        <attr name="middle_circle_color" format="reference|color" />
        <attr name="inner_circle_color" format="reference|color" />
    </declare-styleable>
</resources>

//2. CircleView  
public class CircleView extends View {

    private float mRadius;
    private int mOuterCircleColor, mMiddleCircleColor, mInnerCircleColor;
    private Paint mPaint;

    public CircleView(Context context) {
        this(context, null);
    }

    public CircleView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public CircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initData(context, attrs);
    }

    private void initData(Context context, AttributeSet attrs) {
        //1. 自定义属性的声明与获取
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
        mRadius = typedArray.getDimension(R.styleable.CircleView_circle_radius, getResources().getDimension(R.dimen.avatar_size));
        mOuterCircleColor = typedArray.getColor(R.styleable.CircleView_outer_circle_color, getResources().getColor(R.color.purple_500));
        mMiddleCircleColor = typedArray.getColor(R.styleable.CircleView_middle_circle_color, getResources().getColor(R.color.purple_500));
        mInnerCircleColor = typedArray.getColor(R.styleable.CircleView_inner_circle_color, getResources().getColor(R.color.purple_500));
        typedArray.recycle();

        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setColor(mOuterCircleColor);
    }

    //2. 重写测量阶段相关方法（onMeasure()）；
    //由于不需要自定义 View 的尺寸，所以不用重写该方法
//    @Override
//    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
//        super.onMeasure(widthMeasureSpec, heightMeasureSpec);
//    }

    //3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））；
    //由于没有子 View 需要布局，所以不用重写该方法
//    @Override
//    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
//        super.onLayout(changed, left, top, right, bottom);
//    }

    //4. 重写绘制阶段相关方法（onDraw() 绘制主体、dispatchDraw() 绘制子 View 和 onDrawForeground() 绘制前景）；
    @Override
    protected void onDraw(Canvas canvas) {
        mPaint.setColor(mOuterCircleColor);
        canvas.drawCircle(mRadius, mRadius, mRadius, mPaint);
        mPaint.setColor(mMiddleCircleColor);
        canvas.drawCircle(mRadius, mRadius, mRadius * 2/3, mPaint);
        mPaint.setColor(mInnerCircleColor);
        canvas.drawCircle(mRadius, mRadius, mRadius/3, mPaint);
    }

}

//3. 在 xml 中应用 CircleView  
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    tools:context=".custom_view_only_draw.CustomViewOnlyDrawActivity">

    <com.smart.a03_view_custom_view_example.custom_view_only_draw.CircleView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        app:circle_radius="@dimen/padding_ninety_six"
        app:inner_circle_color="@color/yellow_500"
        app:middle_circle_color="@color/cyan_500"
        app:outer_circle_color="@color/green_500" />

</LinearLayout>
```

最终效果如下：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf01057261a363~tplv-t2oaga2asx-image.image" width=360 height=640 />  

此时，即使你在 xml 中将 CircleView 的宽、高声明为「match_parent」，你会发现最终的显示效果都是一样的。  

主要原因是：默认情况下，View 的 onMeasure() 方法在通过 setMeasuredDimension() 告知父 View 自己的期望尺寸时，会调用 getDefaultSize() 方法。在 getDefaultSize() 方法中，又会调用 getSuggestedMinimumWidth() 和 getSuggestedMinimumHeight() 获取建议的最小宽度和最小高度，并根据最小尺寸和父 View 对自己的尺寸要求进行修正。最主要的是，在 getDefaultSize() 方法中修正的时候，会将 MeasureSpec.AT_MOST 和 MeasureSpec.EXACTLY 一视同仁，直接返回父 View 对 View 的尺寸要求：  

```
//1. 默认 onMeasure 的处理
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    setMeasuredDimension(getDefaultSize(getSuggestedMinimumWidth(), widthMeasureSpec),
            getDefaultSize(getSuggestedMinimumHeight(), heightMeasureSpec));
}

//2. getSuggestedMinimumWidth()
protected int getSuggestedMinimumWidth() {
    return (mBackground == null) ? mMinWidth : max(mMinWidth, mBackground.getMinimumWidth());
}

//3. getSuggestedMinimumHeight()
protected int getSuggestedMinimumHeight() {
    return (mBackground == null) ? mMinHeight : max(mMinHeight, mBackground.getMinimumHeight());
}

//4. getDefaultSize()
public static int getDefaultSize(int size, int measureSpec) {
    int result = size;
    int specMode = MeasureSpec.getMode(measureSpec);
    int specSize = MeasureSpec.getSize(measureSpec);

    switch (specMode) {
    case MeasureSpec.UNSPECIFIED:
        result = size;
        break;
    case MeasureSpec.AT_MOST:
    case MeasureSpec.EXACTLY:
        //MeasureSpec.AT_MOST、MeasureSpec.EXACTLY 一视同仁
        result = specSize;
        break;
    }
    return result;
}
```

正是因为在 getDefaultSize() 方法中处理的时候，将 MeasureSpec.AT_MOST 和 MeasureSpec.EXACTLY 一视同仁，所以才有了上面「在 xml 中应用 CircleView 的时候，无论将 CircleView 的尺寸设置为 match_parent 还是 wrap_content 效果都一样」的现象。  

具体分析如下：  

开发者对 View 的尺寸要求 | View 的父 View 对 View 的尺寸要求 | View 的期望尺寸
---|---|---
android:layout_width="wrap_content"<br/>android:layout_height="wrap_content" | MeasureSpec.AT_MOST<br/>specSize | specSize
android:layout_width="match_parent"<br/>android:layout_height="match_parent" | MeasureSpec.EXACTLY<br/>specSize | specSize

注：  
上表中，「View 的父 View 对 View 的尺寸要求」是 View 的父 View 根据「开发者对子 View 的尺寸要求」、「自己的父 View（View 的父 View 的父 View） 对自己的尺寸要求」和「自己的可用空间」计算出自己对子 View 的尺寸要求。  

另外，由执行结果可知，上表中的 specSize 实际上等于 View 的尺寸：  

```
2019-08-13 17:28:26.855 16024-16024/com.smart.a03_view_custom_view_example E/TAG: Width（getWidth()）:  1080  Height（getHeight()）:  1584
```


#### 4.1.2 自定义 View ——自定义 View 的尺寸和绘制内容  

自定义 View，它的内容是「三个半径不同、颜色不同的同心圆」，效果图如下：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf010572139870~tplv-t2oaga2asx-image.image" width=360 height=640 />  

1. 自定义属性的声明与获取  

```
//1.1 在 xml 中自定义 View 属性
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!--CircleView-->
    <declare-styleable name="CircleView">
        <attr name="circle_radius" format="dimension" />
        <attr name="outer_circle_color" format="reference|color" />
        <attr name="middle_circle_color" format="reference|color" />
        <attr name="inner_circle_color" format="reference|color" />
    </declare-styleable>
</resources>

//1.2 在 View 构造函数中获取自定义 View 属性
TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
mRadius = typedArray.getDimension(R.styleable.CircleView_circle_radius, getResources().getDimension(R.dimen.avatar_size));
mOuterCircleColor = typedArray.getColor(R.styleable.CircleView_outer_circle_color, getResources().getColor(R.color.purple_500));
mMiddleCircleColor = typedArray.getColor(R.styleable.CircleView_middle_circle_color, getResources().getColor(R.color.purple_500));
mInnerCircleColor = typedArray.getColor(R.styleable.CircleView_inner_circle_color, getResources().getColor(R.color.purple_500));
typedArray.recycle();
```

2. 重写测量阶段相关方法（onMeasure()）  

```
//2. onMeasure()
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
    //2.1 根据 View 特点或业务需求计算出 View 的尺寸
    mWidth = (int)(mRadius * 2);
    mHeight = (int)(mRadius * 2);

    //2.2 通过 resolveSize() 方法修正结果
    mWidth = resolveSize(mWidth, widthMeasureSpec);
    mHeight = resolveSize(mHeight, heightMeasureSpec);

    //2.3 通过 setMeasuredDimension() 保存 View 的期望尺寸（通过 setMeasuredDimension() 告知父 View 的期望尺寸）
    setMeasuredDimension(mWidth, mHeight);
}
```

3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））  

由于没有子 View 需要布局，所以，不用重写该方法。  

4. 重写绘制阶段相关方法（onDraw() 绘制主体、dispatchDraw() 绘制子 View 和 onDrawForeground() 绘制前景）  

```
//4. 重写 onDraw() 方法，自定义 View 内容
@Override
protected void onDraw(Canvas canvas) {
    mPaint.setColor(mOuterCircleColor);
    canvas.drawCircle(mRadius, mRadius, mRadius, mPaint);
    mPaint.setColor(mMiddleCircleColor);
    canvas.drawCircle(mRadius, mRadius, mRadius * 2/3, mPaint);
    mPaint.setColor(mInnerCircleColor);
    canvas.drawCircle(mRadius, mRadius, mRadius/3, mPaint);
}
```

5. onTouchEvent()  

由于 View 不需要和用户交互，所以，不用重写该方法。  

6. onInterceptTouchEvent()（仅 ViewGroup 有此方法）  

ViewGroup 的方法。  

完整代码如下：  

```
//1. 自定义属性的声明  
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <!--CircleView-->
    <declare-styleable name="CircleView">
        <attr name="circle_radius" format="dimension" />
        <attr name="outer_circle_color" format="reference|color" />
        <attr name="middle_circle_color" format="reference|color" />
        <attr name="inner_circle_color" format="reference|color" />
    </declare-styleable>
</resources>

//2. MeasuredCircleView
public class MeasuredCircleView extends View {

    private int mWidth, mHeight;
    private float mRadius;
    private int mOuterCircleColor, mMiddleCircleColor, mInnerCircleColor;
    private Paint mPaint;

    public MeasuredCircleView(Context context) {
        this(context, null);
    }

    public MeasuredCircleView(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public MeasuredCircleView(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initData(context, attrs);
    }

    private void initData(Context context, AttributeSet attrs) {
        //1. 自定义属性的声明与获取
        TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.CircleView);
        mRadius = typedArray.getDimension(R.styleable.CircleView_circle_radius, getResources().getDimension(R.dimen.avatar_size));
        mOuterCircleColor = typedArray.getColor(R.styleable.CircleView_outer_circle_color, getResources().getColor(R.color.purple_500));
        mMiddleCircleColor = typedArray.getColor(R.styleable.CircleView_middle_circle_color, getResources().getColor(R.color.purple_500));
        mInnerCircleColor = typedArray.getColor(R.styleable.CircleView_inner_circle_color, getResources().getColor(R.color.purple_500));
        typedArray.recycle();

        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setStyle(Paint.Style.FILL);
        mPaint.setColor(mOuterCircleColor);
    }

    //2. 重写测量阶段相关方法（onMeasure()）；
    //由于不需要自定义 View 的尺寸，所以不用重写该方法
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {
        //2.1 根据 View 特点或业务需求计算出 View 的尺寸
        mWidth = (int)(mRadius * 2);
        mHeight = (int)(mRadius * 2);

        //2.2 通过 resolveSize() 方法修正结果
        mWidth = resolveSize(mWidth, widthMeasureSpec);
        mHeight = resolveSize(mHeight, heightMeasureSpec);

        //2.3 通过 setMeasuredDimension() 保存 View 的期望尺寸（通过 setMeasuredDimension() 告知父 View 的期望尺寸）
        setMeasuredDimension(mWidth, mHeight);
    }

    //3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））；
    //由于没有子 View 需要布局，所以不用重写该方法
//    @Override
//    protected void onLayout(boolean changed, int left, int top, int right, int bottom) {
//        super.onLayout(changed, left, top, right, bottom);
//    }

    //4. 重写绘制阶段相关方法（onDraw() 绘制主体、dispatchDraw() 绘制子 View 和 onDrawForeground() 绘制前景）；
    @Override
    protected void onDraw(Canvas canvas) {
        mPaint.setColor(mOuterCircleColor);
        canvas.drawCircle(mRadius, mRadius, mRadius, mPaint);
        mPaint.setColor(mMiddleCircleColor);
        canvas.drawCircle(mRadius, mRadius, mRadius * 2/3, mPaint);
        mPaint.setColor(mInnerCircleColor);
        canvas.drawCircle(mRadius, mRadius, mRadius/3, mPaint);
    }

}

//3. 在 xml 中应用 MeasuredCircleView  
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:gravity="center"
    android:orientation="vertical"
    tools:context=".custom_view_measure_draw.CustomViewMeasureDrawActivity">

    <com.smart.a03_view_custom_view_example.custom_view_measure_draw.MeasuredCircleView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:circle_radius="@dimen/padding_ninety_six"
        app:inner_circle_color="@color/yellow_500"
        app:middle_circle_color="@color/cyan_500"
        app:outer_circle_color="@color/green_500" />
</LinearLayout>
```

最终效果如下：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf01057261a363~tplv-t2oaga2asx-image.image" width=360 height=640 />  

当在 xml 中将 MeasuredCircleView 的宽、高声明为「match_parent」时，显示效果跟 CircleView 显示效果一样。  

开发者对 View 的尺寸要求 | View 的父 View 对 View 的尺寸要求 | View 的期望尺寸
---|---|---
android:layout_width="match_parent"<br/>android:layout_height="match_parent" | MeasureSpec.EXACTLY<br/>specSize | specSize

但是，当在 xml 中将 MeasuredCircleView 的宽、高声明为「wrap_content」时，显示效果是下面这个样子：  

<img src="https://p1-jj.byteimg.com/tos-cn-i-t2oaga2asx/gold-user-assets/2019/9/2/16cf010572139870~tplv-t2oaga2asx-image.image" width=360 height=640 />  

其实，也很好理解：  

开发者对 View 的尺寸要求 | View 的父 View 对 View 的尺寸要求 | View 的期望尺寸
---|---|---
android:layout_width="wrap_content"<br/>android:layout_height="wrap_content" | MeasureSpec.AT_MOST<br/>specSize | if(childSize < specSize)  childSize<br/>if(childSize > specSize) specSize


### 4.2 自定义 ViewGroup  

自定义 ViewGroup，标签布局，效果图如下：  

<img src="http://photocq.photo.store.qq.com/psc?/V10aaGcc22Xzag/WGZ1a5bTIicW2i.M8eZ28BaND0HNsvyEwfzRLv8OQv0IuOMUUjMKenZl3B78B7u.pcvzxYItlU8hWLb2ywiZNg!!/b&bo=4AFVAwAAAAAClwQ!&rf=viewer_4" width=360 height=640 />  

无论是自定义 View 还是自定义 ViewGroup，大致的流程都是一样的：  

1. 自定义属性的声明与获取；
2. 重写测量阶段相关方法（onMeasure()）；
3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））；
4. 重写绘制阶段相关方法（onDraw() 绘制主体、dispatchDraw() 绘制子 View 和 onDrawForeground() 绘制前景）；
5. onTouchEvent()；
6. onInterceptTouchEvent()（仅 ViewGroup 有此方法）；

只不过，大多数情况下，ViewGroup 不需要「自定义属性」和「重写绘制阶段相关方法」，但有些时候还是需要的，如，开发者想在 ViewGroup 的所有子 View 上方绘制一些内容，就可以通过重写 ViewGroup 的 onDrawForeground() 来实现。  

1. 自定义属性的声明与获取  

在自定义 ViewGroup 中「自定义属性的声明与获取」的方法与在自定义 View 中「自定义属性的声明与获取」的方法一样，且因为大多数情况下，在自定义 ViewGroup 中是不需要自定义属性的，所以，在这里就不自定义属性了。  

2. 重写测量阶段相关方法（onMeasure()）  

```
//2. 重写测量阶段相关方法（onMeasure()）；
@Override
protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

    //2.1 解析 ViewGroup 的父 View 对 ViewGroup 的尺寸要求
    int widthMode = MeasureSpec.getMode(widthMeasureSpec);
    int widthSize = MeasureSpec.getSize(widthMeasureSpec);
    int heightMode = MeasureSpec.getMode(widthMeasureSpec);
    int heightSize = MeasureSpec.getSize(widthMeasureSpec);

    //2.2 ViewGroup 根据「开发者在 xml 中写的对 ViewGroup 子 View 的尺寸要求」、「自己的父 View（ViewGroup 的父 View）对自己的尺寸要求」和
    //「自己的可用空间」计算出自己对子 View 的尺寸要求，并将该尺寸要求通过子 View 的 measure() 方法传给子 View，让子 View 测量自己（View）的期望尺寸
    int widthUsed = 0;
    int heightUsed = getPaddingTop();
    int lineHeight = 0;
    int lineWidthUsed = getPaddingLeft();
    int maxRight = widthSize - getPaddingRight();

    for (int i = 0; i < getChildCount(); i++) {
        View child = getChildAt(i);
        measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, heightUsed);
        //是否需要换行
        if(widthMode != MeasureSpec.UNSPECIFIED && (lineWidthUsed + child.getMeasuredWidth() > maxRight)){
            lineWidthUsed = getPaddingLeft();
            heightUsed += lineHeight + mRowSpace;
            lineHeight = 0;
            measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, heightUsed);
        }

        //2.3 ViewGroup 暂时保存子 View 的尺寸，以便布局阶段和绘制阶段使用
        Rect childBound;
        if(mChildrenBounds.size() <= i){
            childBound = new Rect();
            mChildrenBounds.add(childBound);
        }else{
            childBound = mChildrenBounds.get(i);
        }
        //此处不能用 child.getxxx() 获取子 View 的尺寸值，因为子 View 只是量了尺寸，还没有布局，这些值都是 0
//            childBound.set(child.getLeft(), child.getTop(), child.getRight(), child.getBottom());
        childBound.set(lineWidthUsed, heightUsed, lineWidthUsed + child.getMeasuredWidth(), heightUsed + child.getMeasuredHeight());

        lineWidthUsed += child.getMeasuredWidth() + mItemSpace;
        widthUsed = Math.max(lineWidthUsed, widthUsed);
        lineHeight = Math.max(lineHeight, child.getMeasuredHeight());
    }

    //2.4 ViewGroup 将「根据子 View 的实际尺寸计算出的自己（ViewGroup）的尺寸」结合「自己父 View 对自己的尺寸要求」进行修正，并通
    //过 setMeasuredDimension() 方法告知父 View 自己的期望尺寸
    int measuredWidth = resolveSize(widthUsed, widthMeasureSpec);
    int measuredHeight = resolveSize((heightUsed + lineHeight + getPaddingBottom()), heightMeasureSpec);
    setMeasuredDimension(measuredWidth, measuredHeight);
}

//重写generateLayoutParams()
//2.2.1 在自定义 ViewGroup 中调用 measureChildWithMargins() 方法计算 ViewGroup 对子 View 的尺寸要求时，
//必须在 ViewGroup 中重写 generateLayoutParams() 方法，因为 measureChildWithMargins() 方法中用到了 MarginLayoutParams，
//如果不重写 generateLayoutParams() 方法，那调用 measureChildWithMargins() 方法时，MarginLayoutParams 就为 null，
//所以在自定义 ViewGroup 中调用 measureChildWithMargins() 方法时，必须重写 generateLayoutParams() 方法。
@Override
public LayoutParams generateLayoutParams(AttributeSet attrs) {
    return new MarginLayoutParams(getContext(), attrs);
}
```

3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））  

```
//3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））；
@Override
protected void onLayout(boolean changed, int l, int t, int r, int b) {
    for (int i = 0; i < getChildCount(); i++) {
        //应用测量阶段计算出的子 View 的尺寸值布局子 View
        View child = getChildAt(i);
        Rect childBound = mChildrenBounds.get(i);
        child.layout(childBound.left, childBound.top, childBound.right, childBound.bottom);
    }
}
```

4. 重写绘制阶段相关方法（onDraw() 绘制主体、dispatchDraw() 绘制子 View 和 onDrawForeground() 绘制前景）  

默认情况下，自定义 ViewGroup 时是不需要重写任何绘制阶段的方法的，因为 ViewGroup 的角色是容器，一个透明的容器，它只是用来盛放子 View 的。  

注意：  
- 默认情况下，系统会自动调用 View Group 的 dispatchDraw() 方法，所以不需要重写该方法；
- 出于效率的考虑，ViewGroup 默认会绕过 draw() 方法，换而直接执行 dispatchDraw()，以此来简化绘制流程。所以如果你自定义了一个 ViewGroup ，并且需要在它的除 dispatchDraw() 方法以外的任何一个绘制方法内绘制内容，你可能会需要调用 View.setWillNotDraw(false) 方法来切换到完整的绘制流程（是「可能」而不是「必须」的原因是，有些 ViewGroup 是已经调用过 setWillNotDraw(false) 了的，例如 ScrollView）。除了可以通过调用 View.setWillNotDraw(false) 方法来切换到完整的绘制流程之外，你还可以通过给 ViewGroup 设置背景来切换到完整的绘制流程。

5. onTouchEvent()  

由于 ViewGroup 不需要和用户交互，所以，不用重写该方法。  

6. onInterceptTouchEvent()（仅 ViewGroup 有此方法）  

由于 ViewGroup 不需要和用户交互且 ViewGroup 不需要拦截子 View 的 MotionEvent，所以，不用重写该方法。  

完整代码如下：  

```
//1. TabLayout
public class TabLayout extends ViewGroup {

    private ArrayList<Rect> mChildrenBounds;
    private int mItemSpace;
    private int mRowSpace;

    public TabLayout(Context context) {
        this(context, null);
    }

    public TabLayout(Context context, AttributeSet attrs) {
        this(context, attrs, 0);
    }

    public TabLayout(Context context, AttributeSet attrs, int defStyleAttr) {
        super(context, attrs, defStyleAttr);
        initData();
    }

    private void initData(){
        mChildrenBounds = new ArrayList<>();
        mItemSpace = (int)getResources().getDimension(R.dimen.padding_small);
        mRowSpace = (int)getResources().getDimension(R.dimen.padding_small);
    }

    //2. 重写测量阶段相关方法（onMeasure()）；
    @Override
    protected void onMeasure(int widthMeasureSpec, int heightMeasureSpec) {

        //2.1 解析 ViewGroup 的父 View 对 ViewGroup 的尺寸要求
        int widthMode = MeasureSpec.getMode(widthMeasureSpec);
        int widthSize = MeasureSpec.getSize(widthMeasureSpec);
        int heightMode = MeasureSpec.getMode(widthMeasureSpec);
        int heightSize = MeasureSpec.getSize(widthMeasureSpec);

        //2.2 ViewGroup 根据「开发者在 xml 中写的对 ViewGroup 子 View 的尺寸要求」、「自己的父 View（ViewGroup 的父 View）对自己的尺寸要求」和
        //「自己的可用空间」计算出自己对子 View 的尺寸要求，并将该尺寸要求通过子 View 的 measure() 方法传给子 View，让子 View 测量自己（View）的期望尺寸
        int widthUsed = 0;
        int heightUsed = getPaddingTop();
        int lineHeight = 0;
        int lineWidthUsed = getPaddingLeft();
        int maxRight = widthSize - getPaddingRight();

        for (int i = 0; i < getChildCount(); i++) {
            View child = getChildAt(i);
            measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, heightUsed);
            //是否需要换行
            if(widthMode != MeasureSpec.UNSPECIFIED && (lineWidthUsed + child.getMeasuredWidth() > maxRight)){
                lineWidthUsed = getPaddingLeft();
                heightUsed += lineHeight + mRowSpace;
                lineHeight = 0;
                measureChildWithMargins(child, widthMeasureSpec, 0, heightMeasureSpec, heightUsed);
            }

            //2.3 ViewGroup 暂时保存子 View 的尺寸，以便布局阶段和绘制阶段使用
            Rect childBound;
            if(mChildrenBounds.size() <= i){
                childBound = new Rect();
                mChildrenBounds.add(childBound);
            }else{
                childBound = mChildrenBounds.get(i);
            }
            //此处不能用 child.getxxx() 获取子 View 的尺寸值，因为子 View 只是量了尺寸，还没有布局，这些值都是 0
//            childBound.set(child.getLeft(), child.getTop(), child.getRight(), child.getBottom());
            childBound.set(lineWidthUsed, heightUsed, lineWidthUsed + child.getMeasuredWidth(), heightUsed + child.getMeasuredHeight());

            lineWidthUsed += child.getMeasuredWidth() + mItemSpace;
            widthUsed = Math.max(lineWidthUsed, widthUsed);
            lineHeight = Math.max(lineHeight, child.getMeasuredHeight());
        }

        //2.4 ViewGroup 将「根据子 View 的实际尺寸计算出的自己（ViewGroup）的尺寸」结合「自己父 View 对自己的尺寸要求」进行修正，并通
        //过 setMeasuredDimension() 方法告知父 View 自己的期望尺寸
        int measuredWidth = resolveSize(widthUsed, widthMeasureSpec);
        int measuredHeight = resolveSize((heightUsed + lineHeight + getPaddingBottom()), heightMeasureSpec);
        setMeasuredDimension(measuredWidth, measuredHeight);
    }

    //2.2.1 在自定义 ViewGroup 中调用 measureChildWithMargins() 方法计算 ViewGroup 对子 View 的尺寸要求时，
    //必须在 ViewGroup 中重写 generateLayoutParams() 方法，因为 measureChildWithMargins() 方法中用到了 MarginLayoutParams，
    //如果不重写 generateLayoutParams() 方法，那调用 measureChildWithMargins() 方法时，MarginLayoutParams 就为 null，
    //所以在自定义 ViewGroup 中调用 measureChildWithMargins() 方法时，必须重写 generateLayoutParams() 方法。
    @Override
    public LayoutParams generateLayoutParams(AttributeSet attrs) {
        return new MarginLayoutParams(getContext(), attrs);
    }

    //3. 重写布局阶段相关方法（onLayout()（仅 ViewGroup 需要重写））；
    @Override
    protected void onLayout(boolean changed, int l, int t, int r, int b) {
        for (int i = 0; i < getChildCount(); i++) {
            //应用测量阶段计算出的子 View 的尺寸值布局子 View
            View child = getChildAt(i);
            Rect childBound = mChildrenBounds.get(i);
            child.layout(childBound.left, childBound.top, childBound.right, childBound.bottom);
        }
    }

    @Override
    public boolean onInterceptHoverEvent(MotionEvent event) {
        return super.onInterceptHoverEvent(event);
    }

    @Override
    public boolean onTouchEvent(MotionEvent event) {
        return super.onTouchEvent(event);
    }
}

//2. 在 xml 中应用 TabLayout
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:scrollbars="none"
    tools:context=".MainActivity">

    <com.smart.a04_view_custom_viewgroup_example.custom_layout.TabLayout
        android:id="@+id/tag_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/grey_400"
        android:padding="@dimen/padding_small">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@drawable/common_bg"
            android:text="@string/spending_clothes" />

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@drawable/common_bg"
            android:text="@string/spending_others" />

        ...

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:background="@drawable/common_bg"
            android:text="@string/november" />

    </com.smart.a04_view_custom_viewgroup_example.custom_layout.TabLayout>

</ScrollView>
```

最终效果如下：  

<img src="http://photocq.photo.store.qq.com/psc?/V10aaGcc22Xzag/WGZ1a5bTIicW2i.M8eZ28BaND0HNsvyEwfzRLv8OQv0IuOMUUjMKenZl3B78B7u.pcvzxYItlU8hWLb2ywiZNg!!/b&bo=4AFVAwAAAAAClwQ!&rf=viewer_4" width=360 height=640 />  


## 5. 相关问题  

### 5.1 大方向  

1. Activity、Window、View 之间的关系
2. View 是如何显示出来的？
    - View 是如何显示出来的？
    - View 新增子 View 的时候是将子 View 添加到原来的 View Tree，那 Toast 显示的时候呢？它是怎样显示的？
3. View（ViewGroup） 布局、绘制流程
4. View（ViewGroup） 事件分发


### 5.2 小细节  

1. 用过 View 中的 onSaveInstanceState()/onRestoreInstanceState() 吗？一般在什么情况下使用？
2. onMeasure() 会执行多次吗？为什么？举例说明
    - 能手动触发吗？如果能，怎么做？如果能触发，会出现什么情况？
3. onLayout() 会执行多次吗？为什么？
    - 能手动触发吗？如果能，怎么做？如果能触发，会出现什么情况？
4. onDraw() 会执行多次吗？为什么？
    - 能手动触发吗？如果能，怎么做？如果能触发，会出现什么情况？
5. requestLayout() 作用、使用场景、注意事项
6. invalidate() 作用、使用场景、注意事项
7. postInvalidate() 作用、使用场景、注意事项
8. invalidate()、postInvalidate() 异同
9. scrollBy、scrollTo 作用、使用场景、注意事项、二者的区别


### 5.3 如何优化自定义 View？  

1. 如何优化自定义 View？
2. 如何优化自定义 ViewGroup？


## 6. 如何拓展？  

1. 结合 Drawable
2. 结合动画，让 View 的内容变化显得更加流畅


## 7. 总结  

自定义 View 包括三部分内容：

- 布局（Layout）
- 绘制（Drawing）
- 触摸反馈（Event Handling）

其中布局阶段确定了 View 的位置和尺寸，该阶段主要是为了后面的绘制和触摸反馈做支持；绘制阶段主要用于绘制 View 的内容（大多数情况下，只用实现 OnDraw 方法（Where）方法、按照指定顺序调用相关 API（How）即可实现自定义绘制（What））；触摸反馈阶段确定了用户点击了哪里，三者相辅相成，缺一不可。  


---
## 参考文档  
1. [View](https://developer.android.com/reference/android/view/View)
2. [ViewGroup](https://developer.android.com/reference/android/view/ViewGroup)
3. [HenCoder](https://hencoder.com)
4. [Android面试解密-自定义View](https://www.imooc.com/learn/579)

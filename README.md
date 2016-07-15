# iOS-Core-Animation-Study-Notes
iOS 核心动画学习笔记

---
##1.图层树
###1.1.图层与视图
  在`iOS`当中，所有的视图都从一个叫做`UIVIew`的基类派生而来，`UIView`可以处理触摸事件，可以支持基于`Core Graphics`绘图，可以做仿射变换（例如旋转或者缩放），或者简单的类似于滑动或者渐变的动画。
	
```CALayer```类在概念上和```UIView```类似，同样也是一些被层级关系树管理的矩形块，同样也可以包含一些内容（像图片，文本或者背景色），管理子图层的位置。它们有一些方法和属性用来做动画和变换。和```UIView```最大的不同是```CALayer``` ***不处理用户的交互***。

每一个```UIview```都有一个```CALayer```实例的图层属性，也就是```.layer```属性,他们是平行级别关系。

###1.2.图层的能力
  图层`layer`可以帮你做很多`UIView`做不了的事
 
 * 阴影，圆角，带颜色的边框
 * 3D变换
 * 非矩形范围
 * 透明遮罩
 * 多级非线性动画
 
###1.3.使用图层
给视图添加一个蓝色子图层

```	
	//create sublayer
    CALayer *blueLayer = [CALayer layer];
    blueLayer.frame = CGRectMake(50.0f, 50.0f, 100.0f, 100.0f);
    blueLayer.backgroundColor = [UIColor blueColor].CGColor;
    //add it to our view
    [self.layerView.layer addSublayer:blueLayer];
    
```
###1.4.总结
这一章阐述了图层的树状结构，说明了如何在iOS中由`UIView`的层级关系形成的一种平行的`CALayer`层级关系，在后面的实验中，我们创建了自己的`CALayer`，并把它添加到图层树中。
	
===
##寄宿图
###2.1.contents属性


`CALayer`有个`contents`的属性，这个属性是id类型的，也就是说可以***添加图片***等任何对象。

```
//load an image
UIImage *image = [UIImage imageNamed:@"Snowman.png"];

//add it directly to our view's layer
self.layerView.layer.contents = (__bridge id)image.CGImage;

```


####2.1.2.contentsGravity

如果图片有点变形，```CALayer```与`contentMode`对应的属性叫做`contentsGravity`，但是它是一个NSString类型，而不是像对应的UIKit部分，那里面的值是枚举。

`self.layerView.layer.contentsGravity = kCAGravityResizeAspect;`


####2.1.3.contentsScale
`contentsScale`属性定义了寄宿图的像素尺寸和视图大小的比例，默认情况下它是一个值为1.0的浮点数。

`contentsScale`的目的并不是那么明显。它并不是总会对屏幕上的寄宿图有影响。如果你尝试对我们的例子设置不同的值，你就会发现根本没任何影响。因为`contents`由于设置了`contentsGravity`属性，所以它已经被拉伸以适应图层的边界。


```
	//load an image
	UIImage *image = [UIImage imageNamed:@"Snowman.png"]; //add it directly to our view's layer
	self.layerView.layer.contents = (__bridge id)image.CGImage; //center the image
	self.layerView.layer.contentsGravity = kCAGravityCenter; //这个值并不会拉伸图片
	//set the contentsScale to match image
	self.layerView.layer.contentsScale = image.scale;
```
	
当用代码的方式来处理寄宿图的时候，一定要记住要手动的设置图层的`contentsScale`属性，否则，你的图片在Retina设备上就显示得不正确啦。代码如下：
`layer.contentsScale = [UIScreen mainScreen].scale;`


####2.1.4.maskToBounds
`UIView`有一个叫做`clipsToBounds`的属性可以用来决定是否显示超出边界的内容，`CALayer`对应的属性叫做`masksToBounds`，把它设置为YES，雪人就在边界里啦～


####2.1.5.contentsRect
`CALayer`的`contentsRect`属性允许我们在图层边框里显示寄宿图的一个子域。这涉及到图片是如何显示和拉伸的，所以要比`contentsGravity`灵活多了

和`bounds`，`frame`不同，`contentsRect`不是按点来计算的，它使用了单位坐标，单位坐标指定在0到1之间，是一个相对值（像素和点就是绝对值）。所以他们是相对与寄宿图的尺寸的。

```
@interface ViewController ()
@property (nonatomic, weak) IBOutlet UIView *coneView;
@property (nonatomic, weak) IBOutlet UIView *shipView;
@property (nonatomic, weak) IBOutlet UIView *iglooView;
@property (nonatomic, weak) IBOutlet UIView *anchorView;
@end

@implementation ViewController


- (void)addSpriteImage:(UIImage *)image withContentRect:(CGRect)rect ￼toLayer:(CALayer *)layer //set image
{
  layer.contents = (__bridge id)image.CGImage;

  //scale contents to fit
  layer.contentsGravity = kCAGravityResizeAspect;

  //set contentsRect
  layer.contentsRect = rect;
}

- (void)viewDidLoad 
{
  [super viewDidLoad]; //load sprite sheet
  UIImage *image = [UIImage imageNamed:@"Sprites.png"];
  //set igloo sprite
  [self addSpriteImage:image withContentRect:CGRectMake(0, 0, 0.5, 0.5) toLayer:self.iglooView.layer];
  //set cone sprite
  [self addSpriteImage:image withContentRect:CGRectMake(0.5, 0, 0.5, 0.5) toLayer:self.coneView.layer];
  //set anchor sprite
  [self addSpriteImage:image withContentRect:CGRectMake(0, 0.5, 0.5, 0.5) toLayer:self.anchorView.layer];
  //set spaceship sprite
  [self addSpriteImage:image withContentRect:CGRectMake(0.5, 0.5, 0.5, 0.5) toLayer:self.shipView.layer];
}
@end

```
	
	

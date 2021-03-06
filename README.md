# TurntableWithCA
用核心动画实现一个转盘，just a small demo。
![image](https://github.com/LiDami/TurntableWithCA/blob/master/show.gif)

### 1,搭建界面
		把转盘View给封装起来. 由于界面是固定不变的,可以弄一个Xib展示界面.
		外界使用时直接来一个类方法直接调用.
### 	2.让转盘进行旋转.
		在封装的View内部提供一个开始旋转的方法和结束旋转的方法,供外界直接调用.
		在View内部实现方法.
		开始旋转:
			添加核心动画.动画要添加到里面的背景图片上.不能够直接添加到View.
### 3.添加按钮.
		1.分析按钮的位置.每一个都是倾斜的.想要办到这种效果,可以先把所有的按钮都给添加到圆盘的最上方
		然后让每一个按钮绕着圆盘的中心点进行旋转.
		想要让按钮绕着圆盘中心点进行旋转,要选把每个按钮的锚点设置到按钮底部中心位置(0.5,1)
		再让按钮显示到中心的位置,设置按钮的position为圆盘的中心即可.
		
		2.添加按钮时,让每一个按钮进行旋转,要计算出每一个按钮旋转的角度.总共有12个按钮,正好占圆的一圈,
		  那每一个按钮占30.那一个按钮在上一个按钮的基础上加上30.即可算出每一个按钮旋转的角度.
		  
### 	4.给按钮添加点击事件.让按钮成为选中状态,取消按钮的高亮状态.
		1.在添加按钮时, 给每一个按钮添加选中状态.
		  添加按钮事件的时候要注意.我们每一个按钮都是添加到UIImageView上的.UIImageView默认是不接收事件的.
		  所以需要手动把UIImageView的设置为可接收事件.
		  
		2.让按钮成为选中状态.
		  在事件处理方法当中,我们要先定义一个成员属性记录上一个按钮的选中状态.
		  先把上一个按钮的选中状态取消选中.
		  让当前点击的按钮成为选中状态.
		  把当前的按钮赋值上一个按钮.
		  
### 	5.设置按钮每一个按钮的图片
			
		1.查看美工提供的资料文件.发现所有的按钮都在一张图片上面.
		  这个时候我们就得需要把这一张图片做一个裁剪.
		  首先要确定裁剪的每一张图片的宽,高
		  每一张图片的高度, 为原始图片的高度
		  每一张图片的宽度, 为整个图片的宽度除以总共要裁剪多少张图片.这里我们总共有12张图片, 所以总宽度除以12.
		  用下面这个方法做裁剪.
		  参数说明:
		  image:为要裁剪的图片,既原始图片.它的类型为CGImageRef,类型,所以要转成CGImage
		  rect:为裁剪的范围.这时需要确定每一个X的位置.
		  每一个x为它当前的角标 * 要裁剪的宽度.
		  CGImageCreateWithImageInRect(image, rect);
		  
		  这个方法它会返回一个图片.图片类型为CGImageRef imgR.
		  所要我们在设置背景图片的时候,要把这张图片给转成UIImage.
		  转的方法为:
		  UIImage *image = [UIImage imageWithCGImage:imgR];
		  
	   2.运行时只能看见图片的一部分
	   	  运行的时候只能看到图片的一部分.这个是因为我们加载图片,加载的是@2X的图片.
	   	  而我们CGImageCreateWithImageInRect这个方法它是C语言的方法.它裁剪的区域是一个像素点.
	   	  所以导致我们看到的只有这一部分内容
	   	  解决办法为把裁剪的宽高都乘上一个像素比例.
	   	  获得像素比例
	   	  [UIScreen mainScreen].scale
	   	  这样裁剪出来的图片就是整个图片了.
		 
	   3.但是运行的时候还是发现裁剪的图片尺寸比较大.
	   	  这个时候我们要修改按钮内部图片的尺寸大小.
	   	  在按钮当中实现一个方法.在这个方法当中重新计算图片的尺寸
`	   	  -(CGRect)imageRectForContentRect:(CGRect)contentRect{
    
		    CGFloat btnW = 40;
    		CGFloat btnH = 47;
    		CGFloat btnY = 20;
		    CGFloat btnX = (contentRect.size.width - btnW) * 0.5;
		    return CGRectMake(btnX, btnY, btnW, btnH);

		  }	`
		  
		  
### 	6.在按钮旋转的时候,还要点击按钮所以这个时候就不能够用核心动画.
		这个时候我们就要用UIView动画.
		
		方法为:添加一个定时器, 每次在原来基础上添加一个旋转角度.
		定时器我们只需要添加一次, 我们就采用懒加载的方法.我们采用CADisplayLink的方法.
		因为这个定时器可以让它很方便的暂停,开始，这就是与NSTimer的区别.
`		
		-(CADisplayLink *)link{
    
    	if (_link == nil) {
        _link = [CADisplayLink displayLinkWithTarget:self selector:@selector(rotationChange)];
        [_link addToRunLoop:[NSRunLoop mainRunLoop] forMode:NSDefaultRunLoopMode];
    	}
    		return _link;
    	}`
    	
      在定时器方法当中,做一个旋转,每次让它在原来的基础上旋转一个角度.
      
      
###     7.实现中间点击按钮.
      中间点击按钮为逻辑为,快速的让圆盘旋转几圈,动画完成时,让选择的按钮指上最上方.
      快速旋转几圈, 这个时候不需要与用户进行交互,所以我们可以使用核心动画.
      在动画开始的时候,停止定时器.
      得要监听动画完成.在动画完成的时候,来做事情.
      想要坚听动画的完成.我们可以设置核心动画的代理.
      
      核心动画代理方法比较特殊.它不需要遵守协议,它是NSObject的一个分类.也称它是非常式协议.
      直接实现它的方法就可以了.
      
      
      在这个方法当中要实现的思路为.
      看当前选中的按钮之前是旋转多少度旋转到当前位置的.
      然后再让转盘倒着旋转回去就可以了.
      
      
      可以通过当前按钮的transform求出当前按钮的旋转角度
      方法为:
`      CGFloat angel = atan2(self.selectBtn.transform.b, self.selectBtn.transform.a);  
* 解释：atan2函数，atan2(y,x)，x,y为坐标，获取角度
      
      -(void)animationDidStop:(nonnull CAAnimation *)anim finished:(BOOL)flag{
      
		 CGAffineTransform transform = self.selectBtn.transform;
		 CGFloat angel = atan2(self.selectBtn.transform.b, self.selectBtn.transform.a);
		 self.innerCircle.transform = CGAffineTransformMakeRotation(-angel);
		 
	  }`
##layoutSubviews

###官方文档

>* The default implementation of this method does nothing on iOS 5.1 and earlier. Otherwise, the default implementation uses any constraints you have set to determine the size and position of any subviews.

>* Subclasses can override this method as needed to perform more precise layout of their subviews. You should override this method only if the autoresizing and constraint-based behaviors of the subviews do not offer the behavior you want. You can use your implementation to set the frame rectangles of your subviews directly.

>* You should not call this method directly. If you want to force a layout update, call the setNeedsLayout method instead to do so prior to the next drawing update. If you want to update the layout of your views immediately, call the layoutIfNeeded method.

>---
####Tips:
1. `iOS5.1和更早版本`该方法的实现不会执行任何操作,`iOS5.1之后版本`实现该方法会依照外部已确定的子视图大小和位置的约束进行布局
2. 子类可以重写该方法来进行更精确的布局子控件，通过在此方法中直接调整控件大小和约束为所需的
3. 官方强调不要直接调用`layoutSubviews`,想要做布局更新的话，官方建议使用`setNeedsLayout`而非绘制更新,想要立即更新视图布局的话，官方建议使用`layoutIfNeeded`

###刷新布局

`-setNeedsLayout`标记为需要重新布局，异步调用layoutIfNeeded刷新布局，不立即刷新，但layoutSubviews一定会被调用</br>
`-layoutIfNeeded`有需要刷新的标记，立即调用layoutSubviews进行布局（如果没有标记，不会调用layoutSubviews）</br>
如果要立即刷新，要先调用[view setNeedsLayout]，把标记设为需要布局，然后马上调用[view layoutIfNeeded]，实现布局</br>
在视图第一次显示之前(frame不为CGRectZero)，标记总是“需要刷新”的，可以直接调用[view layoutIfNeeded]

* ####layoutSubviews方法调用先于drawRect
`[self.view addSubview:self.oneView];`
> ---[ViewController One]--</br>
> ---[OneView layoutSubviews]--</br>
> ---[OneView drawRect:]--

* ####setNeedsLayout在receiver标上一个需要被重新布局的标记，在系统runloop的下一个周期自动调用layoutSubviews
`[self.oneView setNeedsLayout];`
> ---[ViewController Two]--</br>
> ---[OneView layoutSubviews]--

* ####layoutIfNeeded方法如其名，UIKit会判断subviews链中receiver是否需要layout，需要，则立即执行
`[self.oneView setNeedsLayout];`</br>
`[self.oneView layoutIfNeeded];`
>---[OneView layoutSubviews]--</br>
>---[ViewController Two]--

###layoutSubviews调用情况

* ####init初始化不会触发layoutSubviews
`OneView *oneView = [[OneView alloc] init];`</br>
`oneView.backgroundColor = KRandomColor;`</br>
`[self.view addSubview:oneView];`
>---[ViewController One]--

* ####initWithFrame 进行初始化时，当rect的值不为CGRectZero才会触发
`OneView *oneView = [[OneView alloc] initWithFrame:CGRectZero];`</br>
 `oneView.backgroundColor = KRandomColor;`</br>
 `[self.view addSubview:oneView];`
>---[ViewController One]--

	`OneView *oneView = [[OneView alloc] initWithFrame:CGRectMake(0, 340, 100, 100)];`</br>
` oneView.backgroundColor = KRandomColor;`</br>
`[self.view addSubview:oneView];`
>---[ViewController One]--</br>
>---[OneView layoutSubviews]--
    
* ####addSubview会触发layoutSubviews

	__1. 设置OneView的rect不为CGRectZero,不调用父类addSubview</br>__
	`OneView *oneView = [[OneView alloc] initWithFrame:CGRectMake(0, 340, 100, 100)];`</br>
    `oneView.backgroundColor = KRandomColor;`
    
    >---[ViewController One]--
    
	__2. 设置OneView的rect不为CGRectZero后立即调用layoutIfNeeded,不调用父类addSubview</br>__
	`OneView *oneView = [[OneView alloc] initWithFrame:CGRectMake(0, 340, 100, 100)];`</br>
    `oneView.backgroundColor = KRandomColor;`<br>
    `[oneView layoutIfNeeded];`
    
    >---[OneView layoutSubviews]--</br>
	>---[ViewController One]--
	
	__3. 设置OneView的rect为CGRectZero后立即调用layoutIfNeeded,不调用父类addSubview</br>__
	`OneView *oneView = [[OneView alloc] initWithFrame:CGRectZero];`</br>
	`oneView.backgroundColor = KRandomColor;`</br>
    `[oneView layoutIfNeeded];`
    
    >---[ViewController One]--
    
    __4. 设置OneView的rect不为CGRectZero后立即调用layoutIfNeeded,延迟3.0f再调用父类addSubview</br>__
    `OneView *oneView = [[OneView alloc] initWithFrame:CGRectMake(0, 340, 100, 100)];`</br>
    `oneView.backgroundColor = KRandomColor;`</br>
    `[oneView layoutIfNeeded];`</br>
    `self.oneView = oneView;`</br>
    `[self performSelector:@selector(Two) withObject:nil afterDelay:3.0f];`
    
    ---
    `[self.view addSubview:self.oneView];`
    
    >---[OneView layoutSubviews]--</br>
	>---[ViewController One]--</br>
	>---[ViewController Two]--</br>
	>---[OneView drawRect:]--

	__5. 设置twoView的rect不为CGRectZero,调用addSubview</br>__
	`OneView *oneView = [[OneView alloc] initWithFrame:CGRectMake(0, 340, 100, 100)];`</br>
    `oneView.backgroundColor = KRandomColor;`</br>
    `[oneView layoutIfNeeded];`</br>
    `[self.view addSubview:oneView];`</br>
    `self.oneView = oneView;`</br>
    `[self performSelector:@selector(Two) withObject:nil afterDelay:3.0f];`</br>
    
    ---
    `TwoView *twoView = [[TwoView alloc] initWithFrame:CGRectMake(10, 10, 10, 10)];`</br>
    `[self.oneView addSubview:twoView];`</br>
    
    >---[OneView layoutSubviews]--</br>
	>---[ViewController One]--</br>
	>---[OneView drawRect:]--</br>
	>---[ViewController Two]--</br>
	>---[OneView layoutSubviews]--</br>
	>---[TwoView layoutSubviews]--</br>
	>---[TwoView drawRect:]--</br>

* ####设置view的Frame会触发layoutSubviews，当然前提是frame的值设置前后发生了变化
	`OneView *oneView = [[OneView alloc] initWithFrame:CGRectMake(0, 340, 100, 100)];`</br>
    `oneView.backgroundColor = KRandomColor;`</br>
    `[self.view addSubview:oneView];`</br>
    `self.oneView = oneView;`</br>
    `[self.oneView addSubview:self.twoView];`</br>
    
    `[self performSelector:@selector(Two) withObject:nil afterDelay:3.0f];`</br>
    
    ---
    `self.oneView.frame = CGRectZero;`
    
    >---[ViewController One]--</br>
	>---[OneView layoutSubviews]--</br>
	>---[TwoView layoutSubviews]--</br>
	>---[OneView drawRect:]--</br>
	>---[TwoView drawRect:]--</br>
	>---[ViewController Two]--</br>
	>---[OneView layoutSubviews]--
	
* ####滚动一个UIScrollView会触发layoutSubviews
	`OneView为UIScrollView子类，滚动其会不断触发layoutSubviews`
	
	>---[OneView layoutSubviews]--
* ####旋转Screen会触发父UIView上的layoutSubviews事件
	`OneView *oneView = [[OneView alloc] initWithFrame:CGRectMake(0, 340, 100, 100)];`</br>
    `oneView.backgroundColor = KRandomColor;`</br>
    `[self.view addSubview:oneView];`</br>
    `self.oneView = oneView;`</br>
    `[self.oneView addSubview:self.twoView];`</br>
    
    ---
    `ThreeView *threeView = [[ThreeView alloc] init];`</br>
    `threeView.frame = CGRectMake(200, 440, 100, 100);`</br>
    `threeView.backgroundColor = KRandomColor;`</br>
    `[self.oneView addSubview:threeView];`
    
    >---[OneView layoutSubviews]--</br>
	>---[ThreeView layoutSubviews]--</br>
	>---[TwoView layoutSubviews]--</br>
	>---[OneView layoutSubviews]--</br>
	>---[ThreeView layoutSubviews]--</br>
	>---[TwoView layoutSubviews]--	</br>
	>---fromInterfaceOrientation==3==toInterfaceOrientation==1
	
* ####改变一个UIView大小的时候也会触发父UIView上的layoutSubviews事件
	`[self.oneView addSubview:self.twoView];`</br>
	`[self.oneView addSubview:self.threeView];`</br>
	
	---
	`self.twoView.frame = CGRectZero;`</br>
	`self.threeView.frame = CGRectMake(200, 340, 100, 200);`
	
	>---[ViewController Two]--</br>
	>---[OneView layoutSubviews]--</br>
	>---[TwoView layoutSubviews]--</br>
	>---[ViewController Four]--</br>
	>---[OneView layoutSubviews]--</br>
	>---[ThreeView layoutSubviews]—


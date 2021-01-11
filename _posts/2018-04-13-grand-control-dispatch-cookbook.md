---
title:  "Grand Control Dispatch Cookbook"
date:   2018-04-13 20:01:17 +0800
toc: true
toc_label: "Table Of Content"
toc_icon: "columns"  # corresponding Font Awesome icon name (without fa prefix)
---

Grand Control Dispatch的常见实用场景。

## 1. 同步转异步

最常见的用法，耗时长的操作放在Global Queue中执行，完成之后将结果交还给主线程处理（特别是UI操作）：

{% highlight objective_c linenos %}
dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_BACKGROUND, 0), ^{
	id result = [self longTimeTask];
	dispatch_async(dispatch_get_main_queue(), ^{
		[self handleResultOnMainThread:result];
	});
});
{% endhighlight %}

## 2. 异步转同步

注意：如果当前是在主线程的话，会导致UI流程卡顿。

{% highlight objective_c linenos %}
dispatch_semaphore_t semaphore = dispatch_semaphore_create(0);
[self someAsyncTaskWithCallback:^(BOOL successed) {
      dispatch_semaphore_signal(semaphore);
}];

dispatch_semaphore_wait(sema, DISPATCH_TIME_FOREVER);
{% endhighlight %}

## 3. 延时操作

{% highlight objective_c linenos %}
dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(1* NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
	[self performOnMainThread];
});
{% endhighlight %}

## 4. 单例

为类增加一个类方法，后续访问单例就用`MyClass.sharedIncstance`。

注意，`MyClass`的初始化方法里面`init`，**一定不要**重复调用`MyClass.sharedIncstance`，或者与其他单例初始化互相调用，会造成循环引用。

{% highlight objective_c linenos %}
+ (instancetype)sharedInstance{
    static dispatch_once_t once;
    static MyClass *sharedInstance;

    dispatch_once(&once, ^
    {
        sharedInstance = [[MyClass alloc] init];
    });
       
    return sharedInstance;
}
{% endhighlight %}

## 5. 等待多个异步任务全部完成（多个任务的等待）

{% highlight objective_c linenos %}
dispatch_group_t group = dispatch_group_create();

dispatch_group_async(group,dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^ {
	dispatch_group_enter(group);
	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
		NSLog(@"Block1a %@",[NSThread currentThread]);
		sleep(5);
		dispatch_group_leave(group);
		NSLog(@"Block1b %@",[NSThread currentThread]);
	});
});

dispatch_group_async(group,dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^ {
	dispatch_group_enter(group);
	dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^{
		NSLog(@"Block2a %@",[NSThread currentThread]);
		sleep(3);
		dispatch_group_leave(group);
		NSLog(@"Block2b %@",[NSThread currentThread]);
	});
});

dispatch_group_notify(group,dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_HIGH, 0), ^ {
	NSLog(@"Block3 %@",[NSThread currentThread]);
});
{% endhighlight %}

## 6. 控制并发数（任务池）

比如计划下载任务有a个，但同一时间内，只有b个在工作中（b<a）。每当某个工作中的任务完成，就有一个新的计划下载任务开始执行。起到控制阀的效果。

{% highlight objective_c linenos %}
dispatch_queue_t queue = dispatch_queue_create("queue", DISPATCH_QUEUE_CONCURRENT);

dispatch_semaphore_t semaphore = dispatch_semaphore_create(5);

dispatch_group_t group = dispatch_group_create();

for (int i = 0; i < 10; i++) {
    dispatch_group_async(group, queue, ^{
        dispatch_semaphore_wait(semaphore, DISPATCH_TIME_FOREVER);
        NSLog(@"%d --- begin ---", i + 1);
        // 线程操作区域 最多有有限个线程在此做事情
        sleep(arc4random()%5+2);
        
        NSLog(@"%d --- end ---", i + 1);
        
        dispatch_semaphore_signal(semaphore);
    });
}

// group任务全部执行完毕回调
dispatch_group_notify(group, queue, ^{
    NSLog(@"all done");
});
{% endhighlight %}

## 7. 任务插队

需要将特定的任务在一个队列的中间位置执行。可以用`dispatch_barrier_async`或者`dispatch_barrier_sync`，两者的区别是，后者会阻塞当前线程。另，根据文档，`queue`必须是`dispatch_queue_create`创建的并行队列，如果是串行队列或者全局并行队列，函数的表现与`dispatch_async`或者`dispatch_sync`一样，也就是插队失败。

{% highlight objective_c linenos %}
dispatch_queue_t queue = dispatch_queue_create("thread", DISPATCH_QUEUE_CONCURRENT);
dispatch_async(queue, ^{
    sleep(3);
    NSLog(@"1");
});
dispatch_async(queue, ^{
    NSLog(@"2");
});

dispatch_barrier_async(queue, ^{  // 这里替换为dispatch_barrier_sync
    sleep(10);
    NSLog(@"barrier, %d",[NSThread isMainThread]);
});

dispatch_async(queue, ^{
    NSLog(@"3");
});

dispatch_async(queue, ^{
    NSLog(@"4");
});
{% endhighlight %}
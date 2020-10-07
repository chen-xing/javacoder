###ideat debug技巧
####断点回退
>错过了关键的断点或者想重新debug错过的信息，难道要重新开始？
>![](http://dl2.iteye.com/upload/attachment/0103/8081/eb1942f2-5d10-3d8d-a20e-48ac91784d6b.jpg)
>
>A、       标识1，表示回退到调用栈的上一级。如现在执行到26行，点1图标，则回退到21行，再点1图标则继续回退到12行。注意，回退到方法调用处时，现场也会回退到调用前的状态（即下面的debugger和variables等窗口中的值会变成调用前）。
B、         2表示直接运行到光标处断点，适合于临时断点。

####断点过滤
>循环中debug难道一直单步下去？
####实例过滤
![](http://dl2.iteye.com/upload/attachment/0103/8083/f80dfaa9-4c4b-3beb-9c6d-84fe29b775cf.jpg)
![](http://dl2.iteye.com/upload/attachment/0103/8109/def8823c-5092-339b-8de7-53d5036797a0.jpg)

####class过滤器
![](http://dl2.iteye.com/upload/attachment/0103/8088/627485db-ee04-3f95-9735-58f54d8d7870.jpg)
![](http://dl2.iteye.com/upload/attachment/0103/8088/627485db-ee04-3f95-9735-58f54d8d7870.jpg)

####condition过滤器
![](http://dl2.iteye.com/upload/attachment/0103/8115/dc82f548-a12d-35d4-b41d-20719db44a8e.jpg)


####其他
![](http://dl2.iteye.com/upload/attachment/0103/8094/e3015daa-1022-3558-8997-1b9632632636.jpg)


##总结
>###1、drop frame
    >回退到方法调用处。在错过关键代码调试，需要重新调试时，可考虑回退，而不用再次发起请求。
>###2、断点过滤
    >适用集合迭代处理时，需要调试集合中某个对象的处理过程，可使用断点过滤。
>###3、属性值改变时调试
    >使用简单，只要在类的属性设置断点即可。
    适用于返回某个值不正确时，可快速定位值改变的代码位置。


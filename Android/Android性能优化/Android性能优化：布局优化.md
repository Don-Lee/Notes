[Android性能优化：布局优化](https://blog.csdn.net/carson_ho/article/details/79620486)

### 扩展
- Space组件  

在ConstraintLayout出来前,我们写布局都会使用到大量的margin或padding,但是这种方式可读性会很差,加一个布局嵌套又会损耗性能  

鉴于这种情况,我们可以使用space,使用方式和View一样,不过主要用来占位置,不会有任何显示效果  
[Space使用方式及情景](https://www.jianshu.com/p/2cd35845b3b3)

- ConstraintLayout布局  

ConstraintLayout是谷歌最新推出的布局组件，它允许你在不嵌套其他视图的情况下创建大型而又复杂的布局，ConstraintLayout 比传统布局的性能更出色。  

在使用传统布局方式进行构建程序时，几乎所有情形下，都需要一个深度嵌套的布局，因此，建议在设计应用布局时使用 ConstraintLayout，ConstraintLayout 应当成是优化性能和易用性的不二之选

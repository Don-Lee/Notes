## 1、Inconvertible types; cannot cast 'java.lang.Double' to 'float ##
如下所示，将Double转为float时会报错。
```java
List<Double> da = new ArrayList<>();
da.add(3.14);
float val = (float)da.get(0);
```
最后发现需要先转化为基本类型，在进行转换，如下所示：
```java
List<Double> da = new ArrayList<>();
da.add(3.14);
float val = (float) da.get(0).doubleValue();
```

## 2、ForegroundLinearLayout 的 Android 命名空间
ForegroundLinearLayout 包括三属性: foregroundInsidePadding, android: 前景, 和 android:foregroundGravity。请注意, foregroundInsidePadding 不包括在 android 名称空间中, 与其他两个属性不同。

在早期版本的 AAPT 中, 当您使用 android 命名空间定义它时, 编译器会默默地忽略 foregroundInsidePadding 属性。在使用 AAPT2 时, 编译器会提前捕获此错误, 并引发下面的生成出错:
```java
Error: (...) resource android:attr/foregroundInsidePadding is private
```
要解决这个问题将android:foregroundInsidePadding替换为foregroundInsidePadding即可

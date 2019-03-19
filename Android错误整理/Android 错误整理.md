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

1、数组转ArrayList
------
为了实现把一个数组转换成一个ArrayList,很多Java程序员会使用如下的代码：
```java
  List<String> list = Arrays.asList(arr);
```
Arrays.asList确实会返回一个ArrayList对象，但是该类是Arrays类中一个私有静态内部类，而不是常见的java.util.ArrayList类。这个
java.util.Arrays.ArrayList类具有set(),get(),contains()等方法，但是不具有任何添加或移除元素的任何方法。因为该类的大小（size）是固定的。
为了创建出一个真正的ArrayList,代码应如下所示：
```java
ArrayList<String> arrayList=new ArrayList<String>(Arrays.asList(arr));
```
我们知道，ArrayList的构造方法可接受一个Collection类型的对象，而我们的java.util.Arrays.ArrayList正好也是它的一个子类。实际上，更高效的代码示例是：<br/>
```Java
ArrayList<String> arrayList=new ArrayList<String>(arr.length);
Collections.addAll(arrayList,arr);
```

2、数组是否包含特定值
-----------
为了检查数组是否包含某个特定值，很多java程序员会使用如下代码：
```java
Set<String> set=new HashSet<String>(Arrays.asList(arr));    
return set.contains(targetValue); 
```
就功能而言，改代码正确无误，但是数组转List,List再转Set的过程消耗了大量的性能.我们可以优化成如下形式： 
```java
Arrays.asList(arr).contains(targetValue);
```
或者进一步优化成如下所示最高效代码：<br/>
```java
for(String s : arr){
  if(s.equals(targetValue))
    return true;
}
```

3、在迭代时移除List中的元素
-------
首先，看一下在迭代过程中移除List中元素的代码：
```java
ArrayList<String> list=new ArrayList<String>(Arrays.asList("a","b","c","d"));
for(int i=0; i<list.size(); i++){
  list.remove(i);
}
System.out.println(list);
```
这个示例代码的输出结果是：<br/>
[b,d]           <br/>
这个示例代码中存在一个非常严重的错误。当一个元素被移除时，该List的大小(size)就会缩减，同事也改变了索引的指向。所以，在迭代的过程中使用索引，
将无法从List中中正确的删除多个指定的元素.<br/>
你可能知道解决这个错误的方式之一是使用迭代器(iterator)。而且，你可能认为Java中的foreach语句与迭代器(iterator)是非常相似的，但实际情况并不是这样。
看一下如下代码示例：
```java
ArrayList<String> list=new ArrayList<String>(Arrays.asList("a","b","c","d"));
for(String s : list){
  if(s.equals("s")){list.remove(s)};
}
```
这个示例代码会抛出一个ConcurrentModificationException。我们应修改成如下代码：
```java
ArrayList<String> list=new ArrayList<String>(Arrays.asList("a","b","c","d"));
Iterator<String> iter=list.iterator();
while(iter.hasNext()){
  String s=iter.next();
  if(s.equals("a")){
    iter.remove();}
}
```
next()方法必须在remove()方法前调用。在foreach循环中，编译器使得remove()方法先于next()方法被调用，这就导致了ConcurrentModificationException 异常。

4、Hashtable vs HashMap
-------------
学习过数据结构的读者都知道一种非常重要的数据结构叫做哈希表。在Java中，对应哈希表的类是HashMap而不是Hashtable.<br/>
HashMap和Hashtable之间最大的核心区别就是:HashMap是非同步的，Hashtable是同步的。

5、ArrayList vs LinkedList
--------------
喝多Java的初学者不明白ArrayList与LinkedList之间的区别，所以他们完全只用相对简单的ArrayList,甚至不知道JDK中还存在LinkedList.但是，
在某些具体场景下，这两种List的选择会导致程序性能的巨大差异。简单而言：当应用场景中有很多的add/remove操作，只有少量的随机访问操作时，
应该选择LinkedList;其他场景下，考虑使用ArrayList.




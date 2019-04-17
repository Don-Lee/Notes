Android Studio编译项目时经常会遇到如下错误：
```java
Process 'command 'C:\Users\Administrator\AppData\Local\Android\Sdk\build-tools\27.0.0\aapt.exe'' finished with non-zero exit value 1
```
解决方案可以归纳为如下几种：

- Android Studio 问题，重启后打开就好
- Clean、Rebuild 缓存问题
- SDK 版本问题
- 项目中各种包之间版本互斥问题
- 资源文件问题  用的图片应该是png而不是jpg
- Build.Gradle 文件里加几句代码...

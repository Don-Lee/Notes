在日常开发中，我们或多或少都会碰到多渠道打包的一些问题，有些是同一个版本要上传到不同的平台，有些是要提供给不同的代理商，
中间可能需要改动里面的图片或其他的一些资源文件，如果每个版本都维护一套代码替换会令人吐血~ 好在可以使用Gradle的productFlavors实现多渠道。
废话不多说，上代码.........

### 资源文件的适配
假如有这么一个需求：现在需要打包两个版本，但要求两个版本的某些图片不一样，并且app的名称也不同，简单的做下适配 
###### 1、这里我新建立了一个res资源文件，res_cq，把这个版本的图片放到这资源文件下，这样项目在打包时就会直接覆盖res文件下对应的资源             
![Image text](https://github.com/Don-Lee/Notes/blob/master/Images/20180613173408.png)            
###### 2、在gradle里配置不同平台采用不同的资源文件
```java
//以下代码放在android{}内
//配置资源文件路径，可动态指定不同版本资源文件
    sourceSets {
        main {
            manifest.srcFile 'src/main/AndroidManifest.xml'
            java.srcDirs = ['src/main/java']
            resources.srcDirs = ['src/main/resources']
            aidl.srcDirs = ['src/main/aidl']
            renderscript.srcDirs = ['src/maom']
            res.srcDirs = ['src/main/res']
            assets.srcDirs = ['src/main/assets']
            jniLibs.srcDir 'src/main/jniLibs'
        }

        //订制版APP对应的资源文件路径,“chunqin”在productFlavors里定义
        chunqin.res.srcDirs = ['src/main/res_cq']
        }
    } 
    
    productFlavors{
        ewd{
            // 对resValue在java代码中的使用：String app_id = getResources().getString(R.string.app_id);
            resValue("string", "app_id", "50074")
            resValue("string", "app_start", "1")
            // 对manifestPlaceholders中资源的使用：在AndroidManifest.xml文件中的application节点下
            // andorid:icon="${icon}"
            // android:app_name="${app_name}"
            manifestPlaceholders=[app_name:"@string/app_name",icon:"@mipmap/ic_launcher"]
        }
        chunqin{
            //包名
            applicationId "com.ewd.cq"
            manifestPlaceholders=[app_name:"@string/app_name_cq",icon:"@mipmap/ic_launcher"]
        }
    }

```

更多可参考https://blog.csdn.net/CHX_W/article/details/78660462

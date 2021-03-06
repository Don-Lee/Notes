1.Matisse内置的图片加载器为Glide和Picasso，默认使用的Glide为图片的加载方式，Matisse目前使用的Glide版本为3.7，
如果项目使用的版本号大于Glide3.8的话则会导致触发如下的异常。为什么会产生异常？因为Glide4.0之后Api的调用方式有了一些更改，所以之前的一些Api调用方式则会出错
```java
 java.lang.NoSuchMethodError: No virtual method load(Landroid/net/Uri;)Lcom/bumptech/glide/DrawableTypeRequest; in class Lcom/bumptech/glide/RequestManager; or its super classes (declaration of 'com.bumptech.glide.RequestManager' appears in /data/app/note.lym.org.noteproject-2/base.apk)
                                                       at com.zhihu.matisse.engine.impl.GlideEngine.loadThumbnail(GlideEngine.java:36)
                                                       at com.zhihu.matisse.internal.ui.widget.PhotoGrid.setImage(PhotoGrid.java:112)
                                                       at com.zhihu.matisse.internal.ui.widget.PhotoGrid.bindPhoto(PhotoGrid.java:80)
                                                       at com.zhihu.matisse.internal.ui.adapter.AlbumPhotosAdapter.onBindViewHolder(AlbumPhotosAdapter.java:111)
                                                       at com.zhihu.matisse.internal.ui.adapter.RecyclerViewCursorAdapter.onBindViewHolder(RecyclerViewCursorAdapter.java:44)
```
解决方案：    
自定义Matisse的iamgeEngine，设置为imageEngine(new GlideEngine())即可。  
GlideEngine代码如下：  
```java
public class GlideEngine implements ImageEngine {

    @Override
    public void loadThumbnail(Context context, int resize, Drawable placeholder, ImageView imageView, Uri uri) {
        RequestOptions options = new RequestOptions()
                .centerCrop()
                .placeholder(placeholder)//这里可自己添加占位图
                .override(resize, resize);
        Glide.with(context)
                .asBitmap()  // some .jpeg files are actually gif
                .load(uri)
                .apply(options)
                .into(imageView);
    }

    @Override
    public void loadGifThumbnail(Context context, int resize, Drawable placeholder, ImageView imageView, Uri uri) {
        RequestOptions options = new RequestOptions()
                .centerCrop()
                .placeholder(placeholder)//这里可自己添加占位图
                .override(resize, resize);
        Glide.with(context)
                .asGif()  // some .jpeg files are actually gif
                .load(uri)
                .apply(options)
                .into(imageView);
    }


    @Override
    public void loadImage(Context context, int resizeX, int resizeY, ImageView imageView, Uri uri) {
        RequestOptions options = new RequestOptions()
                .centerCrop()
                .override(resizeX, resizeY)
                .priority(Priority.HIGH);
        Glide.with(context)
                .load(uri)
                .apply(options)
                .into(imageView);
    }

    @Override
    public void loadGifImage(Context context, int resizeX, int resizeY, ImageView imageView, Uri uri) {
        RequestOptions options = new RequestOptions()
                .centerCrop()
                .override(resizeX, resizeY);
        Glide.with(context)
                .asGif()  // some .jpeg files are actually gif
                .load(uri)
                .apply(options)
                .into(imageView);
    }

    @Override
    public boolean supportAnimatedGif() {
        return true;
    }
}

```

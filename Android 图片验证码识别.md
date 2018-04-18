最近接触了一款理财产品，饥饿销售、定点发放，因此在朋友的怂恿下觉得开发一个外挂，助其抢产品。好了，言归正传，下面进入正题：                           破解过程其实很简单，使用抓包工具抓包获得接口即可，但是有个比较地方比较麻烦那就是验证码的问题，因此我在次记录一下，以备它用：                     
  识别主要过程：               
验证码识别的主要分为两步：图片预处理和图像识别。具体步骤如下：               
1. 图像灰度化、二值化，这一步主要为了后面处理的方便，将图片中每个点的像素值只设为两种，要么为255要么为0，一种表示信息元素，一种表示背景。
但是如果验证码是彩色的，而干扰因素比如噪点和干扰线可以通过颜色过滤，那么在灰度化、二值化之前还是需要先过滤掉干扰信息，具体情况具体分析，
没有一个固定的顺序。                
2. 图像降噪点，去除干扰线。主要通过一些滤波算法、降噪算法将验证码图片的干扰因素去掉。              
3. 接下来就是字符分割，将图片上的字符分为单个的字符                   
4. 训练，将一定规模的标记号的字符经过图片预处理后得到的特征值与其对应的字符内容，选择特定算法进行训练，得到训练模板             

 好了，思路已经有了，接下来就要对图片进行二值化和降噪，这里我用的OpenCV的二值化、腐蚀以及膨胀推图片进行的处理，效果还是不错的，代码如下：
  
```java
 public Bitmap binaryImg(Bitmap bitmap) {

        Mat src = new Mat(bitmap.getHeight(), bitmap.getWidth(), CvType.CV_8UC1);
        Mat grayMat = new Mat();
        Mat destMat = new Mat();
        Mat stMat = new Mat();
        // Bitmap转为Mat
        Utils.bitmapToMat(bitmap, src);

        // 图像置灰
//        Imgproc.cvtColor(src, src, Imgproc.COLOR_RGB2GRAY);
        Imgproc.cvtColor(src, grayMat, Imgproc.COLOR_BGR2GRAY);
        // 自适应阈值化
//        Imgproc.adaptiveThreshold(grayMat, destMat, 255, Imgproc.ADAPTIVE_THRESH_MEAN_C, Imgproc.THRESH_BINARY, 15, 3);

        // 二值阈值化
         Imgproc.threshold(grayMat,destMat,99,255,Imgproc.THRESH_BINARY);
        // 阈值化到零
//         Imgproc.threshold(grayMat,destMat,100,255,Imgproc.THRESH_TOZERO);
        // 截断阈值化
//         Imgproc.threshold(grayMat,destMat,100,255,Imgproc.THRESH_TRUNC);
        // 反转二值阈值化
//         Imgproc.threshold(grayMat,destMat,100,255,Imgproc.THRESH_BINARY_INV);
        // 反转阈值化到零
//         Imgproc.threshold(grayMat,destMat,100,255,Imgproc.THRESH_TOZERO_INV);

        //设置内核形状和内核大小
        Mat kernelDilate = Imgproc.getStructuringElement(Imgproc.MORPH_RECT, new Size(3, 3),new Point(-1, -1));
        Imgproc.erode(destMat,stMat,kernelDilate);//腐蚀
        Imgproc.dilate(stMat,stMat,kernelDilate);//膨胀
        // Mat转Bitmap
        Bitmap processedImage = Bitmap.createBitmap(stMat.cols(), stMat.rows(), Bitmap.Config.ARGB_8888);
//        Bitmap processedImage = Bitmap.createBitmap(stMat.cols(), stMat.rows(), Bitmap.Config.ARGB_8888);
        Utils.matToBitmap(stMat, processedImage);
//        bitmap.recycle();//释放bitmap
        return processedImage;
    }
```
图片处理完后进行识别,代码如下：
```java
    private void ocrImage(Bitmap bitmap) {
        //图片分割，因为我这边的验证码均匀分布，所以我就直接等距分割了
        mItemBitmaps = ImageSplit.split(binaryImg(bitmap), 4); 
        stringBuilder.delete(0, stringBuilder.length());

        for (int i = 0; i < mItemBitmaps.size(); i++) {
            stringBuilder.append(OcrUtil.ScanEnglish(mItemBitmaps.get(i).bitmap));//获取识别后的结果
        }
 }
```
图片识别代码：
```java
 //字体库路径，此路径下必须包含tessdata文件夹，但不用把tessdata写上
    static final String TESSBASE_PATH = Environment.getExternalStorageDirectory() + File.separator;
    //英文字库名称
    static final String ENGLISH_LANGUAGE = "eng";
public static String ScanEnglish(final Bitmap bmp) {
        String result = "";
        TessBaseAPI baseApi = new TessBaseAPI();

        //初始化OCR的字体数据，TESSBASE_PATH为路径，ENGLISH_LANGUAGE指明要用的字体库（不用加后缀）
        if (baseApi.init(TESSBASE_PATH, ENGLISH_LANGUAGE)) {
            baseApi.setDebug(true);
            baseApi.setVariable(TessBaseAPI.VAR_CHAR_WHITELIST, "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789"); // 识别白名单
            baseApi.setVariable(TessBaseAPI.VAR_CHAR_BLACKLIST, "!@#$%^&*()_+=-[]}{;:'\"\\|~`,./<>?"); // 识别黑名单

            //设置识别模式
            baseApi.setPageSegMode(TessBaseAPI.PageSegMode.PSM_SINGLE_CHAR);
            //设置要识别的图片
            baseApi.setImage(bmp);
            //开始识别
            result = baseApi.getUTF8Text();
//            String inspection = baseApi.getHOCRText(0);
//            final String outputText = Html.fromHtml(inspection).toString().trim();

            baseApi.clear();
            baseApi.end();
//            Log.e("Ocr", result + "---" + Thread.currentThread().getName());
        } else {
            Log.e("Ocr", "----init失败---");
        }
        return result;

    }
```

好了，到此验证码识别就完成了。在此补充一下 android studio 配置openCV的方法：                  
1、在http://opencv.org/  官网上下载OpenCV-android-sdk     
2、解压下载的文件：              
sdk目录即是我们开发opencv所需要的类库；                 
samples目录中存放着若干opencv应用示例；                      
apk目录则存放着对应于各内核版本的OpenCV_x_Manager_x应用安装包。此应用用来管理手机设备中的opencv类库，在运行opencv应用之前，
必须确保手机中已经安装了OpenCV_3.2.0_Manager_3.2_*.apk，否则opencv应用将会因为无法加载opencv类库而无法运行；如果实在不想安装
管理opencv类库的安装包，则需要在app启动时添加如下静态代码块就可：
```java
static{
        System.loadLibrary("opencv_java3");
    }
```
3、将OpenCV加入到AndroidStudio中：                                                 
在Android Studio中选择File->New->Import Module，找到OpenCV解压的路径，选择sdk/java文件夹。                                                   
4、添加Module Dependency：              
  使用快捷键Shift+Ctrl+Alt+s  或者直接右击app->Open Module Settings 打开下面对话框选择在app module                
的Dependencies一栏将openCVLibrary添加进去                            
5、在app-->src-->main目录下，新建文件夹jniLibs  并将OpenCV-android-sdk 解压包下面的sdk->native->libs文件夹下面的文件拷贝到jniLibs文件下。
（我只拷贝了armeabi-v7a文件夹）

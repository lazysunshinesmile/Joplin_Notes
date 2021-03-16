Bitmap的高效使用--BitmapFactory.options

Bitmap的高效使用--BitmapFactory.options

# Bitmap基本概念：  
1. bitmap是Android系统中的图像处理的重要类之一；  
2. 通过bitmap我们可以获取到图片的相关信息；  
3. bitmap文件图像效果好就需要占用越大存储空间；  
# Bitmap的加载方式：

```
1.BitmapFactory.decodeBetyArray();
2.BitmapFactory.decodeFile();
3.BitmapFactory.decodeResource();
4.BitmapFactory.decodeStream();

```

# Bitmap不做处理的加载方式：
```
Bitmap bitmap = BitmapFactory.decodeResource(getResources,R.drawable.ic_launcher,null);
```

# BitmapFactory.Options的作用：  
1.防止内存溢出；  
2.节省内存开销；  
3.系统更流畅；

# BitmapFactory.Options的重要属性:  
1. injustDecodeBounds;  
	* 设为true，那么BitmapFactory并不会真的返回一个Bitmap给你，它仅仅会把它的宽，高取回来给你，这样就不会占用太多的内存，也就不会那么频繁的发生OOM。  
	* 设置为false，BitmapFactory返回bitmap;  

2. outWidth&outHeight;  
bitmap图像的宽和高；  

3. inSampleSize;  
获取采样率  
	* inSampleSize大于1时，图像高、宽分别以inSampleSize分之一缩小，inSampleSize是2的n次方，如果不是，将被四舍五入到最接近的2的幂次。比如，输入7，相当于8
	* inSampleSize小于等于1时，图像高、宽不变；  

4. inpreferredConfig：  
	* ALPHA_8: 每个像素用占8位,存储的是图像的透明值,占1个字节;  
	* RGB_565:每个像素用占16位,分别为5-R,6-G,5-B通道,占2个字节;  
	* ARGB-4444:每个像素占16位,即每个通道用4位表示,占2个字节;  
	* ARGB_8888:每个像素占32位,每个通道用8位表示,占4个字节;  

5. inDither:  
是否进行图像抖动处理；  

6. inMutable：  
如果设置为true,将返回一个mutable的bitmap,可用于修改BitmapFactory加载而来的bitmap.  

7. inScale:  
是否需要放缩位图

# BitmapFactory.options的实际应用：
```
public static Bitmap readBitmapFromFile(String filePath, int width, int height) {
        BitmapFactory.Options options = new BitmapFactory.Options();
        options.inJustDecodeBounds = true;
        BitmapFactory.decodeFile(filePath, options);
        float srcWidth = options.outWidth;
        float srcHeight = options.outHeight;
        int inSampleSize = 1;

        if (srcWidth > height && srcWidth > width) {
             inSampleSize  = srcWidth /width;
        } else if(srcWidth <height  && srcHeight >height  ){
           inSampleSize  = srcHeight /height ;
        }

        if(inSampleSize  <=0){
            inSampleSize  =1;
        }
        options.inJustDecodeBounds = false;
        options.inSampleSize = inSampleSize;

        return BitmapFactory.decodeFile(filePath, options);
    }

```

# Bitmap资源回收：

```
if (bitmap!=null&&!bitmap.isRecycle()) {
    bitmap.recycle();  
    bitmap= null;
    system.gc(); 
}

```
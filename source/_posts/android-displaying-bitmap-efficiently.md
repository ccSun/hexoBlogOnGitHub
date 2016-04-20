title: Android-Displaying-Bitmap-Efficiently
categories:
	- Android
tags:
	- Optimization
date: 2016-03-31 18:31:43
---
在手机上的内存有限(一般每个app至少16M)，而图片资源占用空间极大 。比如Galaxy Nexus的camera占用2592x1936 pixels。如果使用ARGB_888，将占用19MB空间（2592x1936x4bytes）。尤其在listview，gridview中，占用的空间更大，更易发生

```
java.lang.OutofMemoryError: bitmap size exceeds VM budget.
```
## 一、 Loading Large Bitmaps Efficiently

在内存加载分辨率合适的图片，而不是清晰度最高的图片。

### 1. 尺寸压缩

```
public static int calculateInSampleSize(BitmapFactory.Options options, int reqWidth, int reqHeight){
	
	final int halfHeight = options.outHeight / 2;
	final int halfWidth = options.outWidth / 2;
	
	int inSampleSize = 1;
	
		
	while( (halfHeight/inSampleSize) > reqHeight  
		&& (halfWidth/inSampleSize) > reqWidth ){
		
			inSampleSize *= 2;
	}

	return inSampleSize;

}

//public static Bitmap decodeSampleBitmapFrom**** 可有多个实现
// decodeByteArray decodeFile decodResource
// 其实这个方法最好不要在ui主线程执行
public static Bitmap decodeSampleBitmapFromResource(Resources res, int resId, int reqWidth, int reqHeight){

	final BitmapFactory.Options options = new BitmapFactory.Options();
	
	options.inJustDecodeBounds = true;
	BitmapFactory.decodeResource(res, resId, options);
	
	options.inSampleSize = calculateInSampleSize(options, reqWidth, reqHeight);
	
	options.inJustDecodeBounds = false;
	
	return BitmapFactory.decodeResource(res, resId, options);

}



imgView.setImageBitmap(decodeSampleBitmapFromResource(getResources(), R.id.myimage, 100,100));

```


## 二、 Processing Bitmaps Off the UI Thread

```
// 把decode放到AsyncTask里执行
    class BitmapWorkerTask extends AsyncTask<Integer, Void, Bitmap> {


        private final WeakReference<ImageView> weakReference;

        public BitmapWorkerTask(ImageView imgView) {
            weakReference = new WeakReference<ImageView>(imgView);
        }

        @Override
        protected Bitmap doInBackground(Integer... params) {
            return decodeSampleBitmapFromResource(getResources(), params[0], 100, 100);
        }

        @Override
        protected void onPostExecute(Bitmap bitmap){
            if(null != weakReference && null != bitmap){


                final ImageView imageView = weakReference.get();
                if(null != imageView){
                    imageView.setImageBitmap(bitmap);
                }
            }
        }
    }

// 注意 如果是刷新，那么AsyncTask会启动多次，注意处理不要重复启动的问题


```
Google案例：
[Handling Concurrency](http://developer.android.com/training/displaying-bitmaps/process-bitmap.html#concurrency)    
实现原理：

1. Async中软引用imageView，保存被加载的图片文件的id；
2. 自实现一个BitmapDrawable,软引用一个Async；
3. 加载图片的时候，new一个async，new一个子类BitmapDrawable，执行async；在这之前先判断为这个imgView加载图片的的async是否正在执行：通过imgView.getDrawable得到drawable，drawble instance of 子类BitMapDrawable，进一步得到对应的Async，拿到Async中正在加载的图片文件id，判断与正准备启动加载的图片是否相同，如果相同，则原来的async继续执行；如果不同，取消原来的async，新建新的async。


深入探索：
[Multithreading for Performance](http://android-developers.blogspot.com/2010/07/multithreading-for-performance.html)


## 三、 Caching Bitmaps

用内存和硬盘缓存来完成listview等滑动造成的view加载问题。

### 1. Use a Memory Cache

以前是是用软引用和弱引用实现内存缓存，但是Android 2.3之后GC会更主动的去回收软引用和弱引用。3.0之后，bitmap存到本地内存中，使用不当会导致内存溢出。On a normal/hdpi device this is a minimum of around 4MB (32/8). A full screen GridView filled with images on a device with 800x480 resolution would use around 1.5MB (800*480*4 bytes), so this would cache a minimum of around 2.5 pages of images in memory.

现在是使用强引用***LinkedHashMap***实现的LruCache来实现。

```
private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    // Get max available VM memory, exceeding this amount will throw an
    // OutOfMemory exception. Stored in kilobytes as LruCache takes an
    // int in its constructor.
    final int maxMemory = (int) (Runtime.getRuntime().maxMemory() / 1024);

    // Use 1/8th of the available memory for this memory cache.
    final int cacheSize = maxMemory / 8;

    mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
        @Override
        protected int sizeOf(String key, Bitmap bitmap) {
            // The cache size will be measured in kilobytes rather than
            // number of items.
            return bitmap.getByteCount() / 1024;
        }
    };
    ...
}

public void addBitmapToMemoryCache(String key, Bitmap bitmap) {
    if (getBitmapFromMemCache(key) == null) {
        mMemoryCache.put(key, bitmap);
    }
}

public Bitmap getBitmapFromMemCache(String key) {
    return mMemoryCache.get(key);
}

```

### 2. Use a Disk Cache


Fetching images from disk is slower than loading from memory and ***should be done in a background thread***, as disk read times can be unpredictable.

***Note:*** A ContentProvider might be a more appropriate place to store cached images if they are accessed more frequently, for example in an image gallery application.


//[代码参考](http://developer.android.com/training/displaying-bitmaps/cache-bitmap.html#disk-cache)
简单分析：
ImageView都会绑定一个Drawable。但是我们需要从这个imageView直接找到当前是否有Async为这个ImageView加载图片。所以自实现一个AsyncDrawable，保存Async的弱引用WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference。

为一个ImageView加载图片的时候，先看当前是否有正在加载相同图片的Async运行。如果没有，则新建Async；在Async onPost加载完图片后，设定到imageView之前，判断这个imageView的drawable的Async是否是当前Async，如果是imageView.setImageBitmap(bitmap);如果不是，那就意味着这个ImageView加载的图片换了。当前的Async加载的图片无用了。

setImageBitmap本质是封装了的setImageDrawable.


### 3. 横竖屏切换

fragment里setRetainInstance(true)保持fragment不被销毁。复用更有效率。

```
private LruCache<String, Bitmap> mMemoryCache;

@Override
protected void onCreate(Bundle savedInstanceState) {
    ...
    RetainFragment retainFragment =
            RetainFragment.findOrCreateRetainFragment(getFragmentManager());
    mMemoryCache = retainFragment.mRetainedCache;
    if (mMemoryCache == null) {
        mMemoryCache = new LruCache<String, Bitmap>(cacheSize) {
            ... // Initialize cache here as usual
        }
        retainFragment.mRetainedCache = mMemoryCache;
    }
    ...
}

class RetainFragment extends Fragment {
    private static final String TAG = "RetainFragment";
    public LruCache<String, Bitmap> mRetainedCache;

    public RetainFragment() {}

    public static RetainFragment findOrCreateRetainFragment(FragmentManager fm) {
        RetainFragment fragment = (RetainFragment) fm.findFragmentByTag(TAG);
        if (fragment == null) {
            fragment = new RetainFragment();
            fm.beginTransaction().add(fragment, TAG).commit();
        }
        return fragment;
    }

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setRetainInstance(true);
    }
}
```

## 四、 Managing Bitmap Memory

### 1. Manage Memory on Android 2.3.3 and Lower
需要用recycle()，程序员确保图片不被使用. On Android 2.3.3 (API level 10) and lower, the backing pixel data for a bitmap is stored in native memory. It is separate from the bitmap itself, which is stored in the Dalvik heap. 

### 2. Manage Memory on Android 3.0 and Higher
 As of Android 3.0 (API level 11), the pixel data is stored on the Dalvik heap along with the associated bitmap.
 
BitmapFactory.Options.inBitmap  If this option is set, decode methods that take the Options object will attempt to reuse an existing bitmap when loading content. 

4.4之前，复用的内存大小需要与之前图片相同。 


[参考案例](http://developer.android.com/training/displaying-bitmaps/manage-memory.html)

简单分析：
LruCache存储满被挤出来的bitmap用Set<SoftReference<Bitmap>>来保存。
decodeBitmapFrom***()的时候，就尝试把解析的图片存储到inBitmap中。在存储之前，要先判断当前图片是否符合添加到inBitmap的条件。

```
Set<SoftReference<Bitmap>> mReusableBitmaps;
private LruCache<String, BitmapDrawable> mMemoryCache;

// If you're running on Honeycomb or newer, create a
// synchronized HashSet of references to reusable bitmaps.
if (Utils.hasHoneycomb()) {
    mReusableBitmaps =
            Collections.synchronizedSet(new HashSet<SoftReference<Bitmap>>());
}



public static Bitmap decodeSampledBitmapFromFile(String filename,
        int reqWidth, int reqHeight, ImageCache cache) {

    final BitmapFactory.Options options = new BitmapFactory.Options();
    ...
    BitmapFactory.decodeFile(filename, options);
    ...

    // If we're running on Honeycomb or newer, try to use inBitmap.
    if (Utils.hasHoneycomb()) {
    	//解析之后，就尝试把图片加到inBitmap中
        addInBitmapOptions(options, cache);
    }
    ...
    return BitmapFactory.decodeFile(filename, options);
}

private static void addInBitmapOptions(BitmapFactory.Options options,
        ImageCache cache) {
    // inBitmap only works with mutable bitmaps, so force the decoder to
    // return mutable bitmaps.
    options.inMutable = true;

    if (cache != null) {
        // Try to find a bitmap from mMemoryCache to use for inBitmap.
        Bitmap inBitmap = cache.getBitmapFromReusableSet(options);

        if (inBitmap != null) {
            // If a suitable bitmap has been found, set it as the value of inBitmap.
            // 只有在软引用set里的bitmap才添加到inBitmap
            options.inBitmap = inBitmap;
        }
    }
}

protected Bitmap getBitmapFromReusableSet(BitmapFactory.Options options) {
	// 遍历mMemoryCache，判断每一个bitmap是否符合添加到inBitmap的条件
	// 如果都不符合，那就不添加到inBitmap

}


// 判断是否符合
static boolean canUseForInBitmap(
        Bitmap candidate, BitmapFactory.Options targetOptions) {

    if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
        // From Android 4.4 (KitKat) onward we can re-use if the byte size of
        // the new bitmap is smaller than the reusable bitmap candidate
        // allocation byte count.
        int width = targetOptions.outWidth / targetOptions.inSampleSize;
        int height = targetOptions.outHeight / targetOptions.inSampleSize;
        int byteCount = width * height * getBytesPerPixel(candidate.getConfig());
        return byteCount <= candidate.getAllocationByteCount();
    }

    // On earlier versions, the dimensions must match exactly and the inSampleSize must be 1
    return candidate.getWidth() == targetOptions.outWidth
            && candidate.getHeight() == targetOptions.outHeight
            && targetOptions.inSampleSize == 1;
}

/**
 * A helper function to return the byte usage per pixel of a bitmap based on its configuration.
 */
static int getBytesPerPixel(Config config) {
    if (config == Config.ARGB_8888) {
        return 4;
    } else if (config == Config.RGB_565) {
        return 2;
    } else if (config == Config.ARGB_4444) {
        return 2;
    } else if (config == Config.ALPHA_8) {
        return 1;
    }
    return 1;
}

```
title: Android-Displaying-Bitmap-Efficiently
categories:
	- Android
tags:
	- Optimization
date: 2016-03-31 18:31:43
---
在手机上的内存有限，而图片资源占用空间极大 。比如Galaxy Nexus的camera占用2592x1936 pixels。如果使用ARGB_888，将占用19MB空间（2592x1936x4bytes）。尤其在listview，gridview中。还没看完～～

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




## 五、 Displaying Bitmaps in Your UI
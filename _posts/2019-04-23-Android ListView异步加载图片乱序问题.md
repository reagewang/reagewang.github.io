---
layout: post
title: "Android ListView异步加载图片乱序问题，原因分析及解决方案"
tags: [Android, ListView, LruCache]
comments: true
---

在Android所有系统自带的控件当中，ListView这个控件算是用法比较复杂的了，关键是用法复杂也就算了，它还经常会出现一些稀奇古怪的问题，让人非常头疼。比如说在ListView中加载图片，如果是同步加载图片倒还好，但是一旦使用异步加载图片那么问题就来了，这个问题我相信很多Android开发者都曾经遇到过，就是异步加载图片会出现错位乱序的情况。遇到这个问题时，不少人在网上搜索找到了相应的解决方案，但是真正深入理解这个问题出现的原因并对症解决的人恐怕还并不是很多。那么今天我们就来具体深入分析一下ListView异步加载图片出现乱序问题的原因，以及怎么样对症下药去解决它。

# 问题重现
---
要想解决问题首先我们要把问题重现出来，这里只需要搭建一个最基本的ListView项目，然后在ListView中去异步请求图片并显示，问题就能够得以重现了，那么我们就新建一个ListViewTest项目。

项目建好之后第一个要解决的是数据源的问题，由于ListView中需要从网络上请求图片，那么我就提前准备好了许多张图片，将它们上传到了我的CSDN相册当中，然后新建一个Images类，将所有相册中图片的URL地址都配置进去就可以了，代码如下所示：
```java
/**
 * 原文地址: http://blog.csdn.net/guolin_blog/article/details/45586553
 * @author guolin
 */
public class Images {
 
    public final static String[] imageUrls = new String[] {
    	"https://img-my.csdn.net/uploads/201508/05/1438760758_3497.jpg",  
        "https://img-my.csdn.net/uploads/201508/05/1438760758_6667.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760757_3588.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760756_3304.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760755_6715.jpeg",
        "https://img-my.csdn.net/uploads/201508/05/1438760726_5120.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760726_8364.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760725_4031.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760724_9463.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760724_2371.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760707_4653.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760706_6864.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760706_9279.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760704_2341.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760704_5707.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760685_5091.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760685_4444.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760684_8827.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760683_3691.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760683_7315.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760663_7318.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760662_3454.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760662_5113.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760661_3305.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760661_7416.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760589_2946.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760589_1100.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760588_8297.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760587_2575.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760587_8906.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760550_2875.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760550_9517.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760549_7093.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760549_1352.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760548_2780.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760531_1776.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760531_1380.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760530_4944.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760530_5750.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760529_3289.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760500_7871.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760500_6063.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760499_6304.jpeg",
        "https://img-my.csdn.net/uploads/201508/05/1438760499_5081.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760498_7007.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760478_3128.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760478_6766.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760477_1358.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760477_3540.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760476_1240.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760446_7993.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760446_3641.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760445_3283.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760444_8623.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760444_6822.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760422_2224.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760421_2824.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760420_2660.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760420_7188.jpg",
        "https://img-my.csdn.net/uploads/201508/05/1438760419_4123.jpg",
    };
}
```
设置好了图片源之后，我们需要一个ListView来展示所有的图片。打开或修改activity_main.xml中的代码，如下所示：
```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" 
    android:orientation="vertical">
 
    <ListView
        android:id="@+id/list_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        >
    </ListView>
 
</LinearLayout>
```
很简单，只是在LinearLayout中写了一个ListView而已。接着我们要定义ListView中每一个子View的布局，新建一个image_item.xml布局，加入如下代码：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
 
    <ImageView
        android:id="@+id/image"
        android:layout_width="match_parent"
        android:layout_height="120dp"
        android:src="@drawable/empty_photo" 
        android:scaleType="fitXY"/>
 
</LinearLayout>
```
仍然很简单，image_item.xml布局中只有一个ImageView控件，就是用它来显示图片的，控件在默认情况下会显示一张empty_photo。这样我们就把所有的布局文件都写好了。

接下来新建ImageAdapter做为ListView的适配器，代码如下所示：
```java
/** 
 * 原文地址: http://blog.csdn.net/guolin_blog/article/details/45586553 
 * @author guolin 
 */  
public class ImageAdapter extends ArrayAdapter<String> {
 
	/**
	 * 图片缓存技术的核心类，用于缓存所有下载好的图片，在程序内存达到设定值时会将最少最近使用的图片移除掉。
	 */
	private LruCache<String, BitmapDrawable> mMemoryCache;
 
	public ImageAdapter(Context context, int resource, String[] objects) {
		super(context, resource, objects);
		// 获取应用程序最大可用内存
		int maxMemory = (int) Runtime.getRuntime().maxMemory();
		int cacheSize = maxMemory / 8;
		mMemoryCache = new LruCache<String, BitmapDrawable>(cacheSize) {
			@Override
			protected int sizeOf(String key, BitmapDrawable drawable) {
				return drawable.getBitmap().getByteCount();
			}
		};
	}
 
	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		String url = getItem(position);
		View view;
		if (convertView == null) {
			view = LayoutInflater.from(getContext()).inflate(R.layout.image_item, null);
		} else {
			view = convertView;
		}
		ImageView image = (ImageView) view.findViewById(R.id.image);
		BitmapDrawable drawable = getBitmapFromMemoryCache(url);
		if (drawable != null) {
			image.setImageDrawable(drawable);
		} else {
			BitmapWorkerTask task = new BitmapWorkerTask(image);
			task.execute(url);
		}
		return view;
	}
 
	/**
	 * 将一张图片存储到LruCache中。
	 * 
	 * @param key
	 *            LruCache的键，这里传入图片的URL地址。
	 * @param drawable
	 *            LruCache的值，这里传入从网络上下载的BitmapDrawable对象。
	 */
	public void addBitmapToMemoryCache(String key, BitmapDrawable drawable) {
		if (getBitmapFromMemoryCache(key) == null) {
			mMemoryCache.put(key, drawable);
		}
	}
 
	/**
	 * 从LruCache中获取一张图片，如果不存在就返回null。
	 * 
	 * @param key
	 *            LruCache的键，这里传入图片的URL地址。
	 * @return 对应传入键的BitmapDrawable对象，或者null。
	 */
	public BitmapDrawable getBitmapFromMemoryCache(String key) {
		return mMemoryCache.get(key);
	}
 
	/**
	 * 异步下载图片的任务。
	 * 
	 * @author guolin
	 */
	class BitmapWorkerTask extends AsyncTask<String, Void, BitmapDrawable> {
 
		private ImageView mImageView;
 
		public BitmapWorkerTask(ImageView imageView) {
			mImageView = imageView;
		}
 
		@Override
		protected BitmapDrawable doInBackground(String... params) {
			String imageUrl = params[0];
			// 在后台开始下载图片
			Bitmap bitmap = downloadBitmap(imageUrl);
			BitmapDrawable drawable = new BitmapDrawable(getContext().getResources(), bitmap);
			addBitmapToMemoryCache(imageUrl, drawable);
			return drawable;
		}
 
		@Override
		protected void onPostExecute(BitmapDrawable drawable) {
			if (mImageView != null && drawable != null) {
				mImageView.setImageDrawable(drawable);
			}
		}
 
		/**
		 * 建立HTTP请求，并获取Bitmap对象。
		 * 
		 * @param imageUrl
		 *            图片的URL地址
		 * @return 解析后的Bitmap对象
		 */
		private Bitmap downloadBitmap(String imageUrl) {
			Bitmap bitmap = null;
			HttpURLConnection con = null;
			try {
				URL url = new URL(imageUrl);
				con = (HttpURLConnection) url.openConnection();
				con.setConnectTimeout(5 * 1000);
				con.setReadTimeout(10 * 1000);
				bitmap = BitmapFactory.decodeStream(con.getInputStream());
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				if (con != null) {
					con.disconnect();
				}
			}
			return bitmap;
		}
	}
}
```
ImageAdapter中的代码还算是比较简单的，在getView()方法中首先根据当前的位置获取到图片的URL地址，然后使用inflate()方法加载image_item.xml这个布局，并获取到ImageView控件的实例，接下来开启了一个BitmapWorkerTask异步任务来从网络上加载图片，最终将加载好的图片设置到ImageView上面。注意这里为了防止图片占用过多的内存，我们还是使用了LruCache技术来进行内存控制。

最后，程序主界面的代码就非常简单了，修改MainActivity中的代码，如下所示：
```java
/**  
 * 原文地址: http://blog.csdn.net/guolin_blog/article/details/45586553  
 * @author guolin  
 */    
public class MainActivity extends Activity {
	
	private ListView listView;
	
	@Override
	protected void onCreate(Bundle savedInstanceState) {
		super.onCreate(savedInstanceState);
		setContentView(R.layout.activity_main);
		listView = (ListView) findViewById(R.id.list_view);
		ImageAdapter adapter = new ImageAdapter(this, 0, Images.imageThumbUrls);
		listView.setAdapter(adapter);
	}
}
```
这就是整个程序所有的代码了，记得还需要在AndroidManifest.xml中添加INTERNET权限。

那么目前程序的思路其实是很简单的，我们在ListView的getView()方法中开启异步请求，从网络上获取图片，当图片获取成功就后就将图片显示到ImageView上面。看起来没什么问题对吗？那么现在我们就来运行一下程序看一看效果吧。

![img](/images/25/1.gif)

恩？怎么会这个样子，当滑动ListView的时候，图片竟然会自动变来变去，而且图片显示的位置也不正确，简直快乱成一锅粥了！可是我们所有的逻辑都很简单呀，怎么会导致出现这种图片自动变来变去的情况？很遗憾，这是由于Listview内部的工作机制所导致的，如果你对Listview的工作机制不了解，那么就会很难理解这种现象，不过好在上篇文章中我已经讲解过ListView的工作原理了，因此下面就让我们一起分析一下这个问题出现的原因。

# 原因分析
---
上篇文章中已经提到了，ListView之所以能够实现加载成百上千条数据都不会OOM，最主要在于它内部优秀的实现机制。虽然作为普通的使用者，我们大可不必关心ListView内部到底是怎么实现的，但是当你了解了它的内部原理之后，很多之前难以解释的问题都变得有理有据了。

ListView在借助RecycleBin机制的帮助下，实现了一个生产者和消费者的模式，不管有任意多条数据需要显示，ListView中的子View其实来来回回就那么几个，移出屏幕的子View会很快被移入屏幕的数据重新利用起来，原理示意图如下所示：

![img](/images/25/2.png)

那么这里我们就可以思考一下了，目前数据源当中大概有60个图片的URL地址，而根据ListView的工作原理，显然不可能为每张图片都单独分配一个ImageView控件，ImageView控件的个数其实就比一屏能显示的图片数量稍微多一点而已，移出屏幕的ImageView控件会进入到RecycleBin当中，而新进入屏幕的元素则会从RecycleBin中获取ImageView控件。

那么，每当有新的元素进入界面时就会回调getView()方法，而在getView()方法中会开启异步请求从网络上获取图片，注意网络操作都是比较耗时的，也就是说当我们快速滑动ListView的时候就很有可能出现这样一种情况，某一个位置上的元素进入屏幕后开始从网络上请求图片，但是还没等图片下载完成，它就又被移出了屏幕。这种情况下会产生什么样的现象呢？根据ListView的工作原理，被移出屏幕的控件将会很快被新进入屏幕的元素重新利用起来，而如果在这个时候刚好前面发起的图片请求有了响应，就会将刚才位置上的图片显示到当前位置上，因为虽然它们位置不同，但都是共用的同一个ImageView实例，这样就出现了图片乱序的情况。

但是还没完，新进入屏幕的元素它也会发起一条网络请求来获取当前位置的图片，等到图片下载完的时候会设置到同样的ImageView上面，因此就会出现先显示一张图片，然后又变成了另外一张图片的情况，那么刚才我们看到的图片会自动变来变去的情况也就得到了解释。

问题原因已经分析出来了，但是这个问题该怎么解决呢？说实话，ListView异步加载图片的问题并没有什么标准的解决方案，很多人都有自己的一套解决思路，这里我准备给大家讲解三种比较经典的解决办法，大家通过任何一种都可以解决这个问题，但是我们每多学习一种思路，水平就能够更进一步的提高。

# 解决方案一  使用findViewWithTag
---
findViewWithTag算是一种比较简单易懂的解决方案。那么这里我们先来看看怎么通过修改代码把这个问题解决掉，然后再研究一下findViewWithTag的工作原理。

使用findViewWithTag并不需要修改太多的代码，只需要改动ImageAdapter这一个类就可以了，如下所示：
```java
/** 
 * 原文地址: http://blog.csdn.net/guolin_blog/article/details/45586553 
 * @author guolin 
 */  
public class ImageAdapter extends ArrayAdapter<String> {
	
	private ListView mListView; 
 
	......
 
	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		if (mListView == null) {  
            mListView = (ListView) parent;  
        } 
		String url = getItem(position);
		View view;
		if (convertView == null) {
			view = LayoutInflater.from(getContext()).inflate(R.layout.image_item, null);
		} else {
			view = convertView;
		}
		ImageView image = (ImageView) view.findViewById(R.id.image);
		image.setImageResource(R.drawable.empty_photo);
        image.setTag(url);
		BitmapDrawable drawable = getBitmapFromMemoryCache(url);
		if (drawable != null) {
			image.setImageDrawable(drawable);
		} else {
			BitmapWorkerTask task = new BitmapWorkerTask();
			task.execute(url);
		}
		return view;
	}
 
	......
 
	/**
	 * 异步下载图片的任务。
	 * 
	 * @author guolin
	 */
	class BitmapWorkerTask extends AsyncTask<String, Void, BitmapDrawable> {
 
		String imageUrl; 
 
		@Override
		protected BitmapDrawable doInBackground(String... params) {
			imageUrl = params[0];
			// 在后台开始下载图片
			Bitmap bitmap = downloadBitmap(imageUrl);
			BitmapDrawable drawable = new BitmapDrawable(getContext().getResources(), bitmap);
			addBitmapToMemoryCache(imageUrl, drawable);
			return drawable;
		}
 
		@Override
		protected void onPostExecute(BitmapDrawable drawable) {
			ImageView imageView = (ImageView) mListView.findViewWithTag(imageUrl);  
            if (imageView != null && drawable != null) {  
                imageView.setImageDrawable(drawable);  
            } 
		}
 
		......
	}
}
```
改动的地方就只有这么多，那么我们来分析一下。由于使用findViewWithTag必须要有ListView的实例才行，那么我们在Adapter中怎样才能拿到ListView的实例呢？其实如果你仔细通读了上一篇文章就能知道，getView()方法中传入的第三个参数其实就是ListView的实例，那么这里我们定义一个全局变量mListView，然后在getView()方法中判断它是否为空，如果为空就把parent这个参数赋值给它。

另外在getView()方法中我们还做了一个操作，就是调用了ImageView的setTag()方法，并把当前位置图片的URL地址作为参数传了进去，这个是为后续的findViewWithTag()方法做准备。

最后，我们修改了BitmapWorkerTask的构造函数，这里不再通过构造函数把ImageView的实例传进去了，而是在onPostExecute()方法当中通过ListView的findVIewWithTag()方法来去获取ImageView控件的实例。获取到控件实例后判断下是否为空，如果不为空就让图片显示到控件上。

这里我们可以尝试分析一下findViewWithTag的工作原理，其实顾名思义，这个方法就是通过Tag的名字来获取具备该Tag名的控件，我们先要调用控件的setTag()方法来给控件设置一个Tag，然后再调用ListView的findViewWithTag()方法使用相同的Tag名来找回控件。

那么为什么用了findViewWithTag()方法之后，图片就不会再出现乱序情况了呢？其实原因很简单，由于ListView中的ImageView控件都是重用的，移出屏幕的控件很快会被进入屏幕的图片重新利用起来，那么getView()方法就会再次得到执行，而在getView()方法中会为这个ImageView控件设置新的Tag，这样老的Tag就会被覆盖掉，于是这时再调用findVIewWithTag()方法并传入老的Tag，就只能得到null了，而我们判断只有ImageView不等于null的时候才会设置图片，这样图片乱序的问题也就不存在了。

# 解决方案二  使用弱引用关联
---
虽然这里我给这种解决方案起名叫弱引用关联，但实际上弱引用只是辅助手段而已，最主要的还是关联，这种解决方案的本质是要让ImageView和BitmapWorkerTask之间建立一个双向关联，互相持有对方的引用，再通过适当的逻辑判断来解决图片乱序问题，然后为了防止出现内存泄漏的情况，双向关联要使用弱引用的方式建立。相比于第一种解决方案，第二种解决方案要明显复杂不少，但在性能和效率方面都会有更好的表现。

我们仍然只需要改动ImageAdapter中的代码，但这次改动的地方比较多，所以我就把ImageAdapter中的全部代码都贴出来了，如下所示：
```java

/** 
 * 原文地址: http://blog.csdn.net/guolin_blog/article/details/45586553 
 * @author guolin 
 */  
public class ImageAdapter extends ArrayAdapter<String> {
	
	private ListView mListView; 
	
	private Bitmap mLoadingBitmap;
 
	/**
	 * 图片缓存技术的核心类，用于缓存所有下载好的图片，在程序内存达到设定值时会将最少最近使用的图片移除掉。
	 */
	private LruCache<String, BitmapDrawable> mMemoryCache;
 
	public ImageAdapter(Context context, int resource, String[] objects) {
		super(context, resource, objects);
		mLoadingBitmap = BitmapFactory.decodeResource(context.getResources(),
				R.drawable.empty_photo);
		// 获取应用程序最大可用内存
		int maxMemory = (int) Runtime.getRuntime().maxMemory();
		int cacheSize = maxMemory / 8;
		mMemoryCache = new LruCache<String, BitmapDrawable>(cacheSize) {
			@Override
			protected int sizeOf(String key, BitmapDrawable drawable) {
				return drawable.getBitmap().getByteCount();
			}
		};
	}
 
	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		if (mListView == null) {  
            mListView = (ListView) parent;  
        } 
		String url = getItem(position);
		View view;
		if (convertView == null) {
			view = LayoutInflater.from(getContext()).inflate(R.layout.image_item, null);
		} else {
			view = convertView;
		}
		ImageView image = (ImageView) view.findViewById(R.id.image);
		BitmapDrawable drawable = getBitmapFromMemoryCache(url);
		if (drawable != null) {
			image.setImageDrawable(drawable);
		} else if (cancelPotentialWork(url, image)) {
			BitmapWorkerTask task = new BitmapWorkerTask(image);
			AsyncDrawable asyncDrawable = new AsyncDrawable(getContext()
					.getResources(), mLoadingBitmap, task);
			image.setImageDrawable(asyncDrawable);
			task.execute(url);
		}
		return view;
	}
	
	/**
	 * 自定义的一个Drawable，让这个Drawable持有BitmapWorkerTask的弱引用。
	 */
	class AsyncDrawable extends BitmapDrawable {
 
		private WeakReference<BitmapWorkerTask> bitmapWorkerTaskReference;
 
		public AsyncDrawable(Resources res, Bitmap bitmap,
				BitmapWorkerTask bitmapWorkerTask) {
			super(res, bitmap);
			bitmapWorkerTaskReference = new WeakReference<BitmapWorkerTask>(
					bitmapWorkerTask);
		}
 
		public BitmapWorkerTask getBitmapWorkerTask() {
			return bitmapWorkerTaskReference.get();
		}
 
	}
	
	/**
	 * 获取传入的ImageView它所对应的BitmapWorkerTask。
	 */
	private BitmapWorkerTask getBitmapWorkerTask(ImageView imageView) {
        if (imageView != null) {
            Drawable drawable = imageView.getDrawable();
            if (drawable instanceof AsyncDrawable) {
                AsyncDrawable asyncDrawable = (AsyncDrawable) drawable;
                return asyncDrawable.getBitmapWorkerTask();
            }
        }
        return null;
    }
	
	/**
	 * 取消掉后台的潜在任务，当认为当前ImageView存在着一个另外图片请求任务时
	 * ，则把它取消掉并返回true，否则返回false。
	 */
    public boolean cancelPotentialWork(String url, ImageView imageView) {
        BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);
        if (bitmapWorkerTask != null) {
            String imageUrl = bitmapWorkerTask.imageUrl;
            if (imageUrl == null || !imageUrl.equals(url)) {
                bitmapWorkerTask.cancel(true);
            } else {
                return false;
            }
        }
        return true;
    }
 
	/**
	 * 将一张图片存储到LruCache中。
	 * 
	 * @param key
	 *            LruCache的键，这里传入图片的URL地址。
	 * @param drawable
	 *            LruCache的值，这里传入从网络上下载的BitmapDrawable对象。
	 */
	public void addBitmapToMemoryCache(String key, BitmapDrawable drawable) {
		if (getBitmapFromMemoryCache(key) == null) {
			mMemoryCache.put(key, drawable);
		}
	}
 
	/**
	 * 从LruCache中获取一张图片，如果不存在就返回null。
	 * 
	 * @param key
	 *            LruCache的键，这里传入图片的URL地址。
	 * @return 对应传入键的BitmapDrawable对象，或者null。
	 */
	public BitmapDrawable getBitmapFromMemoryCache(String key) {
		return mMemoryCache.get(key);
	}
 
	/**
	 * 异步下载图片的任务。
	 * 
	 * @author guolin
	 */
	class BitmapWorkerTask extends AsyncTask<String, Void, BitmapDrawable> {
 
		String imageUrl; 
		
		private WeakReference<ImageView> imageViewReference;
		
		public BitmapWorkerTask(ImageView imageView) {  
			imageViewReference = new WeakReference<ImageView>(imageView);
        }  
 
		@Override
		protected BitmapDrawable doInBackground(String... params) {
			imageUrl = params[0];
			// 在后台开始下载图片
			Bitmap bitmap = downloadBitmap(imageUrl);
			BitmapDrawable drawable = new BitmapDrawable(getContext().getResources(), bitmap);
			addBitmapToMemoryCache(imageUrl, drawable);
			return drawable;
		}
 
		@Override
		protected void onPostExecute(BitmapDrawable drawable) {
			ImageView imageView = getAttachedImageView();
            if (imageView != null && drawable != null) {  
                imageView.setImageDrawable(drawable);  
            } 
		}
		
		/**
		 * 获取当前BitmapWorkerTask所关联的ImageView。
		 */
		private ImageView getAttachedImageView() {
            ImageView imageView = imageViewReference.get();
            BitmapWorkerTask bitmapWorkerTask = getBitmapWorkerTask(imageView);
            if (this == bitmapWorkerTask) {
                return imageView;
            }
            return null;
        }
 
		/**
		 * 建立HTTP请求，并获取Bitmap对象。
		 * 
		 * @param imageUrl
		 *            图片的URL地址
		 * @return 解析后的Bitmap对象
		 */
		private Bitmap downloadBitmap(String imageUrl) {
			Bitmap bitmap = null;
			HttpURLConnection con = null;
			try {
				URL url = new URL(imageUrl);
				con = (HttpURLConnection) url.openConnection();
				con.setConnectTimeout(5 * 1000);
				con.setReadTimeout(10 * 1000);
				bitmap = BitmapFactory.decodeStream(con.getInputStream());
			} catch (Exception e) {
				e.printStackTrace();
			} finally {
				if (con != null) {
					con.disconnect();
				}
			}
			return bitmap;
		}
	}
}
```
那么我们一点点开始解析。首先刚才说到的，ImageView和BitmapWorkerTask之间要建立一个双向的弱引用关联，上述代码中已经建立好了。ImageView中可以获取到它所对应的BitmapWorkerTask，而BitmapWorkerTask也可以获取到它所对应的ImageView。

下面来看一下这个双向弱引用关联是怎么建立的。BitmapWorkerTask指向ImageView的弱引用关联比较简单，就是在BitmapWorkerTask中加入一个构造函数，并在构造函数中要求传入ImageView这个参数。不过我们不再直接持有ImageView的引用，而是使用WeakReference对ImageView进行了一层包装，这样就OK了。

但是ImageView指向BitmapWorkerTask的弱引用关联就没这么容易了，因为我们很难将BitmapWorkerTask的一个弱引用直接设置到ImageView当中。这该怎么办呢？这里使用了一个比较巧的方法，就是借助自定义Drawable的方式来实现。可以看到，我们自定义了一个AsyncDrawable类并让它继承自BitmapDrawable，然后重写了AsyncDrawable的构造函数，在构造函数中要求把BitmapWorkerTask传入，然后在这里给它包装了一层弱引用。那么现在AsyncDrawable指向BitmapWorkerTask的关联已经有了，但是ImageView指向BitmapWorkerTask的关联还不存在，怎么办呢？很简单，让ImageView和AsyncDrawable再关联一下就可以了。可以看到，在getView()方法当中，我们调用了ImageView的setImageDrawable()方法把AsyncDrawable设置了进去，那么ImageView就可以通过getDrawable()方法获取到和它关联的AsyncDrawable，然后再借助AsyncDrawable就可以获取到BitmapWorkerTask了。这样ImageView指向BitmapWorkerTask的弱引用关联也成功建立。

现在双向弱引用的关联已经建立好了，接下来就是逻辑判断的工作了。那么怎样通过逻辑判断来避免图片出现乱序的情况呢？这里我们引入了两个方法，一个是getBitmapWorkerTask()方法，这个方法可以根据传入的ImageView来获取到它对应的BitmapWorkerTask，内部的逻辑就是先获取ImageView对应的AsyncDrawable，再获取AsyncDrawable对应的BitmapWorkerTask。另一个是getAttachedImageView()方法，这个方法会获取当前BitmapWorkerTask所关联的ImageView，然后调用getBitmapWorkerTask()方法来获取该ImageView所对应的BitmapWorkerTask，最后判断，如果获取到的BitmapWorkerTask等于this，也就是当前的BitmapWorkerTask，那么就将ImageView返回，否则就返回null。最后，在onPostExecute()方法当中，只需要使用getAttachedImageView()方法获取到的ImageView来显示图片就可以了。

那么为什么做了这个逻辑判断之后，图片乱序的问题就可以得到解决呢？其实最主要的奥秘就是在getAttachedImageView()方法当中，它会使用当前BitmapWorkerTask所关联的ImageView来反向获取这个ImageView所关联的BitmapWorkerTask，然后用这两个BitmapWorkerTask做对比，如果发现是同一个BitmapWorkerTask才会返回ImageView，否则就返回null。那么什么情况下这两个BitmapWorkerTask才会不同呢？比如说某个图片被移出了屏幕，它的ImageView被另外一个新进入屏幕的图片重用了，那么就会给这个ImageView关联一个新的BitmapWorkerTask，这种情况下，上一个BitmapWorkerTask和新的BitmapWorkerTask肯定就不相等了，这时getAttachedImageView()方法会返回null，而我们又判断ImageView等于null的话是不会设置图片的，因此就不会出现图片乱序的情况了。

除此之外还有另外一个方法非常值得大家注意，就是cancelPotentialWork()方法，这个方法可以大大提高整个ListView图片加载的工作效率。这个方法接收两个参数，一个图片的url，一个ImageView。看一下它的内部逻辑，首先它也是调用了getBitmapWorkerTask()方法来获取传入的ImageView所对应的BitmapWorkerTask，接下来拿BitmapWorkerTask中的imageUrl和传入的url做比较，如果两个url不等的话就调用BitmapWorkerTask的cancel()方法，然后返回true，如果两个url相等的话就返回false。

那么这段逻辑是什么意思呢？其实并不复杂，两个url做比对时，如果发现是相同的，说明请求的是同一张图片，那么直接返回false，这样就不会再去启动BitmapWorkerTask来请求图片，而如果两个url不相同，说明这个ImageView被另外一张图片重新利用了，这个时候就调用了BitmapWorkerTask的cancel()方法把之前的请求取消掉，然后重新启动BitmapWorkerTask来去请求新图片。有了这个操作保护之后，就可以把一些已经移出屏幕的无效的图片请求过滤掉，从而整体提升ListView加载图片的工作效率。

# 解决方案三  使用NetworkImageView
---
前面两种解决方案都需要我们自己去做额外的逻辑处理，因为ImageView本身是不能自动解决这个问题的，但是如果我们使用NetworkImageView这个控件的话就非常简单了，它自身就已经考虑到了这个问题，我们直接使用它就可以了，不用做任何额外的处理也不会出现图片乱序的情况。

NetworkImageView是Volley当中提供的控件。

下面我们看一下如何用NetworkImageView来解决这个问题，首先需要修改一下image_item.xml文件，因为我们已经不再使用ImageView控件了，代码如下所示：
```xml
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
 
    <com.android.volley.toolbox.NetworkImageView
        android:id="@+id/image"
        android:layout_width="match_parent"
        android:layout_height="120dp"
        android:src="@drawable/empty_photo" 
        android:scaleType="fitXY"/>
 
</LinearLayout>
```
很简单，只是把ImageView替换成了NetworkImageView。然后修改ImageAdapter中的代码，如下所示：
```java
/**
 * 原文地址: http://blog.csdn.net/guolin_blog/article/details/45586553
 * @author guolin
 */
public class ImageAdapter extends ArrayAdapter<String> {
	
	ImageLoader mImageLoader;
 
	public ImageAdapter(Context context, int resource, String[] objects) {
		super(context, resource, objects);
		RequestQueue queue = Volley.newRequestQueue(context);
		mImageLoader = new ImageLoader(queue, new BitmapCache());
	}
 
	@Override
	public View getView(int position, View convertView, ViewGroup parent) {
		String url = getItem(position);
		View view;
		if (convertView == null) {
			view = LayoutInflater.from(getContext()).inflate(R.layout.image_item, null);
		} else {
			view = convertView;
		}
		NetworkImageView image = (NetworkImageView) view.findViewById(R.id.image);
		image.setDefaultImageResId(R.drawable.empty_photo);
		image.setErrorImageResId(R.drawable.empty_photo);
		image.setImageUrl(url, mImageLoader);
		return view;
	}
 
	/**
	 * 使用LruCache来缓存图片
	 */
	public class BitmapCache implements ImageCache {
 
		private LruCache<String, Bitmap> mCache;
 
		public BitmapCache() {
			// 获取应用程序最大可用内存
			int maxMemory = (int) Runtime.getRuntime().maxMemory();
			int cacheSize = maxMemory / 8;
			mCache = new LruCache<String, Bitmap>(cacheSize) {
				@Override
				protected int sizeOf(String key, Bitmap bitmap) {
					return bitmap.getRowBytes() * bitmap.getHeight();
				}
			};
		}
 
		@Override
		public Bitmap getBitmap(String url) {
			return mCache.get(url);
		}
 
		@Override
		public void putBitmap(String url, Bitmap bitmap) {
			mCache.put(url, bitmap);
		}
	}
}
```
没错，就是这么简单，一共60行左右的代码搞定一切！我们不需要自己再去写一个BitmapWorkerTask来处理图片的下载和显示，也不需要自己再去管理LruCache的逻辑，一切NetworkImageView都帮我们做好了。至于上面的代码我就不再做解释了，因为实在是太简单了。

那么当然了，虽然现在没有做任何额外的逻辑处理，但是也根本不会出现图片乱序的情况，因为NetworkImageView在内部都帮我们处理掉了。不过大家可能都很好奇，NetworkImageView到底是如何做到的呢？那么就让我们来分析一下它的源码吧。

NetworkImageView中开始加载图片的代码是setImageUrl()方法，源码分析就从这里开始吧，如下所示：
```java
/**
 * Sets URL of the image that should be loaded into this view. Note that calling this will
 * immediately either set the cached image (if available) or the default image specified by
 * {@link NetworkImageView#setDefaultImageResId(int)} on the view.
 *
 * NOTE: If applicable, {@link NetworkImageView#setDefaultImageResId(int)} and
 * {@link NetworkImageView#setErrorImageResId(int)} should be called prior to calling
 * this function.
 *
 * @param url The URL that should be loaded into this ImageView.
 * @param imageLoader ImageLoader that will be used to make the request.
 */
public void setImageUrl(String url, ImageLoader imageLoader) {
    mUrl = url;
    mImageLoader = imageLoader;
    // The URL has potentially changed. See if we need to load it.
    loadImageIfNecessary(false);
}
```
setImageUrl()方法中并没有几行代码，让人值得留意的是loadImageIfNecessary()这个方法，看上去具体加载图片的逻辑就是在这里进行的，那么我们就跟进去瞧一瞧：
```java
/**
 * Loads the image for the view if it isn't already loaded.
 * @param isInLayoutPass True if this was invoked from a layout pass, false otherwise.
 */
private void loadImageIfNecessary(final boolean isInLayoutPass) {
    int width = getWidth();
    int height = getHeight();
 
    boolean isFullyWrapContent = getLayoutParams() != null
            && getLayoutParams().height == LayoutParams.WRAP_CONTENT
            && getLayoutParams().width == LayoutParams.WRAP_CONTENT;
    // if the view's bounds aren't known yet, and this is not a wrap-content/wrap-content
    // view, hold off on loading the image.
    if (width == 0 && height == 0 && !isFullyWrapContent) {
        return;
    }
 
    // if the URL to be loaded in this view is empty, cancel any old requests and clear the
    // currently loaded image.
    if (TextUtils.isEmpty(mUrl)) {
        if (mImageContainer != null) {
            mImageContainer.cancelRequest();
            mImageContainer = null;
        }
        setDefaultImageOrNull();
        return;
    }
 
    // if there was an old request in this view, check if it needs to be canceled.
    if (mImageContainer != null && mImageContainer.getRequestUrl() != null) {
        if (mImageContainer.getRequestUrl().equals(mUrl)) {
            // if the request is from the same URL, return.
            return;
        } else {
            // if there is a pre-existing request, cancel it if it's fetching a different URL.
            mImageContainer.cancelRequest();
            setDefaultImageOrNull();
        }
    }
 
    // The pre-existing content of this view didn't match the current URL. Load the new image
    // from the network.
    ImageContainer newContainer = mImageLoader.get(mUrl,
            new ImageListener() {
                @Override
                public void onErrorResponse(VolleyError error) {
                    if (mErrorImageId != 0) {
                        setImageResource(mErrorImageId);
                    }
                }
 
                @Override
                public void onResponse(final ImageContainer response, boolean isImmediate) {
                    // If this was an immediate response that was delivered inside of a layout
                    // pass do not set the image immediately as it will trigger a requestLayout
                    // inside of a layout. Instead, defer setting the image by posting back to
                    // the main thread.
                    if (isImmediate && isInLayoutPass) {
                        post(new Runnable() {
                            @Override
                            public void run() {
                                onResponse(response, false);
                            }
                        });
                        return;
                    }
 
                    if (response.getBitmap() != null) {
                        setImageBitmap(response.getBitmap());
                    } else if (mDefaultImageId != 0) {
                        setImageResource(mDefaultImageId);
                    }
                }
            });
 
    // update the ImageContainer to be the new bitmap container.
    mImageContainer = newContainer;
}
```
这里在第43行调用了ImageLoader的get()方法来去请求图片，get()方法会返回一个ImageContainer对象，这个对象封装了图片请求地址、Bitmap等数据，每个NetworkImageView中都会对应一个ImageContainer。然后在第31行我们看到，这里从ImageContainer对象中获取封装的图片请求地址，并拿来和当前的请求地址做对比，如果相同的话说明这是一条重复的请求，就直接return掉，如果不同的话就调用cancelRequest()方法将请求取消掉，然后将图片设置为默认图片并重新发起请求。

那么解决图片乱序最核心的逻辑就在这里了，其实NetworkImageView的解决思路还是比较简单的，就是如果这个控件已经被移出了屏幕且被重新利用了，那么就把之前的请求取消掉，仅此而已。

而我们都知道，在通常情况下，仅仅这么处理可能是解决不了问题的，因为Java的线程无法保证一定可以中断，即使像第二种解决方案里使用的BitmapWorkerTask的cancel()方法，也不能保证一定可以把请求取消掉，所以还需要使用弱引用关联的处理方式。但是在NetworkImageView当中就可以这么任性，仅仅调用cancelRequest()方法把请求取消掉就可以了，这主要是得益于Volley的出色设计。由于Volley在网络方面的封装非常优秀，它可以保证，只要是取消掉的请求，就绝对不会进行回调，既然不会回调，那么也就不会回到NetworkImageView当中，自然也就不会出现乱序的情况了。

需要注意的是，Volley只是保证取消掉的请求不会进行回调而已，但并没有说可以中断任何请求。由此可见即使是Volley也无法做到中断一个正在执行的线程，如果有一个线程正在执行，Volley只会保证在它执行完之后不会进行回调，但在调用者看来，就好像是这个请求就被取消掉了一样。

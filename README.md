# AndroidSwigIdiom
Android swig idioms

```java
// 어떤 경우를 고민해야 하는지 생각해보자.
class BitmapUsage {
	// Native에 넘어간 동안 Bitmap이 garbage collection 대상이 아니어야 한다.
	public native void useBitmap(Bitmap bitmap);

	// 메모리를 jvm 내에 잡아야 따로 Bitmap을 release 시켜줄 필요가 없다.
	// 오래 걸리는 연산이면 아래처럼 async하게 감싸서 사용하자.
	public native Bitmap loadBitmap();

	// Native단에 java의 callback함수를 넘길 수 있나?
	public native void loadBitmap(BitmapLoadCompleteCallback callback) {
		final Handler handler = new Handler();
		new Thread() {
			@Override
			public void run() {
				Bitmap bitmap = loadBitmap();
				handler.post(new Runnalble() {
					if (callback != null) {
						callback.onBitmapLoadComplete(bitmap);
					}
				});
			}
		}.start();
	}	

	interface BitmapLoadCompleteCallback {
		void onBitmapLoadComplete(Bitmap bitmap);
	}
}
```

- Native code에서 JVM에 memory를 할당할 수 있다. 그래서 native에서 잡는 memory에 따라 GC가 trigger될 수 있다. 
  [Overriding new and delete to allocate from Java heap](http://www.swig.org/Doc3.0/Java.html#Java_heap_allocations)

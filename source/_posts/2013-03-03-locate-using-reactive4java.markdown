---
layout: post
title: "reactive4java で位置を取得し続ける"
date: 2012-10-03 15:21
comments: true
categories: [Android, reactive4java, Java, ReactiveExtensions]
---
[前回](http://qiita.com/items/07762776102dbc84b1c7)に続き、 reactive4java ネタ。
調子に乗って Android で位置を取得し続けるのを reactive4java を使ってやってみた。

<!--more-->

```java LocationFunctions.java
/**
 * 位置を取得し続ける(finish は呼ばれない)
 */
public static Observable<Location> getCurrentLocationAsObservable(			
		final Context context, final String provider) {
	return Reactive.createWithCloseable(new Func1<Observer<? super Location>, Closeable>() {
		private volatile boolean _stop = false;

		@Override
		public Closeable invoke(final Observer<? super Location> observer) {
			final LocationManager locMan = (LocationManager)context.getSystemService(Context.LOCATION_SERVICE);
			
			final LocationListener listener = new LocationListener() {
				
				@Override
				public void onStatusChanged(String provider, int status, Bundle extras) { }
				
				@Override
				public void onProviderEnabled(String provider) { }
				
				@Override
				public void onProviderDisabled(String provider) {
					observer.error(
						new InvalidParameterException("LocationProvider disabled."));
				}
				
				@Override
				public void onLocationChanged(Location location) {
					if (_stop) {
						return;
					}
					
					// 発火
					observer.next(location);
				}
			};
			
			// 位置取得開始
			locMan.requestLocationUpdates(provider, 0, 0, listener, Looper.getMainLooper());

			return new Closeable() {
				@Override
				public void close() throws IOException {
					if (_stop) {
						return;
					}
					_stop = true;
					locMan.removeUpdates(listener);
					observer.finish();
				}
			};
		}
	});
}
```

使い方は、方位の時とほとんど同じ。パラメータが Float から Location に代わっただけ。

``_stop`` フラグは、Listener を unregister しても溜まってるデータは流しちゃうんじゃないか、という事で用意した。
方位のやつは AtomicBoolean を使ったけどこっちは volatile でやってる。確かどっかで 'AtomicBoolean の方が確実に Atomic' って言ってた気がするけど、そもそもそんなに神経質になるところじゃないか。

Observable にすることで、「ｎ秒間位置を取得して貯めて、その中で一番精度の良いものを通す」みたいなことも簡単にできる。

Android の場合、購読開始時に registerXXXListener、Close で unregisterXXXEventListener てのがひとつのパターン。
たぶん BroadcastReceiver にも適用できる。

XXXListener も BroadcastReceiver も Observable でラップしちゃえば、その後は同じように扱えるので便利♪

##追記 9.27
実はこのプログラム、Cold でしたー。というわけで、Hot についての記事を書きました。
* [Cold を Hot にできる。そう、Publish ならね。 - Qiita](http://qiita.com/items/3a7bda9d0fdcb9248800)

##参考
* [reactive4java で端末の方位を取得しつづける - Qiita](http://qiita.com/items/07762776102dbc84b1c7)
* [Java: volatile boolean vs AtomicBoolean - Stack Overflow](http://stackoverflow.com/questions/3786825/java-volatile-boolean-vs-atomicboolean)
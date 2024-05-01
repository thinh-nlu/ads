### Thư viện gắn ads
```implementation 'com.github.chiennq44:AdsLibrary:1.0.1'```
### Thư viện Rate (tham khảo)
```https://github.com/quangchienictu/RateLibrary```

### Thư viện appflyer (Bắt buộc)
``` appsflyer
implementation 'com.appsflyer:af-android-sdk:6.11.0'
implementation 'com.appsflyer:adrevenue:6.9.1'
```
### Tạo mới file ads_id.xml trong thư mục res/values để lưu các id của ads
>Ví dụ:
```
<resources>
    <string name="app_id" translatable="false">ca-app-pub-3940256099942544~3347511713</string>
    <string name="intersitial_splash" translatable="false">ca-app-pub-3940256099942544/1033173712</string>
    <string name="inter_intro" translatable="false">ca-app-pub-3940256099942544/1033173712</string>
    <string name="banner_main" translatable="false">ca-app-pub-3940256099942544/6300978111</string>
    <string name="banner_reader" translatable="false">ca-app-pub-3940256099942544/6300978111</string>
    <string name="inter_splash_app" translatable="false">ca-app-pub-3940256099942544/1033173712</string>
    <string name="appopen_resume" translatable="false">ca-app-pub-3940256099942544/3419835294</string>
    <string name="intersitial_file" translatable="false">ca-app-pub-3940256099942544/1033173712</string>
    <string name="Native_language" translatable="false">ca-app-pub-3940256099942544/2247696110</string>
    <string name="Native_language1" translatable="false">ca-app-pub-3940256099942544/2247696110</string>
    <string name="Native_permission" translatable="false">ca-app-pub-3940256099942544/2247696110</string>
	//Sử dụng key này cho tất cả các app
    <string name="app_flyer" translatable="false">LikYKU2zUYTct7BoQ7MuJY</string>
</resources>
```

>Lưu ý
Sử dụng key app_flyer sau cho tất cả các app
```
<string name="app_flyer" translatable="false">LikYKU2zUYTct7BoQ7MuJY</string>
```

### Config file Application

```
class MyApp : AdsApplication() {
    override fun onCreate() {
        super.onCreate()

        AppOpenManager.getInstance().disableAppResumeWithActivity(SplashActivity::class.java)
        
        //Appflyer
        AppsFlyerLib.getInstance().init(getString(R.string.app_flyer), null, this)
        AppsFlyerLib.getInstance().start(this)
    }

    override fun enableAdsResume(): Boolean {
        return true
    }

    override fun getListTestDeviceId(): MutableList<String>? {
        return null
    }

    override fun getResumeAdId(): String {
        return getString(R.string.appopen_resume)
    }

    override fun buildDebug(): Boolean {
        return false
    }
}
```

### Ẩn toàn bộ thanh điều hướng
```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        window.decorView.systemUiVisibility = View.SYSTEM_UI_FLAG_HIDE_NAVIGATION or
                View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY
    }
	
override fun onWindowFocusChanged(hasFocus: Boolean) {
	super.onWindowFocusChanged(hasFocus)
	if (hasFocus) {
		fullScreenImmersive(window)
	}
}

open fun fullScreenImmersive(window: Window?) {
        if (window != null) {
            fullScreenImmersive(window.decorView)
        }
    }

    open fun fullScreenImmersive(view: View) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
            val uiOptions =
                View.SYSTEM_UI_FLAG_HIDE_NAVIGATION or View.SYSTEM_UI_FLAG_IMMERSIVE_STICKY
            view.systemUiVisibility = uiOptions
        }
    }
```

### Inter Splash
``` 
 var interCallback: InterCallback? = null
 override fun onCreate(savedInstanceState: Bundle?) {
    interCallback = object : InterCallback() {
            override fun onNextAction() {
                super.onNextAction()
                //TODO: add logic to go to next screen
            }
        }
    Handler(Looper.getMainLooper()).postDelayed({
        Admob.getInstance()
            .loadSplashInterAds2(this, getString(R.string.inter_splash_app), 3000, interCallback)
           
        },2000)
 }

  override fun onResume() {
        super.onResume()
        Admob.getInstance().onCheckShowSplashWhenFail(this, interCallback, 1000)
    }
 ```

 ### Ads Inter
1. Tạo lớp Config và khai báo các ads inter, số lượng tùy thuộc vào kịch bản của ads
 ```
 class AdsConfig {
    companion object{
        var mInterstitialAdIntro: InterstitialAd? = null
        var mInterstitialAdView: InterstitialAd? = null
        fun loadAdsInterIntro(context: Context){
            Admob.getInstance().loadInterAds(context,context.getString(R.string.inter_intro),object : InterCallback(){
                override fun onInterstitialLoad(interstitialAd: InterstitialAd?) {
                    super.onInterstitialLoad(interstitialAd)
                    mInterstitialAdIntro = interstitialAd
                }
            })
        }

        fun loadAdsInterView(context: Context){
            Admob.getInstance().loadInterAds(context,context.getString(R.string.intersitial_file),object : InterCallback(){
                override fun onInterstitialLoad(interstitialAd: InterstitialAd?) {
                    super.onInterstitialLoad(interstitialAd)
                    mInterstitialAdView = interstitialAd
                }
            })
        }
    }
}
```
2. Load ads inter
```
override fun onCreate(savedInstanceState: Bundle?) {
	if (AdsConfig.mInterstitialAdView==null){
		AdsConfig.loadAdsInterView(this@IntroActivity)
	}
}
```
3. Show ads inter
```
if (SharePrefUtils.haveNetworkConnection(this)){
	Admob.getInstance().showInterAds(this@IntroActivity,AdsConfig.mInterstitialAdIntro,object : InterCallback(){
		override fun onNextAction() {
			super.onNextAction()
			//TODO: add logic to go to next screen
			 AdsConfig.loadAdsInterView(this@HomeScreenActivity)
		}
	})
}
```
> Lưu ý: Đối với tất cả các màn không phải màn intro, sau khi show ads inter thì bắt buộc phải load lại ads inter đó.

### Ads native

##### Ads native tại màn language đầu tiên

```
override fun onCreate(savedInstanceState: Bundle?) {
	loadNative()
}

private fun loadNative() {
	if (SharePrefUtils.haveNetworkConnection(this)) {
		try {
			var listID: MutableList<String?> = ArrayList()
			listID.add(getString(R.string.Native_language))
			listID.add(getString(R.string.Native_language1))

			Admob.getInstance().loadNativeAdFloor(this,listID, object : NativeCallback() {
				override fun onNativeAdLoaded(nativeAd: NativeAd?) {
					val adView = LayoutInflater.from(this@LanguageStartActivity)
						.inflate(R.layout.native_ads, null) as NativeAdView
					binding.nativeAds!!.removeAllViews()
					binding.nativeAds!!.addView(adView)
					Admob.getInstance().pushAdsToViewCustom(nativeAd, adView)
				}

				override fun onAdFailedToLoad() {
					binding.nativeAds!!.removeAllViews()
				}
			})
		}catch (e:Exception){
			binding.nativeAds!!.removeAllViews()
		}


	}else{
		binding.nativeAds!!.removeAllViews()
	}

}
```

#####Ads native tại các màn còn lại
```
override fun onCreate(savedInstanceState: Bundle?) {
	loadNative()
}
private fun loadNative() {
	if (SharePrefUtils.haveNetworkConnection(this)) {
		Admob.getInstance().loadNativeAd(this,getString(com.excel.xlsx.reader.office.R.string.Native_intro), object : NativeCallback() {
			override fun onNativeAdLoaded(nativeAd: NativeAd?) {
				val adView = LayoutInflater.from(this@IntroActivity)
					.inflate(com.excel.xlsx.reader.office.R.layout.native_ads, null) as NativeAdView
				binding.nativeAds!!.removeAllViews()
				binding.nativeAds!!.addView(adView)
				Admob.getInstance().pushAdsToViewCustom(nativeAd, adView)
			}

			override fun onAdFailedToLoad() {
				binding.nativeAds!!.removeAllViews()
			}
		})
	}else{
		binding.nativeAds!!.removeAllViews()
	}

}
```
#### layout
```
<androidx.constraintlayout.widget.ConstraintLayout>
...
<RelativeLayout
        android:id="@+id/rl_native"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_alignParentBottom="true"
        app:layout_constraintBottom_toBottomOf="parent">
        <FrameLayout
            android:id="@+id/native_ads"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            >
            <include layout="@layout/ads_native_shimer"/>
        </FrameLayout>
    </RelativeLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
```
##### layout native ads

- Normal
native_ads.xml
```
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.gms.ads.nativead.NativeAdView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <RelativeLayout
        android:background="@drawable/border_dialog_custom"
        android:id="@+id/ad_unit_content"
        android:layout_width="fill_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">
        <LinearLayout
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">
            <Button
                android:background="@drawable/border_native_cus"
                android:id="@+id/ad_call_to_action"
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:layout_marginHorizontal="@dimen/_10sdp"
                android:layout_marginTop="@dimen/_12sdp"
                android:layout_marginBottom="@dimen/_5sdp"
                android:gravity="center"
                android:text="Cài Đặt"
                android:textColor="#ffffff"
                android:textSize="22sp"
                android:textStyle="bold" />

            <LinearLayout
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:padding="8dip">

                <ImageView
                    android:id="@+id/ad_app_icon"
                    android:layout_width="40dip"
                    android:layout_height="40dip"
                    android:adjustViewBounds="true"
                    android:src="@color/colorPrimary" />

                <LinearLayout
                    android:layout_width="fill_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginLeft="5dip"
                    android:orientation="vertical">

                    <TextView
                        android:id="@+id/ad_headline"
                        android:layout_width="fill_parent"
                        android:layout_height="wrap_content"
                        android:ellipsize="end"
                        android:maxLines="2"
                        android:textColor="@color/black"
                        android:text="hellop .this ius dsa dsadsa "
                        android:textSize="11sp" />

                    <LinearLayout
                        android:layout_width="fill_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal">

                        <TextView
                            android:id="@+id/ad_advertiser"
                            android:layout_width="0dp"
                            android:layout_height="wrap_content"
                            android:layout_weight="1"
                            android:gravity="bottom"
                            android:lines="1"
                            android:textColor="@color/black"
                            android:text=" dá dsa dsa ds a dsads ad sad sa"
                            android:textSize="11sp"
                            android:textStyle="bold" />
                    </LinearLayout>
                </LinearLayout>
            </LinearLayout>

            <TextView
                android:id="@+id/ad_body"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:layout_marginLeft="10dip"
                android:layout_marginRight="10dip"
                android:maxLines="3"
                android:textColor="@color/black"
                android:text="@string/appbar_scrolling_view_behavior"
                android:textSize="11sp" />

            <com.google.android.gms.ads.nativead.MediaView
                android:id="@+id/ad_media"
                android:layout_width="fill_parent"
                android:layout_height="120dp"
                android:layout_gravity="center_horizontal"
                android:layout_weight="1"
                android:minWidth="120dp"
                android:minHeight="120dp"
                android:layout_marginBottom="@dimen/_10sdp"/>


        </LinearLayout>

        <TextView style="@style/AppTheme.Ads.Cus" />

    </RelativeLayout>
</com.google.android.gms.ads.nativead.NativeAdView>
```
- Small
native_ads_small.xml
```
<?xml version="1.0" encoding="utf-8"?>
<com.google.android.gms.ads.nativead.NativeAdView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="fill_parent"
    android:layout_height="wrap_content">

    <RelativeLayout
        android:id="@+id/ad_unit_content"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:background="@drawable/bg_native_home"
        android:orientation="vertical">

        <LinearLayout
            android:layout_width="fill_parent"
            android:layout_height="wrap_content"
            android:orientation="vertical">

            <LinearLayout
                android:layout_width="fill_parent"
                android:layout_height="wrap_content"
                android:orientation="horizontal"
                android:paddingStart="25dip"
                android:paddingTop="8dip"
                android:paddingEnd="8dip"
                android:paddingBottom="8dip">

                <ImageView
                    android:id="@+id/ad_app_icon"
                    android:layout_width="35dip"
                    android:layout_height="35dip"
                    android:adjustViewBounds="true"
                    android:src="@color/colorPrimary" />

                <LinearLayout
                    android:layout_width="fill_parent"
                    android:layout_height="wrap_content"
                    android:layout_marginStart="5dip"
                    android:orientation="vertical">

                    <TextView
                        android:id="@+id/ad_headline"
                        android:layout_width="fill_parent"
                        android:layout_height="wrap_content"
                        android:ellipsize="end"
                        android:maxLines="2"
                        android:text=""
                        android:textColor="@color/color_F5871A"
                        android:textSize="@dimen/_10sdp" />


                    <TextView
                        android:id="@+id/ad_advertiser"
                        android:layout_width="match_parent"
                        android:layout_height="0dp"
                        android:layout_weight="1"
                        android:gravity="bottom"
                        android:ellipsize="end"
                        android:maxLines="2"
                        android:textColor="@color/colorAds"
                        android:textSize="12sp"
                        android:textStyle="bold" />

                    <LinearLayout
                        android:layout_width="fill_parent"
                        android:layout_height="wrap_content"
                        android:orientation="horizontal">

                        <TextView
                            android:id="@+id/ad_body"
                            android:layout_width="match_parent"
                            android:layout_height="wrap_content"
                            android:ellipsize="end"
                            android:maxLines="2"
                            android:textSize="12sp" />

                    </LinearLayout>
                </LinearLayout>
            </LinearLayout>


            <Button
                android:id="@+id/ad_call_to_action"
                android:layout_width="fill_parent"
                android:layout_height="@dimen/_35sdp"
                android:layout_marginHorizontal="@dimen/_20sdp"
                android:layout_marginVertical="@dimen/_10sdp"
                android:background="@drawable/border_native_cus"
                android:gravity="center"
                android:text="Install"
                android:textColor="@color/colorWhite"
                android:textSize="@dimen/_12sdp"
                android:textStyle="bold" />
        </LinearLayout>

        <TextView style="@style/AppTheme.Ads.Cus" />

    </RelativeLayout>


</com.google.android.gms.ads.nativead.NativeAdView>
```
style button and ad text

```
    <style name="AppTheme.Ads.Cus" parent="@style/AdsAppTheme">
        <item name="android:textSize">11.0sp</item>
        <item name="android:textColor">#ffffff</item>
        <item name="android:gravity">center</item>
        <item name="android:layout_gravity">left</item>
        <item name="android:background">@drawable/ads_icon_cus</item>
        <item name="android:paddingLeft">3.0dip</item>
        <item name="android:paddingRight">6.0dip</item>
        <item name="android:paddingBottom">1.0dip</item>
        <item name="android:layout_width">wrap_content</item>
        <item name="android:layout_height">wrap_content</item>
        <item name="android:minWidth">15.0px</item>
        <item name="android:minHeight">15.0px</item>
        <item name="android:text">Ad</item>
    </style>

```

boder_native_cus.xml
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid android:color="#278752"/>
    <corners android:radius="10dp"/>

</shape>
```

![](Screenshot_1686298224-1.png)

>Lưu ý:
```
bổ sung 1 chút về checklist "Native language first open"
1. Header: 
+ Language font size lớn hơn text ở dưới, 
+ icon V cùng màu với button ad, 
+ không có line giữa header và list ngôn ngữ ở dưới
2. List language: 
+ có icon cờ, 
+ check box active cùng màu button ad
3. Ads native: 
+ button ở trên, bo góc vuông, kích thước cao bằng 1 item trong list ngôn ngữ
+ title ad ở góc trái phía trên
+Màu của 3 vùng khoanh đỏ bắt buộc phải cùng màu với màu chủ đạo của apps.
+ ad cps màu nền khác giao diện app (thường là đục hơn 1 chút)
```

### Banner
```
if (SharePrefUtils.haveNetworkConnection(this)) {
	Admob.getInstance().loadBanner(this, getString(R.string.banner_main))
}else{
	binding.include.visibility = View.GONE
}
```
layout
```
<RelativeLayout>
...

	<View
        app:layout_constraintBottom_toTopOf="@+id/rl_banner"
        android:layout_above="@+id/rl_banner"
        android:layout_width="match_parent"
        android:layout_height="1sp"
        android:background="@color/black"/>
    <RelativeLayout
        android:id="@+id/rl_banner"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        app:layout_constraintBottom_toBottomOf="parent"
        android:layout_alignParentBottom="true">

        <include
            android:id="@+id/include"
            layout="@layout/layout_banner"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            />
    </RelativeLayout>
</RelativeLayout>
```
####Gắn firebase và firebase messaging (Bắt buộc)
Có thể sử dụng luôn class `MyFirebaseMessagingService` sau để gắn trong file `AndroidManifest.xml`
```
class MyFirebaseMessagingService : FirebaseMessagingService() {
    override fun onNewToken(token: String) {
        Log.d("SERVICE", "Refreshed token: $token")
    }

    override fun onMessageReceived(remoteMessage: RemoteMessage) {
        Log.d(TAG, "From: " + remoteMessage.from)

        if (remoteMessage.data.size > 0) {
            Log.d(TAG, "Message data payload: " + remoteMessage.data)
        }

        if (remoteMessage.notification != null) {
            Log.d(
                TAG, "Message Notification Body: " + remoteMessage.notification!!
                    .body
            )
        }
    }

    companion object {
        private const val TAG = "SERVICE"
    }
}
```

  

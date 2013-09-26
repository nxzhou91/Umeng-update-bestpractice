
#自动更新最佳实践
## 忽略更新

使用友盟自动更新模块实现忽略更新的示例：
有时候对于某些更新很小的版本，用户并没有更新意愿，但是即使选择了下次再说之后，下次启动应用的时候仍然会弹窗提醒用户，友盟自动更新模块在V2.3增加了忽略更新功能，但不是默认开启的，这里给出忽略更新的使用说明。

修改`umeng_update_dialog.xml`中相关控件

```xml
<Button
	android:id="@+id/umeng_update_id_close"
	android:layout_width="wrap_content"
	android:layout_height="wrap_content"
	android:layout_alignParentRight="true"
	android:layout_centerVertical="true"
	android:layout_marginRight="10dp"
	android:focusable="true"
	android:visibility="gone"
	android:background="@drawable/umeng_update_button_close_selector"/>
```

去除其中`android:visibility="gone"`属性，该控件是位于dialog右上角的关闭按钮，用来代替以后再说功能按钮。

修改`umeng_update_dialog.xml`中相关控件

```xml
<Button
	android:id="@+id/umeng_update_id_ignore"
	android:visibility="gone"
	android:layout_width="0dp"
	android:layout_height="wrap_content"
	android:layout_margin="5dp"
	android:layout_weight="1"
	android:background="@drawable/umeng_update_button_cancel_selector"
	android:gravity="center"
	android:padding="12dp"
	android:text="@string/UMNotNow"
	android:focusable="true"
	android:textColor="#AAABAF" />
```

去除其中`android:visibility="gone"`属性，该控件是实现忽略更新功能的按钮。
	
同时修改`umeng_update_dialog.xml`中相关控件

```xml
<Button
	android:id="@+id/umeng_update_id_cancel"
	android:layout_width="0dp"
	android:layout_height="wrap_content"
	android:layout_margin="5dp"
	android:layout_weight="1"
	android:background="@drawable/umeng_update_button_cancel_selector"
	android:gravity="center"
	android:padding="12dp"
	android:text="@string/UMNotNow"
	android:focusable="true"
	android:textColor="#AAABAF" />
```

在其中增加`android:visibility="gone"`属性，该控件是实现以后再说功能的按钮，隐藏该按钮的原因是在dialog中同时显示3个按钮会显得过于拥挤，该功能可以通过点击右上角的关闭按钮或者按回退键来实现。
	
## 手动更新

使用友盟自动更新模块实现手动更新的示例：
除了在进入应用的第一个页面使用自动更新检测版本更新以外，很多应用会在应用的设置页面提供手动检测更新的功能，并手动处理更新逻辑，这里以使用控件点击监听为例展示怎用使用手动更新的功能。

```java
private View.OnClickListener listener = new View.OnClickListener() {
	public void onClick(View v) {
		//禁止自动弹框，手动处理相关逻辑
		UmengUpdateAgent.setUpdateAutoPopup(false);
		UmengUpdateAgent.setUpdateListener(new UmengUpdateListener() {
					
			@Override
			public void onUpdateReturned(int updateStatus, UpdateResponse updateInfo) {
				switch (updateStatus) {
				case 0: // has update
					UmengUpdateAgent.showUpdateDialog(mContext, updateInfo);
					break;
				case 1: // has no update
					Toast.makeText(mContext, "没有更新", Toast.LENGTH_SHORT)
							.show();
					break;
				case 2: // none wifi
					Toast.makeText(mContext, "没有wifi连接， 只在wifi下更新", Toast.LENGTH_SHORT)
							.show();
					break;
				case 3: // time out
					Toast.makeText(mContext, "超时", Toast.LENGTH_SHORT)
							.show();
					break;
				case 4: // is updating
					// We show toast already, do not show again.
					break;
				}
			}
		});
		//当希望手动更新可以在非wifi环境下工作，可以无视版本忽略时，使用此方法
		UmengUpdateAgent.forceUpdate(mContext);
	}
};
```

因为这些关于更新的设置是全局的，所以如果不希望手动更新中的设置运用到自动更新中去，则可以在当前页面退出时将设置恢复成默认，或者在自动更新之前恢复设置，例如：
```java
@Override
protected void onStop() {
	super.onStop();
	UmengUpdateAgent.setUpdateOnlyWifi(true);
	UmengUpdateAgent.setUpdateAutoPopup(true);
	UmengUpdateAgent.setUpdateListener(null);
}
```
或
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	UmengUpdateAgent.setUpdateOnlyWifi(true);
	UmengUpdateAgent.setUpdateAutoPopup(true);
	UmengUpdateAgent.setUpdateListener(null);
	UmengUpdateAgent.update(this);
}
```
	
## 首次启动不更新

有的时候，开发者会希望在应用在下载后第一次启动时不要启动自动更新，这样即使下载的不是最近版本也不会在首次启动时提醒更新。

在应用程序入口`Activity`里的`OnCreate()`方法中调用
```java
@Override
protected void onCreate(Bundle savedInstanceState) {
	super.onCreate(savedInstanceState);
	SharedPreferences sharedPreferences = getSharedPreferences(SHARED_PREFERENCES_NAME, MODE_PRIVATE);
	boolean isFirstStart = sharedPreferences.getBoolean(KEY_FIRST_START, true);
	if (isFirstStart) {
		sharedPreferences.edit().putBoolean(KEY_FIRST_START, false).commit();
	} else {
		UmengUpdateAgent.update(this);
	}
```
其中`SHARED_PREFERENCES_NAME`为sharedPreference文件名，`KEY_FIRST_START`为判断是否为第一次启动值的key
	

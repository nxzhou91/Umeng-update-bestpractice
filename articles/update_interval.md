## 按间隔更新

默认总是在程序启动的时候检测版本，浪费流量，那么加一些自己的小策略，比如按一定时间间隔更新：

```
	private static final String KEY_LAST_UPDATE_TIME = "umeng_last_update_time";

	/**
	 * 自动更新，在main activity 中调用，此方法会请求服务器，检查是否有最新版本
	 * 
	 * @param context
	 *            当前Activity
	 * @param internal
	 *            控制自动更新请求的频率，单位毫秒,eg：update(context,24*60*60*1000) ，每天更新一次
	 */
	public static void update(Context context, final long internal) {
		final Context mContext = context;
		if (mContext == null) {
			Log.i(TAG, "unexpected null Context");
			return;
		}

		SharedPreferences preference = getUpdateSettingPreferences(mContext);
		long lastUpdateTime = preference.getLong(KEY_LAST_UPDATE_TIME, 0);
		long now = System.currentTimeMillis();

		if ((now - lastUpdateTime) > internal) {
			update(mContext);
			preference.edit().putLong(KEY_LAST_UPDATE_TIME, now).commit();
		}
	}
```

这样调用 `update( mContext, 24*60*60*1000 );` 就可以实现按天更新。

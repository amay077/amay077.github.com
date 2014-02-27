---
layout: post
title: "手抜きカラー選択ダイアログを作る"
date: 2014-02-27 21:38
comments: true
categories: [Android]
---
Android で「定められた色リストから１つ選択する」ダイアログを作ります。
<!--more-->
## さっそくコード

``AlertDialog`` には自作の ``Adapter`` が設定できるので、「項目値を背景色にする Adapter」を作って設定すれば OK でした。

```java MainActivity.java
public class MainActivity extends Activity {
	final private Integer[] _colors = { Color.RED, Color.BLUE, Color.GREEN }; // 色リスト
	final int ID_COLOR_PICKER = 1;

    // 背景に色リストを適用する ListAdapter
	static class ColorListAdapter extends ArrayAdapter<Integer> {
		public ColorListAdapter(Context context, Integer[] colors) {
			super(context, android.R.layout.simple_list_item_1, colors);
		}
		
		@Override
		public View getView(int position, View convertView, ViewGroup parent) {
			TextView view = (TextView)super.getView(position, convertView, parent);
			view.setBackgroundColor(getItem(position));
			view.setTextColor(Color.TRANSPARENT); // 色値が表示されないように隠す
			return view;
		}
	}
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        findViewById(R.id.button1).setOnClickListener(new OnClickListener() {
			@Override
			public void onClick(View v) {
				showDialog(ID_COLOR_PICKER); // ダメよ直接 Dialog.show() しちゃ
			}
		});
    }
    
    @Override
    protected Dialog onCreateDialog(int id) {
    	switch (id) {
		case ID_COLOR_PICKER:
	    	ColorListAdapter adapter = new ColorListAdapter(MainActivity.this, _colors);

	    	return new AlertDialog.Builder(MainActivity.this)
		    	.setTitle("Select color")
		    	.setAdapter(adapter, new DialogInterface.OnClickListener() {
					@Override
					public void onClick(DialogInterface dialog, int which) {
		    	        Toast.makeText(MainActivity.this, 
		    	        		"You picked index=" + which, Toast.LENGTH_LONG).show();
					}
				}).create();

		default:
			return super.onCreateDialog(id);
		}
    }
}
```

キャストしてるところがちょっと不安ですけど、まあいいでしょう。

## 動かしてみる

こんな感じです。だいぶ味気ないけど、要件は満たします。

![](https://dl.dropboxusercontent.com/u/264530/qiita/making_shoddy_color_picking_dialog_01.png)


## Xamarin でも作ってみる

本題とまったく関係ありませんが、同じものを Xamarin.Android で作ると、こうなります。

```csharp MainActivity.cs
[Activity(Label = "ColorPickerSample", MainLauncher = true)]
public class MainActivity : Activity
{
    readonly Color[] _colors = { Color.Red, Color.Blue, Color.Green };
    const int ID_COLOR_PICKER = 1;

    class ColorListAdapter : ArrayAdapter<Color>
    {
        public ColorListAdapter(Context context, Color[] colors) 
            : base(context, Android.Resource.Layout.SimpleListItem1, colors)
        {
        }

        public override View GetView(int position, View convertView, ViewGroup parent)
        {
            var view = base.GetView(position, convertView, parent) as TextView;
            view.SetBackgroundColor(new Color(GetItem(position)));
            view.SetTextColor(Color.Transparent); // 色値が表示されないように隠す
            return view;
        }
    }

    protected override void OnCreate(Bundle bundle)
    {
        base.OnCreate(bundle);
        SetContentView(Resource.Layout.Main);

        FindViewById<Button>(Resource.Id.myButton)
            .Click += (s, e) => ShowDialog(ID_COLOR_PICKER);
    }

    protected override Dialog OnCreateDialog(int id)
    {
        switch (id)
        {
            case ID_COLOR_PICKER:
                var adapter = new ColorListAdapter(this, _colors);

                return new AlertDialog.Builder(this)
                    .SetTitle("Select color")
                    .SetAdapter(adapter, 
                        (s, e) => Toast.MakeText(
                            this, "You picked index=" + e.Which.ToString(), 
                            ToastLength.Long).Show())
                    .Create();

            default:
                return base.OnCreateDialog(id);
        }
    }
}
```

Android API は同じなのでほとんど同じ、言語仕様の特性で少しコンパクトになりますかね。

では、また。
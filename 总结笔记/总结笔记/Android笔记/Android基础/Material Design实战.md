#Material Design
2014年Google I/O大会上推出了一套全新的界面设计思想——Material Design。

在2015年Google I/O大会上推出了一个Design Support库，这个库将Material Design中最具代表性的一些控件和效果进行了封装，使得开发者在即使不了解Material Design的情况下也能非常轻松地将自己的应用Material化。以下都为Design Supprot库中控件。

##Toolbar
传统的ActionBar由于设计的原因，被限定只能位于活动的顶部，从而不能实现一些Material Design的效果，因此官方已经不建议使用ActionBar，推荐使用ToolBar。

默认情况下，应用会显示ActionBar。可以从主题中设置不显示ActionBar（由于我们准备使用ToolBar代替ActionBar）。

- `Theme.AppCompat.NoActionBar`：深色主题，会将界面的主题颜色设置成深色，陪衬颜色设置成淡色。

-  `Theme.AppCompat.Light.NoActionBar`：淡色主题，会将界面的主题颜色设置成淡色，陪衬颜色设置成深色。

**AppTheme中的属性有：**

- `colorPrimary`：ActionBar颜色（如果有的话）。
- `colorPrimaryDark`：状态栏颜色。
- `colorAccent`：强调色，不如一些控件的选中颜色。
- `windowBackground`：背景色。可能报错（使用`android:windowBackground`即可解决。原因不知道）

**ToolBar使用：**

	<android.support.v7.widget.Toolbar
        android:id="@+id/toolBar"
        android:layout_width="match_parent"
        android:layout_height="?attr/actionBarSize"
        android:background="?attr/colorPrimary"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
        >
    </android.support.v7.widget.Toolbar>

>使用app命名空间是为了兼容老系统（5.0之前）。

并且在代码中使用：
	
	  Toolbar toolbar = (Toolbar) findViewById(R.id.toolBar);
      setSupportActionBar(toolbar);

如果需要使用菜单：
	
	<item
        android:id="@+id/settings"
        android:icon="@drawable/ic_settings"
        android:title="Settings"
        app:showAsAction="never"/>

>showAsAction主要有以下几种值可选：
>
>- always：永远显示在Toolbar中，如果屏幕空间不够则不显示。
>- ifRoom：表示屏幕空间足够的情况下显示在Toolbar中，不够的话就显示在菜单中。
>- never：表示永远显示在菜单之中。
>Toolbar中的action按钮只会显示图标，菜单中的action按钮只会显示文字。

`onCreateOptionsMenu`中进行布局的加载。

	@Override
    public boolean onCreateOptionsMenu(Menu menu) {
        getMenuInflater().inflate(R.menu.toolbar, menu);
        return true;
    }

    @Override
    public boolean onOptionsItemSelected(MenuItem item) {
        switch (item.getItemId()) {
            case R.id.backup:
                Toast.makeText(this, "You clicked Backup", Toast.LENGTH_SHORT).show();
                break;
            case R.id.delete:
                Toast.makeText(this, "You clicked Delete", Toast.LENGTH_SHORT).show();
                break;
            case R.id.settings:
                Toast.makeText(this, "You clicked Setting", Toast.LENGTH_SHORT).show();
                break;
            default:
        }
        return true;
    }

##DrawerLayout
DrawerLayout是一个布局，在布局中允许放入两个直接子控件，第一个子控件是主屏幕中显示的内容，第二个子控件是滑动菜单中显示的内容。

	<?xml version="1.0" encoding="utf-8"?>
	<android.support.v4.widget.DrawerLayout
	    xmlns:android="http://schemas.android.com/apk/res/android"
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    xmlns:tools="http://schemas.android.com/tools"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    tools:context="jct.android.study.com.mvptest.MainActivity">
	
	    <FrameLayout
	        android:layout_width="match_parent"
	        android:layout_height="match_parent">
	
	        <android.support.v7.widget.Toolbar
	            android:id="@+id/toolBar"
	            android:layout_width="match_parent"
	            android:layout_height="?attr/actionBarSize"
	            android:background="?attr/colorPrimary"
	            android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
	            app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
	            >
	        </android.support.v7.widget.Toolbar>
	
	    </FrameLayout>
	
	
	    <TextView
	        android:layout_width="match_parent"
	        android:layout_height="match_parent"
	        android:layout_gravity="start"
	        android:text="This is menu"
	        android:textSize="30sp"
	        android:background="#FFF"/>
	
	</android.support.v4.widget.DrawerLayout>

一般还会在Toolbar的最左边加入一个导航按钮。

 	if (actionBar != null) {
		设置最左侧的按钮：HomeAsUp
        actionBar.setDisplayHomeAsUpEnabled(true);
        actionBar.setHomeAsUpIndicator(R.drawable.ic_menu);
    }

它的id为android.R.id.home（永远）

	case android.R.id.home:
    	mDrawerLayout.openDrawer(GravityCompat.START);
        break;

##NavigationView
可以设置左侧布局。
##FloatingActionButton
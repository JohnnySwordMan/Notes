# ScrollView的子View的match_parent问题

### 问题
最近在项目中遇到在**ScrollView**设置子View的高度为**match_parent**不起作用。
ImageView已经将图片设置成**match_parent**，但ScrollView却将图片改成**wrap_content**，布局如下：  

```java
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <ImageView
            android:id="@+id/iv_baby_nipple"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:src="@mipmap/baby_nipple"/>

    </LinearLayout>

</ScrollView>
```

### 解决方法
添加ScrollView的属性：`android:fillViewport = "true"`

在ScrollView源码中有这样一个变量：`private boolean mFillViewport;`，如果将该变量设置成true，则ScrollView将会测量子View，使其填满当前可见区域


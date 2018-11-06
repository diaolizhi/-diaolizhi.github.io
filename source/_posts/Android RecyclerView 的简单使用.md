---
title: RecyclerView 的简单使用
date: 2018-11-04
categories: Android
---

RecyclerView 控件被推荐用来展示一组数量较多的或者数据频繁改变的数据集。

<!--more-->

# 官方文档

[Create a List with RecyclerView](https://developer.android.com/guide/topics/ui/layout/recyclerview#select)



# 添加依赖

打开 app module 中的 build.gradle，添加：

```
dependencies {
    implementation 'com.android.support:recyclerview-v7:28.0.0'
}
```



# 在布局文件中添加 RecyclerView

```xml
<?xml version="1.0" encoding="utf-8"?>
<android.support.constraint.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <android.support.v7.widget.Toolbar
        android:id="@+id/toolbar"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:background="?attr/colorPrimary"
        android:minHeight="?attr/actionBarSize"
        android:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
        app:popupTheme="@style/ThemeOverlay.AppCompat.Light"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <android.support.v7.widget.RecyclerView
        android:id="@+id/myRecyclerView"
        android:layout_width="368dp"
        android:layout_height="0dp"
        android:layout_marginTop="16dp"
        android:layout_marginBottom="16dp"
        android:scrollbars="vertical"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/toolbar" />

</android.support.constraint.ConstraintLayout>
```



# 在主界面中使用 RecyclerView

这一步主要是添加布局管理器，以及为 RecyclerView 设置数据集。

设置数据集的方法是新建一个适配器，然后将数据传给适配器，并调用 RecyclerView 的 setAdapter 方法。

```java
public class MyActivity extends Activity {
    private RecyclerView mRecyclerView;
    private RecyclerView.Adapter mAdapter;
    private RecyclerView.LayoutManager mLayoutManager;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.my_activity);
        mRecyclerView = (RecyclerView) findViewById(R.id.my_recycler_view);

        // use this setting to improve performance if you know that changes
        // in content do not change the layout size of the RecyclerView
        mRecyclerView.setHasFixedSize(true);

        // use a linear layout manager
        mLayoutManager = new LinearLayoutManager(this);
        mRecyclerView.setLayoutManager(mLayoutManager);

        // specify an adapter (see also next example)
        mAdapter = new MyAdapter(myDataset);
        mRecyclerView.setAdapter(mAdapter);
    }
    // ...
}
```



# 为列表添加一个适配器

适配器继承自 RecyclerView.Adapter，泛型指定为这个自定义适配器的内部类：MyAdapter.MyViewHolder。

用一个成员变量 myDataset 引用数据集，通过适配器的构造函数初始化。

MyViewHolder 继承自 RecyclerView.ViewHolder，在其中可以定义我们需要的控件引用。

onCreateViewHolder 这个方法被布局管理器（LaoutManager）调用，返回一个 MyViewHolder 对象。这个 MyViewHolder 的类型必须和 MyAdapter 中内部类的类型一致。

onBindViewHolder 将视图与对象位置的数据进行绑定。

R.layout.my_textview 是自定义布局，新建一个布局文件名字叫做：my_textview.xml，根据需要添加控件。

```java
public class MyAdapter extends RecyclerView.Adapter<MyAdapter.MyViewHolder>{

    private String[] myDataset;

    public MyAdapter(String[] dataset) {
        myDataset = dataset;
    }

    @Override
    public MyViewHolder onCreateViewHolder(ViewGroup parent, int i) {
        TextView v = (TextView) LayoutInflater.from(parent.getContext())
                .inflate(R.layout.my_textview, parent, false);
        MyViewHolder vh = new MyViewHolder(v);
        return vh;
    }

    @Override
    public void onBindViewHolder(MyViewHolder myViewHolder, int i) {
        myViewHolder.myTextView.setText(myDataset[i]);
    }

    @Override
    public int getItemCount() {
        return myDataset.length;
    }

    public static class MyViewHolder extends RecyclerView.ViewHolder {
        public TextView myTextView;
        public MyViewHolder(TextView v) {
            super(v);
            myTextView = v;
        }
    }
}

```

```xml
<?xml version="1.0" encoding="utf-8"?>
<TextView xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/text_view"
    android:layout_width="match_parent"
    android:layout_height="80px" />
```



# 更新列表

直接修改数据集，然后调用适配器的 notifyDataSetChanged() 方法。

```java
myData[0] = "上传";
myAdapter.notifyDataSetChanged();
```



# 列表中元素的点击事件

参考的是：[[RecyclerView onClick](https://stackoverflow.com/questions/24471109/recyclerview-onclick)]

关键代码：

```java
private final OnClickListener mOnClickListener = new MyOnClickListener();

@Override
public MyViewHolder onCreateViewHolder(final ViewGroup parent, final int viewType) {
    View view = LayoutInflater.from(mContext).inflate(R.layout.myview, parent, false);
    view.setOnClickListener(mOnClickListener);
    return new MyViewHolder(view);
}
```

以及：

```java
@Override
public void onClick(final View view) {
    int itemPosition = mRecyclerView.getChildLayoutPosition(view);
    String item = mList.get(itemPosition);
    Toast.makeText(mContext, item, Toast.LENGTH_LONG).show();
}
```

这两个方法是写在自定义适配器类中的，所以 MyAdapter 需要实现 View.OnClickListener 接口。

而在 onClick 方法中需要 RecyclerView 的引用，以及上下文对象，所以适配器的构造方法如下：

```java
public MyAdapter(String[] dataset, RecyclerView myRecyclerView, Context mcontext) {
    this.myRecyclerView = myRecyclerView;
    myDataset = dataset;
    this.mcontext = mcontext;
}
```

在 MainActivity 中这样创建适配器：

```java
myAdapter = new MyAdapter(myData, myRecyclerView, this);
```


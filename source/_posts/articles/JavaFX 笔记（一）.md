---
title: JavaFx 笔记
date: 2018-05-28
categories: JavaSE
---

一篇关于 JavaFX 的笔记。
<!--more-->

金旭亮老师在 live 中提到，要自己动手写一个 Windows 下的文件资源管理器。

刚开始动手不久，我就知道这是不可能的了，因为很多控件，我不知道该怎么描述，更别提去谷歌搜索相关的内容了。

后来遇到很多问题，都是在网上搜索，效率可以说非常低。简单的问题想一个晚上，困难一点的问题一个晚上都想不出来，说到底还是自己英语不行。

不过实现功能那一刻还是开心的，这个*文件浏览器*（因为根本没有管理功能）就到此为止。为了让自己下次，遇到同样的问题不用去搜索，我决定把一些东西记录下来。

![](http://wx2.sinaimg.cn/large/0065Ozb6gy1frs5xo4lxhj30k30egmxt.jpg)

# TreeView 创建问题

先说说两个类：

- TreeView

这个控件包含一个根元素（root），这个根元素又可以包含多个元素， 以此类推， 就构成了与树类似的视图。在这里使用它，是因为文件目录也具有类似的特点：C 盘下有多个目录，点开某个目录，这些目录下可能也有多个目录...

- TreeItem<T\> 

上面这个类是树形视图，而这个类就是树形视图中的一个元素。这个元素下面还可以添加多个元素，这样点击它就会展开，就可以看到它的子元素。

这个类是泛型类，根据实际情况决定实际类型，比如字符串、数字、或者文件类型。

当时看到的教程都是创建 TreeItem 之后，再创建 TreeView。而我这里是通过 FXML 文件设计 TreeView，然后在控制类里面创建 TreeItem，结果我就不会写了，又不能在 FXML 中添加 TreeItem。

解决方法很简单，简单到我觉得自己是个睿智，这里忍不住加了一条注释。

*FileTreeItem 是自定义类，原因马上会说。*

```java
Path path = Paths.get("C:\\");
TreeItem<FileTreeItem> root = new TreeItem<>(new FileTreeItem(path));

//设置图标
root.setGraphic(new ImageView(getClass().getResource("../image/folder.png")
		.toExternalForm()));

treeView.setRoot(root);
//找了半天，真是日了狗
```



# 为什么要自定义 FileTreeItem 类

因为如果 TreeItem 的泛型是 File，那么是 TreeView 中显示的就不是文件名，而是更长的路径名。

出现这个问题的原因是：File 类的 toString() 方法最终返回的是对象的 path 属性。而我们想要的应该是文件名，也就是要 getName() 方法返回的值。

*但是这个解决方法不好，应该写一个类继承 File，然后重写 toString 方法。*

```java
// 这个类的目的是，让 TreeItem 里面保存 File，而且显示的时候只显示文件名，而不是一长串
class FileTreeItem {
	
	private File myFile;
	
	public FileTreeItem(Path path) {
		myFile = path.toFile();
	}
	
	public FileTreeItem(File file) {
		myFile = file;
	}
	
	@Override
	public String toString() {
		return myFile.getName();
	}
	
	public void setFile(Path path) {
		myFile = path.toFile();
	}
	
	public void setFile(File file) {
		myFile = file;
	}
	
	public File getFile() {
		return myFile;
	}
}
```

# 多线程问题

既然是文件浏览器，肯定要递归地遍历每个文件夹。如果只是遍历当前工作目录肯定没问题，问题是如果要遍历的是整个 C 盘呢？这显然要花很多时间，总不能让程序一直停在那里吧。

自然而然想到多线程，同时也自然而然想到子线程不能更新用户界面。

这里使用的多线程并不复杂。

```java
Runnable task = () -> {
    createTree(path, root);
};

new Thread(task).start();
```

如何更新用户界面才是个问题。

金老师的教育网站提到了解决方式，但我不清楚自己是否理解正确。

我的理解是，子线程不能更新用户界面，如果要在子线程更新界面，就应该把**更新界面这件事**交给专门处理用户界面的线程，这个线程什么时候有空了，就会执行这个更新操作。

```java
public void createTree(Path path, TreeItem<FileTreeItem> treeItem) {
    try {
        //			遍历文件夹的方式
        DirectoryStream<Path> children = Files.newDirectoryStream(path);
        for(Path child: children) {
            File file = child.toFile();
            if(file.isDirectory()) {
                TreeItem<FileTreeItem> newTreeItem 	= new TreeItem<>(new FileTreeItem(file));
                System.out.println(getClass().getSimpleName());
                newTreeItem.setGraphic(new ImageView(getClass().getResource("../image/folder.png")
                                                     .toExternalForm()));

                Platform.runLater(() -> {
                    treeItem.getChildren().add(newTreeItem);
                });
            }
            //				if(file.isDirectory()) {
            //					createTree(child, newTreeItem);
            //				}

            System.out.println(child);
        }
    } catch (IOException e) {
        System.out.println("访问出错");
    }
}
```

 重点就在于

```java
Platform.runLater(() -> {
    treeItem.getChildren().add(newTreeItem);
});
```

(这里注释掉那几句，是因为后来我不想一开始就遍历所有文件夹，而是等用户点击时才去访问。)



# TableView 中列的问题

## TableColumn 问题

暂且把 TableColumn 看作是表头，第一个泛型类型是 TableView 中某一行元素的类型，第二个泛型类型是单元格中的类型。

```java
TableColumn<MyFile, String> column2 = new TableColumn<>("文件名");
column2.setCellValueFactory(new PropertyValueFactory<>("Name"));
...
table.getColumns().add(column2);
```

这里写的是："文件名"和 "Name"，一开始不明白程序实际上是调用 getXXX() 方法来获取单元格的内容的，所以我两个都写了"文件名"，结果表格一直为空...

这里写 "Name" 就会调用 MyFile 类的 getName() 方法，以此类推。

## TableView 添加元素方法

创建 TableColumn 之后，然后添加到 TableView 之中。

要想表格有内容，还得给 TableView 添加要显示的列表。

```java
ObservableList<MyFile> list = FXCollections.observableArrayList();

try {
    DirectoryStream<Path> children = Files.newDirectoryStream(myPath);
    for(Path child :  children) {
        list.add(new MyFile(child.toString()));
    }
} catch (IOException e) {
    e.printStackTrace();
}
...
table.setItems(list);
```

## 获取文件其他属性的方法

File 的 getName() 方法可以获取文件名，但是没办法获取到文件的大小（尽管有 length() 方法，但是这个方法不是 get 开头的。）

而 MyFile 是一个自定义类，并且继承自 File 类。它的目的是**添加一些方法：比如 getSize() 等**，在 getSize() 方法内调用 length() 方法，这样就可以获取到文件的长度了。

## TableView 高度问题

TableView 表格是有背景的，如果表格元素不够多，不够铺满父级容器，看起来就不顺眼，所以要设置它的大小。

```java
table.prefHeightProperty().bind(vbox.heightProperty());
```

上面这句话是可行的，但我搞不懂它的含义是什么。

另外，这个的 TableView 是在 VBox 里面的。当时开始写的时候，好像还一直不能显示出表格，至于原因就不记得了。总之，放在 VBox 里是可行的。

## TableView 更新问题

这个 TableView 是显示一个文件夹里面的内容，如果点击一个文件夹 A，那么显示的就应该是 A 的内容。

一开始我是使用：先调用

```java
vbox.getChildren().clear();
```

把已存在的表格删除掉，然后再创建一个表格的方法。

后来感觉这样不太好，就添加一个用于更新表格的方法，这个方法里生成一个新的列表，然后调用 table 的 setItems() 方法。

```java
public void updateTable(Path myPath) {
    nowPath = myPath;
    ObservableList<MyFile> list = FXCollections.observableArrayList();
    try {
        DirectoryStream<Path> children = Files.newDirectoryStream(myPath);
        for(Path child :  children) {
            list.add(new MyFile(child.toString()));
        }
    } catch (IOException e) {
        e.printStackTrace();
    }

    table.setItems(list);
}
```

由此可知，改变一个表格，只需要改变它的 items。

# 添加事件响应问题

这个问题是困扰我最久的一个问题了。如果是一个按钮还好办，问题是 TreeItem 没有 setOnMouseClicked() 方法。后面尝试的一些方法也不好用，最后在 GitHub 上找了一份 JavaFX 文件管理器的代码，终于看到了解决方法。

```java
//		设置树视图的点击事件
treeView.setOnMouseClicked(e -> {
    TreeItem<FileTreeItem> item = treeView.getSelectionModel().getSelectedItem();
    if(e.getClickCount() == 2 && item != null) {
//		获取到被点击的 Item
        File file = item.getValue().getFile();
        String path2 = file.getPath();
        System.out.println(path2);
        updateTable(Paths.get(path2));

        if(file.isDirectory()) {
            createTree(file.toPath(), item);
        }
        item.setExpanded(true);

    }
});
```

关键就在于

```java
getSelectionModel().getSelectedItem()
```

获取到被点击的元素。

获取到被点击的元素之后事情就变得简单了，这里只是更新表格和更新树视图。

# 总结

我所遇到的所有问题基本都在这里了，这份代码还存在不少问题，有时候我都不清楚到底是传一个 Path 对象还是要给 String 对象...

## 不足之处

- 没有设置好图标（其实是不会）
- 没有实现右键单击弹出选项的功能（还是不会）
- 没有实现文件资源管理器的布局切换（更加不会）
- 没有对文件排序，使文件夹排在前面（菜是原罪）
- 对很多类和方法一知半解
- ......


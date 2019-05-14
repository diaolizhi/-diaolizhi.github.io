---
title: Bootstrap4 Dropdown 组件扩展 hover 事件
date: 2019-5-11
categories: 前端
---



![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/19051201.gif)

<!--more-->

效果如图所示，很简单的一个功能，但是因为我在这上面花了很多时间，所以我决定把它记录下来。



# 官方文档

[Dropdowns · Bootstrap ](<https://getbootstrap.com/docs/4.0/components/dropdowns/> )



# 示例 

Bootstrap 提供的下拉菜单默认是通过鼠标点击触发，下面是一个简单的完整示例。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/2019051201.gif)

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>


    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm"
        crossorigin="anonymous">
</head>

<body>
    <div class="container">
        <!-- Default dropright button -->
        <div class="btn-group dropright">
            <button type="button" class="btn btn-secondary dropdown-toggle" data-toggle="dropdown" aria-haspopup="true"
                aria-expanded="false">
                Dropright
            </button>

            <div class="dropdown-menu">
                <a class="dropdown-item" href="#">Action</a>
                <a class="dropdown-item" href="#">Another action</a>
                <a class="dropdown-item" href="#">Something else here</a>
                <div class="dropdown-divider"></div>
                <a class="dropdown-item" href="#">Separated link</a>
            </div>
        </div>
    </div>


    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN"
        crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q"
        crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl"
        crossorigin="anonymous"></script>
</body>

</html>
```



# 扩展 hover 事件

有时候希望鼠标放上去就展开菜单，而不需要鼠标点击，只需要一点简单的代码就可以实现：

```html
<style>
    .dropdown-menu {
        margin:0 !important;
    }
    .dropright:hover > .dropdown-menu {
        display:block;
    }
</style>
```

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/19051202.gif)

为什么加上这几句就行了呢？因为 `dropdown-menu` 类的原来的样式是 `display:none;`，所以设置 `display:block;` 之后 `dropdown-menu` 就会显示出来了。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2019-05-12_12-07-15.png)



# 问题一

当鼠标放上去时没有问题，但是如果鼠标点击之后就会**出现问题**。首先是位置，下拉菜单的位置改变了，其次是鼠标移走之后下拉菜单也不会自动关闭。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/19051203.gif)

# 问题一的解决方法

删掉 `button` 标签的  `data-toggle="dropdown" `，这个方法看起来一点也不优雅，但是目前只能想到这个。



# 问题二

本来下拉菜单是在右侧显示的，但是现在却是在下面显示。



# 问题二的解决方法

在 `<style>` 标签中添加：

```css
.dropright {
    position: relative;
}
.dropdown-menu {
    position: absolute;
    left: 100%;
    top: 0;
}
```



# 总结

一开始我就搜到了解决办法，但是由于我太蠢了，看都不看直接复制，导致一直得不到想要的效果（根本原因是：别人的 HTML 代码跟官方示例不一样，所以直接复制过来自然没效果）。

后来找到了另一种方法，然后那个方法也有很大的缺陷：鼠标移走时下拉菜单不会关闭。尽管最后也解决了这个问题，但是效果并不好，而且有时还有 bug。

直到昨晚一点多终于搞明白了，一方面觉得自己真是蠢，抄代码都能抄歪来，另一方面没人分享自己的喜悦，所以有了这篇文章。

欢迎批评指点，如果对你有帮助那就更好了。



# 完整代码

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>


    <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm"
        crossorigin="anonymous">
</head>

<body>


    <style>
        .dropdown-menu {
            margin:0 !important;
        }
        .dropright:hover > .dropdown-menu {
            display:block;
        }
        .dropright {
            position: relative;
        }
        .dropdown-menu {
            position: absolute;
            left: 100%;
            top: 0;
        }
    </style>

    <div class="container">
        <!-- Default dropright button -->
        <div class="btn-group dropright">
            <button type="button" class="btn btn-secondary dropdown-toggle" aria-haspopup="true" aria-expanded="false">
                Dropright
            </button>

            <div class="dropdown-menu">
                <a class="dropdown-item" href="#">Action</a>
                <a class="dropdown-item" href="#">Another action</a>
                <a class="dropdown-item" href="#">Something else here</a>
                <div class="dropdown-divider"></div>
                <a class="dropdown-item" href="#">Separated link</a>
            </div>
        </div>
        <br>
        <div class="btn-group dropright">
            <button type="button" class="btn btn-secondary dropdown-toggle" aria-haspopup="true" aria-expanded="false">
                Dropright
            </button>

            <div class="dropdown-menu">
                <a class="dropdown-item" href="#">Action</a>
                <a class="dropdown-item" href="#">Another action</a>
                <a class="dropdown-item" href="#">Something else here</a>
                <div class="dropdown-divider"></div>
                <a class="dropdown-item" href="#">Separated link</a>
            </div>
        </div>

        <br>
        <div class="btn-group dropright">
            <button type="button" class="btn btn-secondary dropdown-toggle" aria-haspopup="true" aria-expanded="false">
                Dropright
            </button>

            <div class="dropdown-menu">
                <a class="dropdown-item" href="#">Action</a>
                <a class="dropdown-item" href="#">Another action</a>
                <a class="dropdown-item" href="#">Something else here</a>
                <div class="dropdown-divider"></div>
                <a class="dropdown-item" href="#">Separated link</a>
            </div>
        </div>
    </div>


    <script src="https://code.jquery.com/jquery-3.2.1.slim.min.js" integrity="sha384-KJ3o2DKtIkvYIK3UENzmM7KCkRr/rE9/Qpg6aAZGJwFDMVNA/GpGFF93hXpG5KkN"
        crossorigin="anonymous"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js" integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q"
        crossorigin="anonymous"></script>
    <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js" integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl"
        crossorigin="anonymous"></script>

</body>

</html>
```






---
title: 通过表单上传 Blob 对象
date: 2018-12-17
categories: Javascript
---

获取到 Blob 对象之后，如何通过表单提交？

<!--more-->



# 目的

在 Javascript 中获取到了 Blob 对象，希望将 Blob 对象传递给后端。



# 条件

在 HTML 中有一个 form 表单，form 中有一个可以上传文件的 input 输入框。

```html
<input id="recordInput" type="file" name="file">
```

但是这个输入框只能点击 -> 选择文件，那么如何将 Blob 与输入框联系起来呢？



# 解决

可以直接看这个回答：

[Send Blob object with POST Form](https://stackoverflow.com/questions/50157450/send-blob-object-with-post-form)



首先需要这个方法，它将 File 对象转换成一个 FileList 对象：

```javascript
function createFileList(a) {
    a = [].slice.call(Array.isArray(a) ? a : arguments);
    for (var c, b = c = a.length, d = !0; b-- && d;) d = a[b] instanceof File;
    if (!d) throw new TypeError('expected argument to FileList is File or array of File objects');
    for (b = (new ClipboardEvent('')).clipboardData || new DataTransfer; c--;) b.items.add(a[c]);
    return b.files;
};	
```

之所以要转换，是因为 input 元素中有一个属性：files，它的类型就是 FileList，必须要转换才可以进行赋值。*（注意：并没有 file 这个属性。）*



第二步，设置 input 的 files 属性（使用 jQuery 查找元素）：

```javascript
var recInput = $("#recordInput")[0];
recInput.files = createFileList(new File([blob], "my-record.mp3", {type: "audio/mp3"}));
```



这样一来，不用点击选择文件，input 框就已经选中文件了。

![](https://md-img-1252869657.cos.ap-shanghai.myqcloud.com/hexo/Snipaste_2018-12-17_22-25-20.png)

后端简略代码：

```java
@RestController
public class IndexController {


    private final String path = "D:\\del\\";

    @PostMapping("/upload")
    public Object upload(@RequestParam(name = "file") MultipartFile file) {
        System.out.println(file);

        File file1 = new File(path + "aa.mp3");
        try {
            file.transferTo(file1);
        } catch (IOException e) {
            e.printStackTrace();
        }

        return file.getOriginalFilename();
    }

}
```



# 总结

- Blob、File、FileList  各自是一种类型，而且 File 是 Blob 的子类。
- 查找元素时即使通过 ID 查找，也要使用下标 [0] 选择元素


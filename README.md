# 如何使用jquery.fileupload.js上传文件

我是在这里找到的这款文件上传插件：http://www.jq22.com/jquery-info230

上面已经有了一些教程，这里再复制一遍。

`jQuery File Upload 是一个Jquery图片上传组件，支持多文件上传、取消、删除，上传前缩略图预览、列表显示图片大小，支持上传进度条显示；支持各种动态语言开发的服务器端。`

`jQuery File Upload有多个文件选择，拖放上传控件拖放支持，进度条，验证和预览图像，音频和视频 。`

`支持跨域，分块和可恢复的文件上传和客户端图像大小调整。适用于任何服务器端平台（PHP, Python, Ruby on Rails, Java, Node.js, Go etc.） ，支持标准的HTML表单文件上传。`

### `需要注意的是默认的jquery.fileupload.js不支持文件名为中文的文件上传，修改看下面`

## 使用方法：

```html
<!DOCTYPE HTML>
<html>
<head>
<meta charset="utf-8">
<title>jQuery File Upload Example</title>
<style>
/*进度条样式*/
.bar {
    height: 18px;
    background: green;
}
</style>
</head>
<body>
<!-- data-url表示服务器地址，不一定要在同一个域名下 -->
<input id="fileupload" type="file" name="files[]" data-url="receive.php" multiple="multiple">
<div id="progress">
    <div class="bar" style="width: 0%;"></div>
</div>
<div>
    <p class="uploaded-files">已上传文件：</p>
</div>
<form  action="server.php" method="post" name="myForm" >
    <div class="files-wrap"></div>
    <input type="submit" value="确定"  />
</form>
<script src="//libs.baidu.com/jquery/1.8.3/jquery.js"></script>
<script src="js/vendor/jquery.ui.widget.js"></script>
<script src="js/jquery.iframe-transport.js"></script>
<script src="js/jquery.fileupload.js"></script>
<script>
$(function () {
    $('#fileupload').fileupload({
        dataType: 'json',
        // 进度条设置
        progressall: function (e, data) {
            var progress = parseInt(data.loaded / data.total * 100, 10);
            $('#progress .bar').css(
                'width',
                progress + '%'
            );
        },
        done: function (e, data) {
            $.each(data.result.files, function (index, file) {
                var newEle = '<input type="hidden" name="file_url[]" value="'+file.url+'">'+
                '<input type="hidden" name="file_name[]" value="'+file.name+'">';
                $(".files-wrap").html($(".files-wrap").html() + newEle);  // 将已上传的文件url和文件名用hidden类型输入框保存并提交至数据库
                $(".uploaded-files").append(file.name+'<br>');  //在页面上显示已上传的文件名
                $('#progress .bar').css("width", "0%");  //重置进度条
            });
        }
    });
});
</script>
</body> 
</html>
```

上例是自动上传文件的，如果不想自动上传，可以这样：

```javascript
$(function () {
    $('#fileupload').fileupload({
        dataType: 'json',
        add: function (e, data) {
            // 通过在add方法中添加一个按钮，实现点击按钮时提交文件数据(submit)
            // add方法在上传数据之前，你可以在这里做很多其他的逻辑
            data.context = $('<button/>').text('Upload')
                .appendTo(document.body)
                .click(function () {
                    $(this).replaceWith($('<p/>').text('Uploading...'));
                    data.submit();
                });
        },
        done: function (e, data) {
            data.context.text('Upload finished.');
        }
    });
});
```

## 文件名中有中文的文件上传方法

默认的jquery.fileupload.js插件可能是作者在制作时没有考虑到这个问题，所以如果上传的是文件名含有中文的文件，是上传不了的，修改如下：

【参考：http://blog.csdn.net/zhouyingge1104/article/details/38322403】

编辑jquery.fileupload.js，找到第477行
```javascript
  if (options.blob) {
      formData.append(paramName, options.blob, file.name);
  } else {
      $.each(options.files, function (index, file) {
          // This check allows the tests to run with
          // dummy objects:
          if (that._isInstanceOf('File', file) ||
                  that._isInstanceOf('Blob', file)) {
              formData.append(
                  ($.type(options.paramName) === 'array' &&
                      options.paramName[index]) || paramName,
                  file,
                  file.uploadName || file.name         //就是这一行
              );
          }
      });
  }
```

将其修改为
```javascript
encodeURI(file.uploadName || file.name)
```

此时再次上传含有中文的文件，你会发现，文件可以上传了，只是文件名变成了一串编码过的字符。如果是用java的，可以参考上面给的参考链接修改，php的我改了大半天也没改成，于是决定不使用这个插件自带的php，自己写一个接收文件的php，再将其文件保存时把文件名urldecode一下就可以了。

```php
    $up_files = $_FILES['files'];
    $result = array();
    for($i=0; $i < count($up_files['name']); $i++){
        if (empty($up_files['error'][$i]) || (!isset($up_files['error'][$i]) && isset($up_files['tmp_name'][$i]) && $up_files['tmp_name'][$i] != 'none'))
        {
            // 检查文件格式
            if (!check_file_type($up_files['tmp_name'][$i], $up_files['name'][$i], $allow_file_types))
            {
                continue;
            }

            // 复制文件
            $res = upload_article_file($up_files, $i);
            if ($res != false)
            {
                $result[] = array(
                    "name" => urldecode($up_files['name'][$i]),
                    "url"  => $res
                );
            }
        }
    }
    echo json_encode($result);die;
```

## 关于php上传文件的大小限制问题

如果你在修改了php.ini 文件中 upload_max_filesize（上载文件的最大许可大小） 这个参数之后仍然发现只能上传2M以内的文件，那么就需要再次修改：post_max_size（表单提交最大数据），将其设置成一个较大的值

如果还是不行，修改：max_execution_time（脚本运行时间）和memory_limit（脚本运行最大消耗的内存）

---
title: 随笔-flask处理上传问题
name: sb5
date: 2019-08-26
id: 0
tags: [暂时没有标签]
categories: []
abstract: ""
---


印象深刻，进一步加深了对js的使用

<!--more-->

## Flask上传图片

### 使用flask_uploads插件

初始化app

```python
from flask_uploads import UploadSet, configure_uploads, IMAGES
app = Flask(__name__,static_url_path='')
photos = UploadSet('photos', IMAGES)
configure_uploads(app, photos)
```

### wtf表单

使用flask_wtf来渲染表单

```python
from flask_wtf import FlaskForm
from wtforms import StringField, SubmitField, FileField, TextAreaField
from flask_wtf.file import FileField, FileAllowed, FileRequired

class NewAlbumForm(FlaskForm):
    title = StringField(u'标题')
    about = TextAreaField(u'介绍', render_kw={'rows': 8})
    photo = FileField(u'图片', validators=[
        FileRequired(u'你还没有选择图片！'),
        FileAllowed(photos, u'只能上传图片！')])
    submit = SubmitField(u'上传')
```

### html样式

```html
 <form class="form form-horizontal" method="POST" enctype="multipart/form-data">
    {{ form.hidden_tag() }}
    {{ form.title }}
     <a class="file">{{ form.photo(multiple="multiple")}}点击上传</a>
    {{ form.submit }}
</form>
```

因为是多文件上传，所以传入参数`multiple="multiple"`

### 多个文件request表单处理

```python
form = NewAlbumForm()
if form.validate_on_submit():
   if request.method == 'POST' or 'photo' in request.files.getlist("photo"):
		images = save_image(request.files.getlist("photo"),photos=photos)
```

**注意**因为是多个文件，表单的文件是一个列表对象，使用`getlist`获取

常见错误:`TypeError("storage must be a werkzeug.FileStorage")`返回的一定要是文件对象

### 缩略图处理

可以使用PIL库生成缩略图

```python
from PIL import Image
img = Image.open(photos.path(image))
img = img.resize((base_width, h_size))
```

常见问题:图片格式错误会导致压缩失败，`IOError: image file is truncated (0 bytes not processed)`
解决:添加以下

```python
from PIL import ImageFile
ImageFile.LOAD_TRUNCATED_IMAGES = True
```

常见问题:图片颜色模式不是标准RGB，`cannot write mode P as JPEG`
添加以下判断

```python
if img.mode == "P":
	img = img.convert('RGB')
```

### 图片删除

使用flask_uploads自带的`path`方法获取图片地址

```python
photos = UploadSet('photos', IMAGES)
os.remove(photos.path(filename))
```


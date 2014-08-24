Visual studio及其插件
===
在WINDOWS下面开发C/C++程序，可以使用的IDE还是有那么一些，但是都用了之后，会发现还是只有MS VS用起来是比较舒服的。
VS有些让人不爽的地方，比如说其要捆绑安装很多不想要的东西，C#，F#,Basic之类的。但是在比较好的电脑上面，安装这写东西实际上也不会有什么问题。

还有就是VS要钱，但是网上有很多的KEY,如果经济能力允许的时候还是可以买一个号。

#### Vs可以同时安装多个版本。
以前安装vs的时候，总是喜欢在电脑上面只有一个版本，实际上，可以在电脑上面同时安装几个版本的。

### Vassistx插件
使用VS，那么vassistx插件就是一个很好使用的东西了，其各种代码辅助功能用起来相当的舒服。建议将所有的选项都打开。

vx的安装网上有很多的教程，破解的时候就是替换一个DLL文件就是了。

#### 添加文件后缀名让vx支持
Vx默认只能认识一些文件后缀名，比如对于C/C++，它只认识c,cpp,cc,cxx，如果我们用的一个东西其是C/C++的一个变种（比如CUDA编程的时候，用的就是.cu和.cuh的文件）,其文件后缀名不是上面的这些，那么我们可以通过修改注册表的方式使vx将这种特殊的文件认为是c/c++文件，在注册表中的
```
HKEY_CURRENT_USER\Software\Whole Tomato\Visual Assist X\VANet11\ExtSource
```
注意上面的vanet11表示的是vs 2012,extsource是一个字符串，在其后面加上.cu就可以了。

```
HKEY_CURRENT_USER\Software\Whole Tomato\Visual Assist X\VANet11\ExtHeader
```

同样的，在上面加上.cuh那么这种文件就会被认为是header文件了。

这样语法高亮就会有效了。

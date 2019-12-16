---
categories:
- 随笔
tags:
- visual code
---



### 两种模式：

#### 第一种：
快捷键：`alt+shift `，这种模式下快捷键，按住鼠标左键拖动，便可以选择多行中某一列同时编辑，此时只能选择多行的同一列，某一行较短的话此行光标将会在行末
![多行编辑1.png](https://upload-images.jianshu.io/upload_images/20062114-cfbc8bcba81ab205.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
**注意**：未用鼠标拖动选中的行没有光标显示，不会进行编辑
![多行编辑2.png](https://upload-images.jianshu.io/upload_images/20062114-1a6f73d7b297003b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 第二种：
快捷键：`ctrl+shift`，`alt+shift`貌似也可以，在此模式下按住快捷键同时按住鼠标左键拖动选中多行中的某一列，行长问题同上，不同的是：再次按住`ctrl`，同时鼠标左键单击未被选中的行、选中行中未被选中的部分时，可添加多个光标（编辑点）；当按住`ctrl`，再次单击已存在的光标时，可取消改光标
![多行编辑3](https://upload-images.jianshu.io/upload_images/20062114-989c8b4e18d0adeb.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 模式选择：
方法一：按住`crtl+shift+p`，在弹出的快捷命令窗口中输入`cursor`，点击`Toggle Multi-Cursor Modifier`，即可改变模式
![多行编辑4.jpg](https://upload-images.jianshu.io/upload_images/20062114-1ac851c2551de641.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

方法2：按住`crtl+,`，在弹出的setting窗口搜索栏中输入`cursor`，下拉找到下图所示设置选项,在下拉框框中选择即可，默认为`alt`，即第一种模式
![多行编辑5.jpg](https://upload-images.jianshu.io/upload_images/20062114-b55f438c938063ed.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

方法三：直接编辑setting.json文件，添加`"editor.multiCursorModifier": "ctrlCmd"`，即可更改为第二种模式，不需要更改删掉就行，vscode有默认的defaultsetting.json文件，其中默认定义为第一种模式


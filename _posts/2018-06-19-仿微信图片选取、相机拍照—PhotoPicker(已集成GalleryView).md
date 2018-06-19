---
layout:     post
title:      仿微信图片选取、相机拍照—PhotoPicker(已集成GalleryView)
subtitle:   相册 相机选取图片
date:       2018-06-19
author:     Cedar7
header-img: img/post-bg-debug.png
catalog: true
tags:
    - 图片选取
---
## Background  
最近在开发中遇到图片选取的需求，要求从相册和相机拍照选取图片，这可以说是非常常规的需求了，在我等待UE出图的过程中，产品走到我身边，默默的打开了微信，露出蜜汁微笑。“这次咱们就不出图了，就按照微信这个来就行了，有好的、成熟的产品就要借鉴嘛”，然后潇洒离去...  
  
我还能说什么，我也打开了微信，平时在使用的过程中，确实没有特别在意微信的这个功能，但当自己要开发的时候，就发现，微信之所以有这么大的用户量，对于细节的追求确实特别细腻，只是我们在平时使用过程中并不会在意，因为它的设计都合理且完美。话不多说先上图。  
![image](https://github.com/cedear/Cedear.github.io/blob/master/%E5%9B%BE%E7%89%87%E6%96%87%E4%BB%B6%E5%A4%B9/xiangceliulan.gif)  
![image](https://github.com/cedear/Cedear.github.io/blob/master/%E5%9B%BE%E7%89%87%E6%96%87%E4%BB%B6%E5%A4%B9/xiangjipaizhao.gif)  
## Tips   
其实在准备做任何东西之前，我都会先思考一下整体需要哪方面的技术点，然后再考虑各个技术的可行性。再做这个东西之前，我也是仔仔细细、认认真真的观察了微信的图片选取的整个流程，并且给它的一些细节之处也实现了。可以说，在整体上有90%与原型相似吧。至于抄袭人家产品...嗯..."有好的、成熟的产品就要借鉴嘛"。
### 1 本地图片的获取  
首先我们要知道，手机上的图片都是存储在数据库里的，Android系统对外提供了查询方法，供我们去查询存储在数据库中的图片  

```
public static final Cursor query(ContentResolver cr, Uri uri, String[] projection, String selection, String[] selectionArgs, String orderBy) {
}
```  
然后我们操作cursor就可以取出我们想要的图片了，存储在我们自定义的model里供使用。  
### 2 从底部弹出的目录选择view实现  
其实最开始是想利用BottomSheetDialog这个Material design控件来实现了，但是最后发现并不能完美实现现微信这个效果，并且在自己使用的过程中，和通过一些blog的博主介绍，发现这个控件还是有一些问题的，所以，为了节约开发成本，控制风险。我将这个view封装成了一个单独的控件，最后通过动画来实现弹出、回缩的效果。（以后有时间，要用新控件再试一下）  
### 3 全屏预览（清除状态栏）  
因为完全仿照微信来做，所以这个效果也是必须要有的。当单击图片时，会将状态栏和缩略图预览这些东西都隐藏。这里主要放一下关于状态栏相关的东西。  
（1）首先将预览的Activity的theme设置成NoActionBar的。  
（2）在oncreate（）方法中设置全屏。 应在setContentView()方法之前。 

```
getWindow().getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN | View.SYSTEM_UI_FLAG_LAYOUT_STABLE);
```
单击时展示、隐藏状态栏的方法  

```
private void hideStatusBar(Window window){
    window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_FULLSCREEN |View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN);
}
```  

```
private void showStatusBar(Window window){
    window.getDecorView().setSystemUiVisibility(View.SYSTEM_UI_FLAG_LAYOUT_FULLSCREEN;
}
```  
### 4 android 7.0 + 相机拍照FileUriExposedException  
因为7.0以上的系统不能直接拿到文件的地址了，要通过Uri来转换。出现这个问题也非常容易解决，自定义一个Provider去获取文件的Uri，这个解决方案一找一大堆，我就不多说了，就是提醒一下。  
## How to use  
（1）相册选取图片  
                
```
                PhotoPicker
                        .getPhotoPicker()
                        .setPhotoSpanCount(4)
                        .setMaxPhotoCounts(9)
                        .setGetPhotoPickerCallBack(new PhotoPicker.OnGetPhotoPickerCallBack() {
                            @Override
                            public void onGetPhotoPickerSuccess(List<PhotoInfo> photoList) {
                            
                            }

                            @Override
                            public void onGetPhotoPickerFail() {

                            }
                        })
                        .startSelectPhoto(MainActivity.this);  
```    
（2）相机拍摄图片  

```
                PhotoPicker
                        .getPhotoPicker()
                        .setGetPhotoPickerCallBack(new PhotoPicker.OnGetPhotoPickerCallBack() {
                            @Override
                            public void onGetPhotoPickerSuccess(List<PhotoInfo> photoList) {

                            }

                            @Override
                            public void onGetPhotoPickerFail() {

                            }
                        })
                        .startTakePhoto(MainActivity.this);
```    
### 1 获取PhotoPicker实例  

```
PhotoPicker.getPhotoPicker()
```

### 2 参数设置
setMaxPhotoCounts：限制图片选取的张数（默认为9）。  
setThemeColor：    设置主题色（默认ff6c00)（按钮的颜色，勾选为图片，并不生效）  
setPhotoSpanCount：图片在集体浏览时的列数（默认4列）  
setIsPhotoPreviewWithCamera：设置是否把拍照功能放在图片预览的第一个展示，也可单独调用（默认不展示）    
setIsMutilSelectType: 设置是否为单选模式（默认多选）
setGetPhotoPickerCallBack：获取图片的回调  
### 3 启动方法  
startSelectPhoto(Context context)；可以在启动之前设置各种参数。  
startTakePhoto(Context context)；不用设置各种参数，直接开启摄像头。  
## Final  
这个PhotoPicker目前只经过我的一下简单测试，可能还会有一些问题，如果在使用过程发现问题，我也会及时修复，以后也会渐渐的丰富这个库的功能，希望能让它成为一个图片处理的一个公共库。如果有感兴趣的小伙伴，[请移步到我的gayhub](https://github.com/cedear/PhotoPicker),好心人顺便Star一下，小弟不胜感激，抱拳了，溜了~~~
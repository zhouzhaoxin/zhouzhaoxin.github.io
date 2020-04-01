---
layout: post
title: "Hybrid App 数据存储及ListView的应用"
author: zhouzhaoxin
categories: ["APP"]
image: assets/images/17.jpg
comments: false
---
AppCan是Hybrid App开发框架即混合开发框架,由官方提供底层功能使用API

HTML5和JavaScript只是作为一种解析语言，真正调用的都是Native App一样封装的底层功能

## AppCan打包
- 本地打包
IDE生成的ipa包是越狱包，只能在越狱机安装，并且不支持app上传到应用市场

- 云端打包
只需要按照AppCan的开发流程和规范开发，应用开发完后可直接将其上传到AppCan打包服务器，进行打包，平台会自动生成iOS/Android平台安装包，同事支持上传AppStore

## HTML5
- LocalStorage
LocalStorage 是window的全局属性，包括localStorage和sessionStorage,二者用法基本相同，但sessionStorage是会话级别的，窗口一旦被关闭就没了，而localStorage则一直存储在本地
```text
在AppCan中的使用
appcan.locStorage.getVal(key)               获取key保存在localStorage中对应的值
appcan.locStorage.setVal(key，Val)          要设置的键值对
appcan.locStorage.remove(key)               清除localStorage中对应的值
appcan.locStorage.keys()                    获取localStorage中，保存的所有键值
```

## AppCan中ListView的使用

列表组件是根据AppCan 布局框架对数据列表进行封装的JS对象，通过配合的样式，使开发者在界面中可以快速完成列表控件的开发。

#### 使用之前要添加依赖
```python

appcan.js
appcan.control.js
appcan.listview.js
appcan.control.css
```
## 使用方法
- 常用参数
```text
selector:                                        /*选择器*/
type:   thinLine or thickLine                    /*窄行和宽行设定*/
hasIcon:   true or false                         /*是否有图片*/
hasAngle:   true or false                        /*是否有右侧箭头*/
hasSubTitle:   true or false                     /*是否有子标题*/
hasTouchEffect:   true or false                  /*是否有点击效果*/
hasCheckbox:   true or false                     /*是否有复选按钮*/
hasRadiobox:   true or false                     /*是否有单选按钮*/
align:   "left" or "right"                       /*checkbox和radiobox居左还是居右*/
multiLine:  1 2 or 3                             /*主标题文字占用最大行数。到达行数显示不全使用…替换*/
touchClass: 'sc-bg-active' or 用户自定义         /*列表条目点击效果CSS类*/
hasControl:   true or false                      /*列表条目中是否包含switch组件。*/
hasGroup:   true or false                        /*列表条目是否以分组的形式展示。*/
```

## 示例
![appcan]({{ site.baseurl }}/assets/images/appcan1.png)
#### 定义HTML
```html
<!--定义一个listview的容器-->
<style>
    .ubt {
        border-top: 1px solid;
    }
    .ubb {
        border-bottom: 1px solid;
    }
    .bc-border {
        border-color: #BABABA;
    }
    .c-wh{
        background-color: white;
    }
    .umar-at1{
        margin-top:0.625em;
    }
    .uinn-a7{
        padding:0 0.625em;
    }
</style>
<div id="listview"  class="ubt bc-border ubb c-wh umar-at1 uinn-a7"></div>

```

#### 第一种script写法
```javascript
 var lv = appcan.listview({
            selector : "#listview", //选择器，指定body标签中id为listview的容器
            type : "thinLine",      //窄行
            hasIcon : true,         //指定是否有图标
            hasAngle : true,        //指定是否有向右侧的箭头
            hasSubTitle : true,     //指定是否有子标题
            multiLine : 1           //指定主标题文字占的最大行数
        });
        lv.set([{
            icon : 'personal_content/css/myImg/myImg1.png',         //指定图标
            title : '我的相册',                                     //指定标题文字
            subTitle : '备注文字'                                   //指定子标题文字
        }, {
            icon : 'personal_content/css/myImg/myImg2.png',
            title : '我的收藏',
            subTitle : ''
        }, {
            icon : 'personal_content/css/myImg/myImg3.png',
            title : '我的银行卡',
            subTitle : ''
        }]);
lv.on("click",function(obj,data,subObj){
        console.log(obj);                                            //列表条目DOM对象
        console.log(data);                                           //列表条目对应数据源对象
        console.log(subObj);                                         //列表条目点击时的子元素DOM对象例如图片
        appcan.window.open(data.pagename,data.pageurl,10);           //通过此方法打开对应的界面
    })

```
#### 第二种script写法
```javascript
var arrData = [{
            'tupian' : 'myWorkDOTO_content/css/myImg/myImg1.png',
            'biaoti' : '我的相册',
            'zibiaoti' : '备注文字',
        }, {
            'tupian' : 'myWorkDOTO_content/css/myImg/myImg2.png',
            'biaoti' : '我的收藏',
            'zibiaoti' : '',
        }, {
            'tupian' : 'myWorkDOTO_content/css/myImg/myImg3.png',
            'biaoti' : '我的银行卡',
            'zibiaoti' : '',
        }];
        var listData = [];
        for (var i = 0,
            len = arrData.length; i < len; i++) {
            var list = {
                title : arrData[i].biaoti,
                icon : arrData[i].tupian,
                subTitle : arrData[i].zibiaoti
            }
            listData.push(list);
        }
        var lv = appcan.listview({
            selector : "#listview",
            type : "thinLine",
            hasIcon : true,
            hasAngle : true,
            hasSubTitle : true,
            multiLine : 1
        });
        lv.set(listData);
        lv.on('click', function(ele, context, obj, subobj) {
        })
```
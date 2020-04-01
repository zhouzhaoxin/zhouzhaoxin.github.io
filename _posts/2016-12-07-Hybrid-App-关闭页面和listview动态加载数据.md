---
layout: post
title: "Hybrid App 关闭页面和listview动态加载数据"
author: zhouzhaoxin
categories: ["APP"]
image: assets/images/3.jpg
comments: false
---
AppCan 的页面是由两个HTML组成，如果要完全关闭的话需要在主HTML eg.index.html中关闭,关闭方法：
`appcan.window.close(-1);`

## 管道
- AppCan中两个页面通过管道连接，并传递数据
- appcan.window.publish(channelId,msg) 向指定通道发送消息
- appcan.window.subscribe(channelId,callback) 订阅一个频道，如果有消息发给该频道，则会执行响应的回调如果是用超链接打开的页面收不到消息

## 示例
需要注意的是，要确保publish方法执行过即开通了一个管道才可以接收到信息
```javascript
//在..._content页面发送消息
function send() {
    //发送消息
    appcan.window.publish('test', 'hello');
    alert("发送成功");
}

//在.. .html文件ready方法中打开管道
appcan.ready(function() {
    appcan.window.subscribe('test', function(msg) {
    if (msg == 'hello') {
        closeMyself();
    } else {
        alert("test");
    }
});

//执行获取正确信息后的方法
function closeMyself() {
    alert("closeMyself调用");
    appcan.window.close(-1);
}
```

## listview动态加载数据
appcan —> 添加列表 —->带图片的列表
```javascript
//自动生成

var lv = appcan.listview({
    selector : "#listview",
    type : "thinLine",
    hasIcon : false,
    hasAngle : true,
    hasSubTitle : true,
    multiLine : 1,
});
lv.set([{
    title : "临时数据",
    subTitle : "12:05",
    id : "1"
}, {
    title : "临时数据",
    subTitle : "12:05",
    id : "2"
}])
lv.on("click", function(ele, obj, curEle) {
})
```

```javascript
//从服务器获取数据
function getData() {
    var url = "服务器地址" + "用户登录id";
    apcan.request.getJSON(url, function(data) {
        //提前判断是否加载成功，现将data解析
        showMenu(data) {
        }
    }, 'json', function(err) {
        alert(err);
    }, "get", "", false);
}
```

```javascript
//将信息动态赋值给listview
function showMenu(data) {
    var lv = appcan.listview({
        selector : "#listview",
        type : "thinLine",
        hasIcon : false,
        hasAngle : true,
        hasSubTitle : true,
        multiLine : 1,
    });
    var datalist = data.data;
    for(i=0;i<data.datalist.length;i++){
        datalist[i].title = data.data[i].title;
        datalist[i].describ = data.data[i].content;
        datalist[i].subtit = data.data[i].uid;
        lv.set(datalist);
    }
}
```
---
layout: post
title: "Hybrid App Request和登录功能简单实现"
author: zhouzhaoxin
categories: ["APP"]
image: assets/images/17.jpg
comments: false
---
记录在apcan中对数据的请求获取

实现appcan中网络数据的上传和获取

#### 常用参数

```text
options.type:        请求的类型，包括GET、POST等
options.url:         要请求的地址 注：get方式请求中携带中文参数，需要对参数进行encode编码，具体函数：encodeURIComponent
options.data:        要请求的URL的参数,如果要上传文件则data数据中必须传一个对象包含一个path的key 例如：data:{file:{path:'a.jpeg'},file2:{path:'b.jpeg'}}上传                a.jpeg,b.jpeg图片
options.dataType:    服务端的响应类型，包括json, jsonp, script, xml, html, text中的一种
options.timeout:     请求的超时时间
options.success(data, status,,requestCode,response, xhr):    请求发送成功后的回调
options.error(xhr, errorType, error,msg):                    请求如果出现错误后的回调;msg: 错误详细信息，服务器返回的result信息

```

#### script代码
```javascript
appcan.button("#submit", "ani-act", function() {
           login();
       })
       function login() {
           var name = $("#username").val();
           var pwd = $("#password").val();
           console.log(name + ":" + pwd);
           appcan.ajax({
               url : "http://testmas.appcan.cn:9000/ODBC/login?uName=" + name + "&pwd=" + pwd,
               type : 'get',
               dateType : 'json',
               success : function(data, status, xhr) {
                   var obj = eval('(' + data + ')');
                   alert(obj.status);
                   if(obj.status == "0"){
                       alert("登录成功");
                   }else{
                       alert("用户名或密码不正确");
                   }
               },
               error : function(xhr, status, errMessage) {
                   alert("errMessage");
               }
           });
       }
```
#### html代码

```html
<body class="um-vp bc-bg" ontouchstart>
        <div class="ub ub-ver uinn-a3 ub-fv">
            <div class="ub ub-ver uinn uinn-at1">
                <div class="umar-a uba bc-border c-wh">
                    <div class="ub ub-ac ubb umh5 bc-border ">
                        <div class=" uinput ub ub-f1">
                            <div class="uinn fa fa-user sc-text"></div>
                            <input id="username" placeholder="手机/邮箱/用户名" type="text" class="ub-f1">
                        </div>
                    </div>
                    <div class="ub ub-ac umh5 bc-border ">
                        <div class=" uinput ub ub-f1">
                            <div class="uinn fa fa-lock sc-text"></div>
                            <input id="password" placeholder="密码" type="password" class="umw4 ub-f1">
                        </div>
                    </div>
                </div>
                <div class="ub ub-ver">
                    <div class="ub ub-pe uinn-a6 sc-text-active ulev-4">
                        忘记密码
                    </div>
                    <div class="uinn-at1">
                        <div class="btn ub ub-ac bc-text-head ub-pc bc-btn uc-a1" id="submit">
                            登录
                        </div>
                    </div>
                    <div class="uinn-at2 ub sc-text-active ulev-4">
                    </div>
                </div>
                <button type="submit"class="uinvisible"></button>
            </div>
        </div>
        <script src="js/appcan.js"></script>
        <script src="js/appcan.control.js"></script>
    </body>
```
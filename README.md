# 双十一前端总结

## 项目构建
    es6 + scss + gulp
[首页](http://www.embatabletennis.net/)
### 整体思路
* 抽离公共js模块和css模块。
* 每个逻辑构建独自js。
* 每个页面引入需要的js和css完成页面。
    * `es5`
            js将模块绑定在window上，通过gulp合并同一个js，通过window.*module的形式调用。
            `scss` 通过`@import`加载需要的css模块。
    * `es6`
            通过export和import的形式调用(需要webpack loader支持)。

    [index.js](https://gitlab.ixiaopu.com/xiangqu-static/xiangqu-campaign/blob/master/assets/activity/xiangqu-11-11/dev/js/index.js)
    ```javascript
    import '../style/index.scss';
    import './utils/utils.js'
    import {CanvasModule} from './canvas/canvas.js';
    import {ProductModule} from './product/product.js';
    import {ShopModule} from './shops/shops.js';
    ```
    [webpack.config.js](https://gitlab.ixiaopu.com/xiangqu-static/xiangqu-campaign/blob/master/assets/activity/xiangqu-11-11/webpack.config.js)
    ```javascript
    module: {
            loaders: [
                {
                    test: /\.js$/,
                    exclude: /node_modules/,
                    loader: 'babel',
                    query: {
                        presets: ['es2015']
                    }
                },
                {
                    test: /\.scss$/,
                    loader: "style!css!sass"
                }
                 ]
    }
    ```


### 目录结构
    目录结构跟以往项目一样开发与上线分离

```javascript
dev
  .js
  .cs
build
  .min.js
  .min.css
html
  .html
```
### 命名规范

* js 命名规范

    ```javascript
       s：表示字符串。例如：sName，sHtml；
       n：表示数字。例如：nPage，nTotal；
       b：表示逻辑。例如：bChecked，bHasLogin；
       a：表示数组。例如：aList，aGroup；
       r：表示正则表达式。例如：rDomain，rEmail；
       o：表示以上未涉及到的其他对象，例如：oButton，oDate；
       g：表示全局变量，例如：gUserName，gLoginTime；
       $：表示Jquery对象。例如：$Content，$Module；
       j：表示Jquery对象。例如：jContent， jModule；
       fn：表示函数。例如：fnGetName，fnSetAge；
       dom：表示Dom对象，例如：domForm，domInput；
    ```

### js框架
    通过自己封装util.js来代替zepto。包括基础的dom节点操作，事件绑定和ajax。

[utils.js](https://gitlab.ixiaopu.com/xiangqu-static/xiangqu-campaign/blob/master/assets/activity/xiangqu-11-11/dev-gulp/js/utils/utils.js)
```javascript
let UtilsModule = {
 init:function(){
   Array.prototype.shuffle = function(n){
    let len = this.length,num = n ? Math.min(n,len) : len,index,
        arr = this.slice(0),temp,
        lib = {};
        lib.range = function(min,max){
          return min + Math.floor(Math.random()*(max-min+1))
        }
        for(let i =0;i<len;i++){
          index = lib.range(i,len-1);
             temp = arr[i];
             arr[i] = arr[index];
             arr[index] = temp;
          }
          return arr.slice(0,num);
    }
 },
 addEvent:function(dom, event, fn) {
   dom.attachEvent?dom.attachEvent("on"+event,fn):dom.addEventListener(event,fn,!1)
 },
 requestAnimation:function(fn){
   let requestAnimationFrame = window.requestAnimationFrame ||
       window.mozRequestAnimationFrame ||
       window.webkitRequestAnimationFrame ||
       window.msRequestAnimationFrame ||
       window.oRequestAnimationFrame ||
     function(callback) {
       setTimeout(callback, 1000 / 60);
     };
     return requestAnimationFrame(fn);
 },
 ......
}
```

### 自动化工具

* webpack
    * webpack适合`react/vue`项目。对于普通项目，加载css+js文件，dom节点会有无样式阶段。
    * 结合gulp使用，小项目失去webpack意义(js模块化)。
    * `webpack`配合`es6` module很强大。
    
```javascript
    export class Module{};
    .....
    import {...} from 'js/css';
```
* gulp
    * 通过`gulp-babel`支持编译es6。对于es6 模块化，只能转成es5的require(..)，需要通过`browserify/webpack`等模块化工具实现。
    * 速度快，适合小项目


## 前后端联调
	通过nginx设置代理，时时联调。
### 接口
* /image/draw 生成图片
    * 入参
    ```javascript
    "pigments":{
      "width":图片宽度,
      "height":图片高度,
      "bgRGB":图片背景颜色,
      "snippets"：{
         "imgUrl":图片地址,
         "colorStr":字体颜色,
         "size":字体大小,
         "font":字体,
         "context":内容,
         "x":x坐标,
         "y":y坐标,
         "style":样式,
         "degree":旋转角度,
         "rx":旋转原点x坐标,
         "ry"旋转原点y坐标:
      }
    },
    "quality":
    ```
    * 出参
    ```javascript
    data:base64图片
    ```

* /common/wx/tmphost 获取域名
    * 入参
    ```javascript
    "preHost":当前域名
    ```
    * 出参
    ```javascript
    rs:{
     "data":{
       "host":域名
       "hostQrcodeUrl":域名二维码
     },
     "msg":信息
    }
    ```

### 配置nginx

* 代理到服务端ip。
* 模拟线上nginx gzip。
* 配置多个配置文件。

### 操作nginx
```javascript
    start nginx
    start nginx -c conf/nginx2.conf
    nginx -s reload/stop/..
    nginx -s reload -c conf/nginx2/conf
```
* 启动/重启不同配置文件。

### 配置文件

```javascript
    //gzip
    gzip on;
	gzip_min_length 1k;
	gzip_buffers 16 64k;
	gzip_http_version 1.1;
	gzip_comp_level 6;
	gzip_types text/plain application/x-javascript text/css application/xml;
	gzip_vary on;
	.....
	//代理
    upstream devxiangqutestproxy
    {
       server 10.8.100.107:8080;
    }
	upstream proxy_xq_api
	{
		server 10.8.82.26:8888;
	}
	location / {
       if ($args ~* "source=.*") {
          add_header Set-Cookie "FromChannel=$args;path=/";
         }
       proxy_pass http://devxiangqutestproxy;
       proxy_set_header Host $host:$server_port;
       proxy_set_header Cookie "$http_cookie;CAS_SID=$arg_cas_t";
       proxy_set_header X-Real-IP $remote_addr;
       proxy_http_version 1.1;
       proxy_set_header Connection "";
    }
    .....
	//本地项目访问路径
	location ~ ^/(build|velocity|assets|d11)/ {
        root F:\gitlab\xiangqu-campaign;
        concat on;
        concat_max_files 20;
        concat_unique off;
    }
    location ~ ^/(build|velocity|assets|d11)/ {
        root F:\gitlab\xiangqu-campaign;

    }
	.....
```

## 问题以及解决方案

### 开发测试阶段


* 移动端无法下载图片

	* 需求点击图片保存到本地。pc页面前端保存图片逻辑在移动端不适用，通过上传图片到服务端保存，iosAPP/微信支持性差。
	* 解决办法：原有需求改为长按图片保存。微信/浏览器调用自带菜单保存图片。app开发图片保存功能(IOS)。

	```javascript
	android
	window.ShareObj.shareContentWindow(JSON.stringify(oShareData));
    window.ShareObj.saveImage(sImgSrc);
    ios
    window.iOS.custShareItem(JSON.stringify(oShareData))
    iOS.saveImageBase64Callback(sImgSrc,function(succ){iOS.alert('保存成功');});
	```


* 特殊字体文件过大

    * 需求生成图片采用特殊字体，canvas引入第三方字体文件太大。
	* 解决方案：将字体文件安装在服务端，图片通过服务端生成，下发base64格式到前端。

	```javascript
	UtilsModule.ajax({
       method : 'post',
       url : '/image/draw',
       data : {
         pigments:JSON.stringify(self.oFontMsg),
         quality:'0.8f',
        },
       success : function (rs) {
         let sBase = rs ;
         self.viewResult(sBase);
       },
       async : true
    });
	```

* 不同病症文案显示样式。

    * 需求不同病症文案中字体大小，样式不同。按服务端接收格式，手动操作工作量太大。

    ```javascript
    {
      "colorStr": "0,0,0",
      "size": 120,
      "font": "方正兰亭特黑简体",
      "context": "风后", // 文案替换不同字号，不同字体需要重新定义。
      "x": 292,
      "y": 236,
      "style": 0
    }
    ```
    * 解决方案：自定义文案格式 $+@50@**&方正兰亭特黑简体&+$，通过就是逻辑拼装数据格式。

    ```javascript
    [
     '$+@50@&方正兰亭特黑简体&**+$不爱回人消息',
     '$+@70@&方正兰亭特黑简体&不是因为高冷+$',
     '$+@100@&方正兰亭特黑简体&而是因为手冷+$'
    ]
    $+ +$ 包含有特殊字体文本
    & &   特殊字体
    @ @   字号
    **    替换文案
    ```

* 微信/App用户浏览位置

    * 在微信/App中浏览首页商品查看商品后跳回首页，页面刷新，失去浏览位置。
    * 结局方案：此次商品数据存在本地，渲染所有商品dom节点，图片懒加载，通过localStorage保存当前滚动高度和最后浏览时间。再次进入这个页面时判断当前时间和上次浏览时间，判断是否读取记录。
    ```javascript
     if(window.localStorage && window.localStorage.getItem(vm+'ScrollTop')){
        let oStorage = window.localStorage,
         nScrollTop = oStorage.getItem(vm+'ScrollTop'),
         tScrollTime = oStorage.getItem(vm+'ScrollTime'),
         tNowTime = new Date().getTime();
         if((tNowTime-tScrollTime)/1000/60 < 5){
           document.body.scrollTop = nScrollTop;
         }
         else{
           oStorage.setItem(vm+'ScrollTop',0);
         }

     }
    ```

* 生成测试结构返回

    * 输入测试数据和生成测试结果为一个页面。生成结果后跳出该页面返回，页面失去测试结果数据。
    * 解决方案：生成测试结果时刷新页面，在链接后面拼装用户输入数据。页面刷新时判断链接后面数据格式显示结果。

    ```javascript
    let oXz = XZ[inputObj.monthDate],
      sXz =  inputObj.dayDate <= oXz.division ? oXz.items[0] : oXz.items[1],
      sBz = XZBZ[sXz].shuffle(1),
      oBz = BZ[sBz],
      nShuffle = Math.floor(Math.random()*oBz.items.length),
      sLink = '';
    sLink = location.href+'?userName='+inputObj.userName+'&bz='+sBz+'&shuffleNum='+nShuffle;
    //?userName=风后&bz=QPZ&shuffleNum=0
    location.href = sLink;
    ```


* 浏览器惯性滑动。

    * 需求导航栏在页面滚动到一定位置时固定在页面顶部。但在微信浏览器中，会出现惯性滚动的事件(不可监听，无法执行js)。此时导航栏定位会出现一定延时。
    * 待解决：结构变动过大
    * 待解决方案：Iscroll插件。通过自己模拟滚动事件。


### 上线阶段

* 二维码识别

    * 不同机型二维码辨识度不同。
    * 解决方案：增加二维码尺寸。缩短二维码链接。

    ```javascript
     http://www.embatabletennis.net/velocity/index.html
     变为
     http://www.embatabletennis.net
    ```

* 广点通隐藏部分商品

	* 数据结构定义全在前端，考虑情况不全。运营提出需求在不同的推广渠道展示不同的商品列表。原商品数据结构不支持。
	* 解决方案：重新构建数据结构，新增一个渠道字段。




* 暂存本地记录

    * 需求首页进去显示卫衣弹层，5分钟内刷新页面不显示。androidApp禁止了localStorage。
    * 解决方案：用cookie代替(在不支持localStorage的浏览器，也可以通过这种方式)。

    ```javascript
    if(window.localStorage){
        UtilsModule.setLocalStorage(key,value)
        ...
        UtilsModule.getLocalStorage(key)
        ...
    }
    else{
        UtilsModule.setCookie(key,value,time);
        ...
        UtilsModule.getCookie(key);
        ...
    }

    ```

* 生成图片字体模糊

    * 在dpr=2 的显示器下，图片是750*750中dpr=1下生成的，所以会有模糊。
    * 解决方案：图片尺寸x2，字体大小x2，前端拿到图片只显示750尺寸。缺点：base64数据格式增加(增加近180kb)。


* 域名跳转

    * 项目发布在阿里云服务器下，访问第三方域名首页点击shop或product跳回想去域名下地址。
    * 解决方案：nginx做forward跳转(具体实现服务端配置nginx)。

            forward:是服务器内部的操作，通过服务器访问目标地址url，把url的响应内容读取过来显示。地址栏不变。可以共享request请求数据。
            redirect:即重新定向。客户端请求，服务端通知客户端发起跳转请求，跳转到指定url。地址栏改变。不能共享数据。

* 域名被封
    * 第三方域名在微信被封
    * 解决方案：配置域名文件，前端接口请求，按配置文件顺序下发（排除当前域名），发现被封，修改配置文件。

    ```javascript
    被封域名
        hhcz2009.net
        baoxianggui.net
        www.dcstgroup.cn
        abc.xy2016.dcstgroup.cn
        abc1.xy2016.dcstgroup.cn
         abc2.xy2016.dcstgroup.cn

    推广未封
        www.embatabletennis.net
        mai.pian.tierxq.cn
        dian.xian.odinxq.cn
        bei.zi.limuxq.cn
        xqabc.limu.lcar999.cn
        xqaaa.tier.lcar999.cn
        www.fenghouxq.cn
    ```






	






> SENOU机器人控制器平台，提供了丰富的接口，通过简单的网页文件就可以驱动复杂，能力强大的机器人，本文提供了快速连接到局域网内部机器人，获取状态和移动机器人的方法。阅读本文大概需要10分钟。

# 改造浏览器
电脑中安装了一个非常强大的IDE工具，就是“浏览器”，以Chrome浏览器为例，我们可以使用F12键将浏览器的命令行打开，并使用这个工具。

但是在使用前，我们需要对浏览器进行改造，将它变成一个超级调试，工具——关闭浏览器的安全设置，允许它访问本地文件。在浏览器图标快捷方式上点击右键-属性，在“目标”的输入框的最后，添加下面的配置项（开发时可以关闭，平时建议打开浏览器的安全防护）。
```sh
    --disable-web-security --user-data-dir
```

好了，现在双击浏览器图标，浏览器已经变成一个超级IDE了。

# 新建html文件
新建文件夹，并在文件夹内新建一个html文件。在里面输入如下结构(大部分的编辑器对于html文件都有自动提示功能，建议使用VS code，或者sublime编辑器)
```HTML
    <!DOCTYPE html>
    <html>
    <head>
        <title></title>
    </head>
    <body>

    </body>
    </html>
```

# 引入axios和socket.io
在body标签的后面，引入axios.js和socket.io.js文件
```html
<!DOCTYPE html>
<html>

<head>
    <title></title>
    <!-- <style type="text/css">如果有样式信息请定义到此处</style> -->
</head>

<body>
    <!-- 之后所有的界面都写在此处 -->
</body>
<script type="text/javascript" src="./axios.js"></script>
<script type="text/javascript" src="./socket.io.js"></script>
<script type="text/javascript">
    //之后所有的函数都定义在此处
</script>

</html>
```
这两个文件可以从启玄科技的官网下载 [axios.js](https://www.robotmeta.com/axios.js)， [socket.io.js](https://www.robotmeta.com/socket.io.js)。

# 配置axios
在socket.io.js后面新建script标签。并配置axios的请求地址（地址以实际机器人内网地址为准），请求头，请求参数的处理回调（直接粘贴就可以，具体作用以后介绍）。

```javascript
var apis = 'http://192.168.0.123:1029' // 此处改成机器人的IP地址

axios.defaults.headers.common['Content-Type'] = 'application/x-www-form-urlencoded'

axios.defaults.transformRequest = [function(data) {
	let ret = ''
    for (let it in data) {
        ret += encodeURIComponent(it) + '=' + encodeURIComponent(data[it]) + '&'
    }
return ret
}]
```

# 查看机器人软件版本
在<body>标签中增加一个获取版本信息的按钮（并绑定click事件）和一个用于显示结果的<span>。
```html
<h1>获取版本</h1>
<div>
    <span id='display'></span>
    <button onclick="versionBtnHdl()">获取版本</button>
</div>
```

在script标签内定义获取版本信息的方法。此时我们会用到axios，在axios的then方法内定义获取到之后的回调方法。并显示到界面上。

```javascript
function versionBtnHdl() {
    axios({
            method: "GET",
            url: apis + "/system/version",
        })
        .then(function(res) {
            display.innerHTML = res.data.data.robot
        })
}
```
点击按钮，你已经可以但看到，机器人的版本信息显示在了界面上。这是一个非常激动人心的开始。因为SENOU控制器平台，提供了和机器人，生产工艺相关的绝大部分功能都可以通过这种方式调用。

#上电&上使能
在界面上添加如下标签
```html
<h1>电源和伺服</h1>
<button onclick="powerBtnHdl()">打开电源</button>
<button onclick="servoBtnHdl()">打开使能</button>
```

在脚本部分添加如下方法：
```javascript
function powerBtnHdl() {
    axios({
            method: "POST",
            url: apis + "/robot/power",
            data: { status: 1 },
        })
        .then(function(res) {
            alert(res.data.msg)
        })
}

function servoBtnHdl() {
    axios({
            method: "POST",
            url: apis + "/robot/servo",
            data: { status: 1 },
        })
        .then(function(res) {
            console.log(res.data)
        })
}
```
此处，我们使用了浏览器自带的alert()方法，将上电的结果以“阻塞式模态”的形式显示出来。离控制机器人运动就只有一步了。
# 控制机器人
我们再使用一个html的标签select。在界面部分追加如下代码：
```html
<h1>关节移动</h1>
<button onmousedown="jogDown(1)" onmouseup='jodUp()'>JOG+</button>
<button onmousedown="jogDown(-1)" onmouseup='jodUp()'>JOG-</button>
<select onchange="changeHdl(event)">
    <option value="1">1</option>
    <option value="2">2</option>
    <option value="3">3</option>
    <option value="4">4</option>
    <option value="5">5</option>
    <option value="6">6</option>
</select>
```
因为运动可以有两个方向，所以我们通过1和-1代表转动方向。接着，我们需要定义三个方法，模拟实体按键，上升沿和下降沿的事件。
```javascript

var joint = 1
function jogDown() {
    axios({
            method: "POST",
            url: apis + "/robot/jog_start",
            data: {
                axis: joint,
                dir: 1
            },
        })
        .then(function(res) {
            console.log(res.data.msg)
        })
}

function jodUp() {
    axios({
            method: "POST",
            url: apis + "/robot/jog_stop",
        })
        .then(function(res) {
            console.log(res.data.msg)
        })
}

function changeHdl(e) {
    console.log(e.target.value)
}
```
# socket通讯
我们使用socket的方式，实现服务器端向客户端推动消息，和机器人的实时监控。在界面部分增加一个div显示机器人实时监控的结果。
```html
<h1>socket</h1>
<div id='joint_cmd'>display</div>
```
在脚本部分，通过socket连接机器人，并获取实时数据。
```javascript
var socket = io(apis)

socket.on('connect', function() {
    console.log('connected')
})
socket.on('system_status', function(res) {
    for (var i = res.data.joint_cmd.length - 1; i >= 0; i--) {
        res.data.joint_cmd[i] = res.data.joint_cmd[i].toFixed(2)
    }
    joint_cmd.innerHTML = res.data.joint_cmd.join('</br>')
})
```

# 增加样式
最后，我们给按钮添加一些简单的样式。在head标签内增加样式代码
```html
<style type="text/css">
button {
    background: #8bc34a;
    padding: 8px 16px;
}
</style>
```

SENOU为您提供了更多的api接口，您可以使用这些接口，快速简单的定制符合您的机器人控制器。更多接口请访问[SENOU官网](http://senou.robotmeta.com)，或 [在github上下载示例代码](https://github.com/timAnim/senou_demo.git)。



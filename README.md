> 某一天 我只是想模拟实现一下百度下拉搜索 结果发现小小的一点代码 居然可以有这么多的知识点 涉及到JSONP 以及 事件委托 还有如何实现在点击页面其他地方的时候实现下拉框的收起...生命在于折腾

### (●'◡'●) 不用后台请求 不用ajax请求 直接使用百度的jsonp请求地址进行请求就可以啦~~~(●'◡'●)
####1 先看结果
![test.gif](http://upload-images.jianshu.io/upload_images/5763769-b652a3c8a77e5375.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####2 涉及知识点

- JSONP 是什么？💁
- 事件委托的使用。🤦
- 点击其他位置实现下拉框收起。🙌

####3 知识点详解
- JSONP  
> 我们都知道 在使用ajax请求的时候 跨域这种情况是要另外处理的 那JSONP就是可以解决这种问题的。具体是使用什么原理呢？你想一下，平时我们在页面中引入script标签之后，script标签也有相关的url发起请求，而且可以请求任何有效地址的数据，正是因为这个漏洞，我们可以发起跨域请求。那么机智的程序员们，就这么想，我可不可以将需要用到的JSON 数据填充回来.所以就有了JSONP。

JSONP执行之后，会返回包含json编码数据的响应体，也就是会自动执行。
当通过<script>元素调用数据时，响应内容必须是用js函数名和圆括号包裹起来，而不是发这样的json数据。
```
[1,2,{"name": "katherine"}]
```
它会发送一个这样被js包裹后的JSON响应
```
handleResponse (
[1,2,{"name": "katherine"}]
)
```
所以我们可以事先在js代码中定义一个类似handleResponse的函数，这样我们就可以在响应获取到这些JSON响应之后，通过事先定义好的函数，对传过来的数据为所欲为了。

其实说简单一点，JSONP就是在你向页面填充某个url的<script>标签之后，返回一个可以执行的函数名，同时是有带参数的函数名。所以只要我们事先定义好这个函数的相关操作就可以了。

当然，如果这个函数名是定死的话，那就不好玩了，所以一般支持JSONP的服务不会强制指定客户端必须实现的回调函数名，如果每次函数名都一定是```handleResponse```,岂不是很受限。所以，一般会用查询参数的值，允许客户端指定一个函数名，然后使用函数名去填充响应。也就是说我们在这个```script```标签的url给定地址中填充类似 ```？cb=callbackname```来告诉服务端，我们这边客户端需要什么样的函数名。

这里推荐一个觉得讲得很好的地址：[谈谈JSON和JSONP](http://www.cnblogs.com/dowinning/archive/2012/04/19/json-jsonp-jquery.html)

后面我在实际应用中也会说到。

- 事件委托

在这个例子中，需要实现一个就是，当有下拉热词出现的时候，其实是包裹来一个ul内部的，而这些填充的很多的li，每一个li都需要绑定一个click事件，用来干什么呢？当点击其中一个li的时候，需要将这个li的内容填充到搜索输入框。

因为这些li是动态生成的，不可能每一个li都去绑定点击事件。而且这样对于性能消耗也很大，所以，可以使用事件冒泡的特点，将点击件绑定在父级元素ul上。

然后通过事件的属性，```event.target ```来获取当前真正被点击的元素，然后就可以获取这个被点击元素的内容啦。😉

因为篇幅有限 我也没有讲得很详细  传送门： [事件委托和事件代理](https://www.cnblogs.com/owenChen/archive/2013/02/18/2915521.html)

#### 4 实际代码 
```
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <title>Title</title>
    <body>
        <div id="search-arr"><input type="text" id="searchInput" type="search" placeholder="search"><ul id="selectItem"></ul></div>
    </body>
  </head>
</html>
  <script>
    // 执行JSONP操作的函数，设置添加script.
// 这个看不懂的话 建议参考一下js权威指南 第507页
      function getJSON (url,callback) {
        // 因为百度的那个请求的地址是 https://sp0.baidu.com/5a1Fazu8AA54nxGko9WTAnF6hhy/su?wd=迷糊查询参数名&cb=回调函数名
        // 所以这里会每次给getJSON 函数添加一个
        var cbnum = "cb" + getJSON.counter++  // 假设这里就是cb1
        var cbname = "getJSON." + cbnum  // 这里的函数名字就是 getJSON.cb1
        url+="&cb="+ cbname    // 将这个回调函数名添加 到url后面
        var script = document.createElement("script")   // 创建一个script标签
        getJSON[cbnum] = function(response) {   // 给这个getJSON函数添加一个getJSON.cb1的函数属性。这也就是我们的回调函数
          try {
            callbackfn(response)    // 对获取到的response数据进行处理的回调函数
          }
          finally {   // 记得哦 执行完函数之后 要删除这个添加的属性  同时删除这个script标签 不然查询次数增加之后  会越来越多
            delete getJSON[cbname]
            script.parentNode.removeChild(script)
          }
        }
        script.src = url   // 添加url属性
        document.body.appendChild(script)
      }
      getJSON.counter = 0   // 初始化一个counter的值
      // 对获取到的JSONP数据进行处理的回调函数
      function callbackfn(data) {
        var selectList = document.getElementById("selectItem")
        // 可以通过视频中看到，数据的结构中的关键字存在data.s中
        if (data.s.length !== 0) {　　　
          let newList = data.s.map((item) => `<li><a target="_blank" href="https://www.baidu.com/s?wd=${item}">${item}</a></li>`) 
          let newStr = newList.join('')
          selectList.innerHTML = newStr
          selectList.style.display = 'block'
        }
      }
    window.onload = function () {
      var target = document.getElementById("searchInput")
      var selectList = document.getElementById("selectItem")
      AutoCompleteInput()
      //  定义一个隐藏下拉单的函数
      function hideList (selectList) {
        selectList.innerHTML = ''
        selectList.style.display = 'none'
      }
      // 对输入框进行监听 数据变化就执行getJSONP函数，请求新的关键字
      function AutoCompleteInput() {
        target.addEventListener('input', () => {
          let oldValue = ''
          let newValue = target.value
          if (newValue !== '' && newValue !== oldValue) {
            getJSON('https://sp0.baidu.com/5a1Fazu8AA54nxGko9WTAnF6hhy/su?wd='+(newValue))
          } else {
            hideList(selectList)
          }
          oldValue = target.value
        })
        // 点击除了下拉框和输入框之外的其他地方 要隐藏下拉框
        document.addEventListener('click', (e) => {
          // 这里使用了contains，contaisn可以知道某个元素是否是其子元素
          if(selectList.contains(e.target)||e.target.getAttribute('id') !== 'searchInput') {
            hideList(selectList)
          }
        })
        target.addEventListener('keydown', () => {
          hideList(selectList)
        })
        // 事件代理部分代码 
        selectList.addEventListener('click', (e) => {
          e.preventDefault()   // 我为了防止点击之后跳转到搜索页面，所以写了这句，你可以去掉，就可以跳转到搜索结果页面了
          let selectValue = ''
          if (e.target && e.target.nodeName == "A") {
            selectValue = e.target.innerHTML  
          } else {
            selectValue = e.target.getElementByTagName('a').innerHTML
          }
          target.value = selectValue
        })
      }
}
  </script>
<style>
* {
  padding:0;
  margin:0;
  box-sizing:border-box;
}
#search-arr {
  position:relative;
  width:200px;
  margin:50px auto;
}
#searchInput {
  outline: none;
  width:200px;
  height:30px;
  border:1px solid rgba(109,207,246,.5);
  border-radius:25px;
  padding:10px 10px 10px 20px;
  background:#ededed;
  transition:all ease .4s;
}
#searchInput:focus {
  /* width:250px; */
  box-shadow: 0 0 5px rgba(109,207,246,.5);
  background:white;
}
ul#selectItem {
  display: none;
  position:absolute;
  top:31px;
  width:100%;
  border:1px solid gray;
  border-top:none;
}
li {
  width:100%;
  list-style-type:none;
  background:white;
  color:black;
  /* padding:10px; */
  cursor:pointer;
}
li:hover {
    background: #ededed;
}
a {
  display: inline-block;
  text-decoration: none;
  color:black;
  width:100%;
  padding:1px 5px;
}
</style>
```
####5 遗留问题
**1 JSONP请求的错误处理：** 我想了这个问题，像我们在执行ajax请求的时候，是可以有相应状态码告诉我们发生错误的，404啊，之类的。但是这个JSONP请求我们要怎么知道错误呢？怎么处理错误呢？
**2 JSONP请求之后动态添加script标签，需要移除这些新添加的script标签，那是不是也会有漏洞或者会有问题呢？**

😢然后我惊讶地发现，有人的问题和我一样，我就。。。。 [遗留问题](https://segmentfault.com/q/1010000000483131/a-1020000000484884)
我看了答案还不是很理解。
会的欢迎评论哟~~~🤦





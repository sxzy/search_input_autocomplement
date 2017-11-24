> 某一天 我只是想模拟实现一下百度下拉搜索 结果发现小小的一点代码 居然可以有这么多的知识点 涉及到JSONP 以及 事件委托 还有如何实现在点击页面其他地方的时候实现下拉框的收起...生命在于折腾

### (●'◡'●) 不用后台请求 不用ajax请求 直接使用百度的jsonp请求地址进行请求就可以啦~~~(●'◡'●)
#### 1 先看结果
![test.gif](http://upload-images.jianshu.io/upload_images/5763769-b652a3c8a77e5375.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

#### 2 涉及知识点

- JSONP 是什么？💁
- 事件委托的使用。🤦
- 点击其他位置实现下拉框收起。🙌

#### 3 知识点详解
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
见文件
```
#### 5 遗留问题
**1 JSONP请求的错误处理：** 我想了这个问题，像我们在执行ajax请求的时候，是可以有相应状态码告诉我们发生错误的，404啊，之类的。但是这个JSONP请求我们要怎么知道错误呢？怎么处理错误呢？
**2 JSONP请求之后动态添加script标签，需要移除这些新添加的script标签，那是不是也会有漏洞或者会有问题呢？**

😢然后我惊讶地发现，有人的问题和我一样，我就。。。。 [遗留问题](https://segmentfault.com/q/1010000000483131/a-1020000000484884)
我看了答案还不是很理解。
会的欢迎评论哟~~~🤦





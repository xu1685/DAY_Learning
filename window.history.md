# 关于window.history 与 window.location
## window.history
### window.onpopstate 与 popstate
1. 关系  
window.onpopstate是`popstate事件`在window对象上的事件处理程序.
2. 触发条件  
每当处于激活状态的历史记录条目发生变化时（**并且页面没有重新加载**）,popstate事件就会在对应window对象上触发.  
( 什么是处于激活状态的历史条目？ 当前的url就是history条目中的激活状态条目，为了方便理解，可以将 处于激活状态的历史条目 称为（浏览器）当前url )
3. 特例  
但是history的pushState和replaceState方法也会改变当前url，这两种情况是不会触发popstate的，改变后的url当下也不会被加载


### (window.)history.pushState 和 replaceState
html5中，history新增两个方法
1. 语法  
history.pushState(stateObj, title, [,url]);  
history.replaceState(stateObj, title, [,url]);  

2. 相关理解  
    * url可以是绝对路径也可以是相对路径,如果url参数与当前页面的url不同源，会抛出错误  
    * 不同点：replaceState是修改覆盖当前history条目，pushState是新增一条 
    * 相同点：都会改变浏览器当前url，更新history条目，但是url被修改后不会立即重载，页面不变化，不会触发poostate事件。
    * 当 页面刷新 或者 在浏览器部分行为（history.back()、history.forward()、history.go()方法）下，锚定到该条history条目时，会触发popstate事件，并加载到匹配页面
    * 执行完上面两个方法后 ，`stateObj`会和`url`对应的历史条目绑定在一起。即该url下的`history.state` 是 `stateObj` 的一个拷贝

### Example：
假如当前网页地址为 http://example.com/example.html 
 ```  
 window.onpopstate = function(popstateEvent) {
    alert("location: " + window.location + ", state: " + JSON.stringify(popstateEvent.state));
 };
```
  `window.location` 和 `document.location` 互相等价的
``` 
//绑定事件处理函数. 
history.pushState({page: 1}, "title 1", "?page=1");    
//添加并激活一个历史记录条目 http://example.com/example.html?page=1
//(说人话就是 更改当前url，添加一条history)  

history.pushState({page: 2}, "title 2", "?page=2");    
//添加并激活一个历史记录条目 http://example.com/example.html?page=2

history.replaceState({page: 3}, "title 3", "?page=3"); 
//修改当前激活的历史记录条目 http://ex..?page=2 变为 http://ex..?page=3
//(说人话就是 覆盖当前url，修改这条history)

history.back(); 
// "location: http://example.com/example.html?page=1, state: {"page":1}"  
history.back();   
// "location: http://example.com/example.html, state: null"  
history.go(2);   
// "location: http://example.com/example.html?page=3, state: {"page":3}"  
```

## window.location
围绕两个重要属性 location.hash 和 location.href  
这两个属性都可以被读写
### (window.)location.hash
window.onhashchange 与 hashchange
1. 关系  
window.onhashchange 是 hashchange 在当前window对象上的事件处理程序.
2. 触发条件  
当我们只更改了URL的片段标识符 (跟在＃符号后面的URL部分，包括＃符号)（**页面没有重新加载(document会变化)**）,hashchange事件就会在对应window对象上触发.  
3. 例子  
假设当前页面url为 http://example.com/example.html
```
window.onhashchange = function(){
  console.log('hash changes')
}
location.hash = '#123'
// 页面url改变 更改了页面标识符（hash值）http://example.com/example.html/#123
//（控制台打印） hash changes
```

### (windows.)location.href
对location.href进行写操作可以将页面重新定位到一个新的url，页面会被加载

### location的几种方法
* location.reload([param])  
  * 作用是重载当前文档，参数可选  
  * 如果没有参数或者参数是false，重新加载页面时会用http头 If-Modified-Since 来检测服务器上的文档是否已经改变，没有改变则从缓存取，改变的话就重新从服务器下载
  * 如果参数为true，则不管文档的最终修改日期是什么，都会直接绕过缓存从服务器重新下载文档
* location.assign(url)
  * 作用是加载新的文档
  ```
  window.location.assign('http://www.mozilla.org')
  // 等同于
  window.location = 'http://www.mozilla.org'
  ```
* location.replace(url)
  * 作用是加载新的文档替换当前文档
  * 不会在 History 对象中生成一个新的记录，新的 URL 将覆盖 History 对象中的当前记录


#### 参考文章
[HTML5 History API 和 Location 对象剖析](https://hijiangtao.github.io/2017/08/20/History-API-and-Location-Object/)  
[MDN web docs - window.onpopstate](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/onpopstate)  
[MDN web docs - Manipulating the browser history](https://developer.mozilla.org/zh-CN/docs/Web/API/History_API)
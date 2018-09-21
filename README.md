# 历史搜索框
## 仿百度，jsonp写的历史记录和搜索框
相信大多数前端开发者在需要与后端进行数据交互时，为了方便快捷，都会选择JQuery中封装的AJAX方法，但是有些时候，我们只需要JQuery的AJAX请求方法，而其他的功能用到的很少，这显然是没必要的。<br> 
其实，原生JavaScript实现AJAX并不难，这篇文章将会讲解如何实现简单的AJAX，还有跨域请求JSONP！<br> 
### 一、AJAX <br>
AJAX的核心是XMLHttpRequest。 <br>
一个完整的AJAX请求一般包括以下步骤：<br>
实例化XMLHttpRequest对象<br>
连接服务器<br>
发送请求<br>
接收响应数据<br>
我将AJAX请求封装成ajax()方法，它接受一个配置对象params。<br>
function ajax(params) {<br> 
 params = params || {}; <br>
 params.data = params.data || {}; <br>
 // 判断是ajax请求还是jsonp请求<br>
 var json = params.jsonp ? jsonp(params) : json(params); <br>
 // ajax请求 <br>
 function json(params) { <br>
  // 请求方式，默认是GET<br>
  params.type = (params.type || 'GET').toUpperCase();<br>
  // 避免有特殊字符，必须格式化传输数据<br>
  params.data = formatParams(params.data); <br>
  var xhr = null; <br> 

  // 实例化XMLHttpRequest对象 <br>
  if(window.XMLHttpRequest) {<br> 
   xhr = new XMLHttpRequest(); <br>
  } else { <br>
   // IE6及其以下版本 <br>
   xhr = new ActiveXObjcet('Microsoft.XMLHTTP'); <br>
  };<br>


  // 监听事件，只要 readyState 的值变化，就会调用 readystatechange 事件<br>
  xhr.onreadystatechange = function() { <br>
   // readyState属性表示请求/响应过程的当前活动阶段，4为完成，已经接收到全部响应数据<br>
   if(xhr.readyState == 4) { <br>
    var status = xhr.status;<br> 
    // status：响应的HTTP状态码，以2开头的都是成功<br>
    if(status >= 200 && status < 300) { <br>
     var response = '';<br>
     // 判断接受数据的内容类型<br>
     var type = xhr.getResponseHeader('Content-type'); <br>
     if(type.indexOf('xml') !== -1 && xhr.responseXML) { <br>
      response = xhr.responseXML;<br>
      //Document对象响应 <br>
     } else if(type === 'application/json') { <br>
      response = JSON.parse(xhr.responseText);<br>
      //JSON响应 <br>
     } else { <br>
      response = xhr.responseText; <br>
      //字符串响应 <br>
     }; 
     // 成功回调函数<br>
     params.success && params.success(response); <br>
   } <br>
   else { <br>
     params.error && params.error(status);<br> 
   } <br>
   }; <br>
  }; <br>
  
  // 连接和传输数据 <br>
  if(params.type == 'GET') {<br>
   // 三个参数：请求方式、请求地址(get方式时，传输数据是加在地址后的)、是否异步请求(同步请求的情况极少)；<br>
   xhr.open(params.type, params.url + '?' + params.data, true); <br>
   xhr.send(null); <br>
  } else { <br>
   xhr.open(params.type, params.url, true); <br>
   //必须，设置提交时的内容类型 <br>
   xhr.setRequestHeader('Content-Type', 'application/x-www-form-urlencoded; charset=UTF-8');<br>
   // 传输数据<br>
   xhr.send(params.data); <br>
  } <br>
 }<br> 
 
 //格式化参数<br> 
 function formatParams(data) { <br>
  var arr = []; <br>
  for(var name in data) {<br>
   // encodeURIComponent() ：用于对 URI 中的某一部分进行编码<br>
   arr.push(encodeURIComponent(name) + '=' + encodeURIComponent(data[name])); <br>
  }; 
  // 添加一个随机数参数，防止缓存 <br>
  arr.push('v=' + random()); <br>
  return arr.join('&'); <br>
 }<br>



 // 获取随机数 <br>
 function random() { <br>
  return Math.floor(Math.random() * 10000 + 500); <br>
 }<br>

}<br>
## ajax
ajax({ <br>
 url: 'test.php',  // 请求地址<br>
 type: 'POST',  // 请求类型，默认"GET"，还可以是"POST"<br>
 data: {'b': '异步请求'},  // 传输数据<br>
 success: function(res){  // 请求成功的回调函数<br>
  console.log(JSON.parse(res)); <br>
 },<br>
 error: function(error) {}  // 请求失败的回调函数<br>
});<br>


## 二、JSONP 
### 同源策略 
AJAX之所以需要“跨域”，罪魁祸首就是浏览器的 同源策略。即，一个页面的AJAX只能获取这个页面相同源或者相同域的数据。 如何叫“同源”或者“同域”呢？——协议、域名、端口号都必须相同。例如：<br>
http://example.com  和  https://example.com 不同，因为协议不同；<br> 
http://localhost:8080  和  http://localhost:1000 不同，因为端口不同； <br>
http://localhost:8080  和  https://example.com 不同，协议、域名、端口号都不同，根本不是一家的。<br>
当跨域请求时，一般都会看到这个错误： XMLHttpRequest cannot load http://ghmagical.com/article/?intro=jsonp%E8%AF%B7%E6%B1%82&v=5520. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://localhost' is therefore not allowed access.<br>

那如何跨域请求呢？这时，JSONP就登场了！<br> 
JSONP(JSON with Padding) 是一种跨域请求方式。主要原理是利用了script 标签可以跨域请求的特性，由其 src 属性发送请求到服务器，服务器返回 JavaScript 代码，浏览器接受响应，然后就直接执行了，这和通过 script 标签引用外部文件的原理是一样的。<br>

JSONP由两部分组成：回调函数和数据，回调函数一般是在浏览器控制，作为参数发往服务器端（当然，你也可以固定回调函数的名字，但客户端和服务器端的名称一定要一致）。当服务器响应时，服务器端就会把该函数和数据拼成字符串返回。 <br>
JSONP的请求过程：
请求阶段：浏览器创建一个 script 标签，并给其src 赋值(类似 http://example.com/api/?callback=jsonpCallback ）。
发送请求：当给script的src赋值时，浏览器就会发起一个请求。<br>
数据响应：服务端将要返回的数据作为参数和函数名称拼接在一起(格式类似”jsonpCallback({name: 'abc'})”)返回。当浏览器接收到了响应数据，由于发起请求的是 script，所以相当于直接调用 jsonpCallback 方法，并且传入了一个参数。<br>

对于JQuery的JSONP请求，这里就不多讲了，之前也写过一篇文章《JQuery的Ajax请求跨域问题》。 <br>
在这里讲解一下用原生JavaScript如何实现。 <br>
依旧是ajax()方法里添加JSONP，后面会将两者整合在一起，JSONP的配置参数主要多了一个jsonp参数，它就是你的回调函数名。<br>
   function ajax(params) { <br>
       params = params || {}; <br>
       params.data = params.data || {}; <br>
       var json = params.jsonp ? jsonp(params) : json(params); <br>  
    

     // jsonp请求 <br>
     function jsonp(params) { <br>
      //创建script标签并加入到页面中 <br>
      var callbackName = params.jsonp; <br>
      var head = document.getElementsByTagName('head')[0]; <br>
      // 设置传递给后台的回调参数名 <br>
      params.data['callback'] = callbackName; <br>
      var data = formatParams(params.data); <br>
      var script = document.createElement('script'); <br>
      head.appendChild(script); <br> 
      
      //创建jsonp回调函数<br> 
      window[callbackName] = function(json) {<br> 
       head.removeChild(script); <br>
       clearTimeout(script.timer); <br>
       window[callbackName] = null; <br>
       params.success && params.success(json); <br>
      };  <br>
    

      //发送请求 <br>
      script.src = params.url + '?' + data;  <br>
    

      //为了得知此次请求是否成功，设置超时处理 <br>
      if(params.time) {<br> 
      script.timer = setTimeout(function() { <br>
       window[callbackName] = null; <br>
       head.removeChild(script); <br>
       params.error && params.error({ <br>
        message: '超时'<br> 
       }); <br>
      }, time); <br>
      } <br>
     };  <br>
    

     //格式化参数 <br>
     function formatParams(data) { <br>
      var arr = []; <br>
      for(var name in data) { <br>
       arr.push(encodeURIComponent(name) + '=' + encodeURIComponent(data[name])); <br>
      }; <br>
    

      // 添加一个随机数，防止缓存 <br>
      arr.push('v=' + random());<br> 
      return arr.join('&'); <br>
     } 
    

     // 获取随机数<br> 
     function random() { <br>
      return Math.floor(Math.random() * 10000 + 500); <br>
     }<br>
}<br>
注意：因为 script 标签的 src 属性只在第一次设置的时候起作用，导致 script 标签没法重用，所以每次完成操作之后要移除； 
使用实例：<br>
  ajax({ <br>
       url: 'test.php',  // 请求地址<br>
       jsonp: 'jsonpCallback', // 采用jsonp请求，且回调函数名为"jsonpCallbak"，可以设置为合法的字符串<br>
       data: {'b': '异步请求'},  // 传输数据<br>
       success:function(res){  // 请求成功的回调函数<br>
        console.log(res); <br>
       },<br>
       error: function(error) {}  // 请求失败的回调函数<br>
      });<br>
在这里后台使用PHP处理：<br>
    <?php<br>
   
      $data = array('type'=>'jsonp');<br>
      $callback = isset($_GET['callback']) ? trim($_GET['callback']) : ''; <br>
       echo $callback.'('.json_encode($data).')';<br>
        注意：别漏了用函数名与数据拼接返回。 <br>
       当然，前面也说过，你可以给定固定回调函数名：<br>
  function jsonpCallback() {}<br>
    

    <?php<br>
     echo 'jsonpCallback('.$data.<br>

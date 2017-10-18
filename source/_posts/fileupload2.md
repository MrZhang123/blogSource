---
title: 文件上传（二）---新XMLHttpRequest实现带进度条文件上传
date: 2016-04-14 23:10:58
tags: H5 file，canvas
comments: true
categories: "Javascript"
---
前几天搞得那个文件上传，最近工作不太忙，又开始完善。这次主要添加了文件上传的进度部分，主要用到的则是XMLHttpRequest2的progress，由于我读的是《javascript高级程序设计》（第三版），这里我遇到一个关于progeress事件的坑，后面详细解答。
## 创建XMLHttpRequest对象
首先说说XMLHttpRequest，在IE7+浏览器中，只需要new一个XMLHttpRequest对象即可：
```javascript
let xhr=new XMLHttpRequest();
```
由于现在淘宝都不再支持IE6以及7，所以这里不考虑IE7及以下浏览器关于xhr的创建方法。
<!--more-->
### open()方法
在使用XHR对象时候，第一个方法是open()方法，它接受3个参数：
* 要发送的请求的类型（"get"、"post"等）；
* 请求的URL；
* 是否异步发送请求的布尔值。
例如：
```javascript
let xhr=new XMLHttpRequest();
xhr.open('get','example.php',false);
```
以上代码会启动一个针对example.php的GET请求。这里需要注意两点：
* URL相对于执行代码的当前页面；
* 调用open()方法并不会真正发送请求，只是启动一个请求以备发送

### sned()方法
send()方法接收一个参数，即要作为请求主题发送的数据。如果不需要通过请求主题发送数据，则传入null。在发送数据收到响应后，响应的数据会自动填充XHR对象的属性，相关属性如下：
* responseText：作为响应主题被返回的文本；
* responseXML：如果响应的内容类型是"text/xml"或者"application/xml"，则这个属性中将保存包含着响应数据的XML DOM文档；
* status：响应HTTP状态；
* statusText：响应HTTP状态的说明。

在接收到响应后，第一步是检查 status 属性，以确定响应已经成功返回。一般来说，可以将 HTTP状态代码为 200作为成功的标志。此时，responseText 属性的内容已经就绪，而且在内容类型正确的情况下，responseXML也应该能够访问了。此外，状态代码为304 表示请求的资源并没有被修改，可以直接使用浏览器中缓存的版本。由于我们用XHR对象常常是为了异步传输，所以---可以检测 XHR 对象的readyState 属性，该属性表示请求/响应过程的当前活动阶段。这个属性可取的值如下：
* 0：未初始化。尚未调用 open()方法。 
* 1：启动。已经调用 open()方法，但尚未调用 send()方法。 
* 2：发送。已经调用 send()方法，但尚未接收到响应。 
* 3：接收。已经接收到部分响应数据。 
* 4：完成。已经接收到全部响应数据，而且已经可以在客户端使用了。
 
<font color=red>必须在调用 open()之前指定 onreadystatechange事件处理程序才能确保跨浏览器兼容性。</font>所以，实现代码如下：
```javascript
 var xhr = new XMLHttpRequest();
//必须在open之前指定onreadystatechange才能保证跨浏览器兼容性！！！！
xhr.onreadystatechange=function () {
    if(xhr.readyState==4){
        if((xhr.status>=200&&xhr.status<300)||xhr.status==304){
            alert(xhr.responseText);
        }else{
            alert('请求失败'+xhr.status);
        }                    
    }
};
```
所以利用以上代码就可以实现文件是上传成功还是失败。
## XMLHttpRequest2级的进度事件
XHR2有如下6个进度事件：
* loadstart：在接收到响应数据的第一个字节时触发。 
* progress：在接收响应期间持续不断地触发。 
* error：在请求发生错误时触发。 
* abort：在因为调用 abort()方法而终止连接时触发。 
* load：在接收到完整的响应数据时触发。 
* loadend：在通信完成或者触发 error、abort 或 load 事件后触发。

以上事件触发顺序如下：
laodstart => progress => error => abort/load => loadend
### 关于progress事件中上传与下载的事件
这里就是我遇到的坑，在《javascript高级程序设计》中并没有说清楚，让我感到困惑，在原书中581页这么描写：“onprogress事件处理程序会接收到一个 event 对象，其 target 属性是 XHR 对象，但包含着三个额外的属性：lengthComputable、position 和 totalSize。其中，lengthComputable 是一个表示进度信息是否可用的布尔值，position 表示已经接收的字节数，totalSize 表示根据Content-Length 响应头部确定的预期字节数。”而实际上表示<font color=red>总字节数的属性是total，表示已经传输的字节数是load属性。</font>
XHR2传输数据有一个progress事件，用来返回进度信息，它分成上传和下载两种情况，<font color=red>下载的progress事件属于XMLHttpRequest对象，而上传的progress事件属于XMLHttpRequest.upload对象</font>。
首先定义传输文件的函数：
```javascript
function percentFun(event){
    if (event.lengthComputable) {
        /*event.total是需要传输的总字节数，event.load是已经传输的字节数，如果event.lengthComputable!=true,则event.total=0*/       
　　　　 var percent = event.loaded / event.total;
　　}
}
```
如果需要显示进度，则分别如下：
```javascript
/*下载的进度*/
xhr.onprogress = percentFun;
/*上传的进度*/
xhr.upload.onprogress = percentFun;
```
基于以上几点，实现了上传进度显示，具体代码如下：
```javascript
addEvent(button,'click',function () {
    if(filesArray.length!=0){
        var data=new FormData();
        var i=0;
        while(i<filesLen){
            data.append('file'+i,filesArray[i]);
            i++;
        }
        var xhr = new XMLHttpRequest();
        //必须在open之前指定onreadystatechange才能保证跨浏览器兼容性！！！！
        xhr.onreadystatechange=function () {
            if(xhr.readyState==4){
                if((xhr.status>=200&&xhr.status<300)||xhr.status==304){
                    //console.log(xhr.responseText);
                }else{
                    alert('请求失败'+xhr.status);
                }                    
            }
        };
        xhr.upload.onprogress=function (event) {
            var e=event||window.event;
            var percentComplete = Math.ceil((e.loaded / e.total)*100);
            var progressFont=document.querySelectorAll('.progress-font');
            for(let i=0;i<filesArray.length;i++){
                ThumbnailArray[i].innerHTML=percentComplete +'%';
            }
        }
        /*上传完成后滞空数组，保证下次上传不会重复上传*/
        xhr.upload.onload=function () {
            filesArray=[];
            ThumbnailArray=[];
            alert('上传完成，数组置空');                
        }
        xhr.open('post','uploader.php',true); 
        xhr.send(data);
    }        
});
```
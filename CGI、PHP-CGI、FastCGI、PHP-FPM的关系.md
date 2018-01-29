###  CGI /Common Gateway Interface   

 CGI描述了服务器和请求处理程序之间传输数据的一种标准，与语言无关。  
 文档：https://www.rfc-editor.org/rfc/rfc3875.txt        
 示意图：![](https://github.com/huluzhang/doc/blob/master/img/CGI.png)

### FastCGI /Fast Common Gateway Interface   
 
 FastCGI是一种改善型扩展的CGI，相对于CGI日复一日的fork加载的机制，FastCGI改为常驻内存的机制，并增加了分布式运算和多扩展角色。
文档：https://fastcgi-archives.github.io/FastCGI_Specification.html  
示意图：![](https://github.com/huluzhang/doc/blob/master/img/FASTCGI.png)  

###  PHP-CGI    
 PHP自己的FastCGI的实现。只是实现，没有Master。

### PHP-FPM   
 替换了PHP-CGI 的大部分功能，演变成Master与Pool的关系，支持了平滑停止/启动及高级进程管理等，成功从绿林好汉升级成正规军。

通过上面的解释，那么我们就认为CGI/FastCGI 是标准，而PHP-CGI和PHP-FPM只是实现。

创建PHP扩展的扩展骨架
------------

>开发PHP扩展其实就是个模板填坑,首先我们要生成扩展的框架，当然你对扩展开发很熟悉的话可以忽略这个工具。`ext_skel`是PHP自带的一个生成骨架扩展的脚本，另外还有一种扩展骨架生成工具PECL_Gen。我们通过一个例子来讲解PHP的扩展的构建。使用`ext_skel`来生成扩展的骨架（模板），`ext_skel` 的使用请看工具的帮助命令。`php_knowledge`是我们扩展的名字。

     [root@iZ8 ~]# cd php-7.0.2/ext   
     [root@iZ8 ext]# ./ext_skel --extname=php_knowledge
     
配置config.m4文件
-----------

执行完`ext_skel`后会在ext目录生成扩展的骨架文件。

    [root@iZ8 ext]# cd php_knowledge
    [root@iZ8 php_knowledge]# ll
    total 32
    -rw-r--r-- 1 root root 2274 Jan 29 15:36 config.m4
    -rw-r--r-- 1 root root  345 Jan 29 15:36 config.w32
    -rw-r--r-- 1 root root   13 Jan 29 15:36 CREDITS
    -rw-r--r-- 1 root root    0 Jan 29 15:36 EXPERIMENTAL
    -rw-r--r-- 1 root root 5337 Jan 29 15:36 php_knowledge.c 
    -rw-r--r-- 1 root root  523 Jan 29 15:36 php_knowledge.php
    -rw-r--r-- 1 root root 2773 Jan 29 15:36 php_php_knowledge.h
    drwxr-xr-x 2 root root 4096 Jan 29 15:36 tests  

>查看一下目录，主要包含以下文件：
***`config.m4`***
>文件使用 GNU autoconf语法编写，UNIX构建系统配置,生成configure脚本、Makefile等。
***`config.w32`***
>window下面的系统配置。
***`php_knowledge.c`***
>扩展的主要源码文件，主要包含模块结构定义、INI条目、管理函数、用户空间函数和其它扩展所需的内容。
***`php_php_knowledge.h`***
>扩展的头文件，主要包含结构的指针定义、附加的宏、原型等。
***`php_knowledge.php`***
>扩展的验证脚本，主要用来验证扩展是否被成功地编译到PHP中。

>接下来我们要修改config.m4，做好编译前的配置，首先我们先拆解分析这个文件，需要关注几个函数（宏）：


----------


***`PHP_ARG_WITH（）`***

    PHP_ARG_WITH(php_knowledge, for php_knowledge support, 
    Make sure that the comment is aligned:                               
    [  --with-php_knowledge   Include php_knowledge support])
>选择时`configure`使用`--with-*`选项，需要依赖第三方的lib。方法有三个参数:
 1. 扩展的名称。
 2. configure运行显示的内容。
 3. configure --help时显示的内容。


----------


***`PHP_ARG_ENABLE（）`***

    PHP_ARG_ENABLE(php_knowledge, whether to enable php_knowledge support,  Make sure that the comment is aligned:         
    [  --enable-php_knowledge  Enable php_knowledge support]) 

>选择时`configure`使用`--enable-*`选项，不需要依赖第三方的lib。方法的三个参数如上。


----------


***`PHP_ADD_INCLUDE（）`***

     PHP_ADD_INCLUDE($PHP_KNOWLEDGE_DIR/include)

>扩展模块用到的头文件路径加到`CFLAGS`中。


----------


***`PHP_CHECK_LIBRARY（）`***

    LIBNAME=php_knowledge 
    LIBSYMBOL=php_knowledge 
    #PHP_CHECK_LIBRARY检查$PHP_KNOWLEDGE_DIR/$PHP_LIBDIR目录下的库是否存在LIBSYMBOL的定义。
    PHP_CHECK_LIBRARY($LIBNAME,$LIBSYMBOL, 
    #如果验证存在，则添加库文件。                   
    [ PHP_ADD_LIBRARY_WITH_PATH($LIBNAME, $PHP_KNOWLEDGE_DIR/$PHP_LIBDIR, PHP_KNOWLEDGE_SHARED_LIBADD) 
      AC_DEFINE(HAVE_PHP_KNOWLEDGELIB,1,[ ])                           
    ],[#如果不存在，则报错.                                              
      AC_MSG_ERROR([wrong php_knowledge lib version or lib not found])  ],[  -L$PHP_KNOWLEDGE_DIR/$PHP_LIBDIR -lm ])        

>`PHP_CHECK_LIBRARY（）`用于验证我们的共享对象是否包含一个已知函数或符号，有五个参数：
 1. 库的名称`LIBNAME`。
 2. 寻找的库函数`LIBSYMBOL`。
 3. 找到后采取的行动，用`PHP_ADD_LIBRARY_WITH_PATH()`构建所需库文件路径和库资源，主要是为解决同名库或者库不兼容。
 4. 没找到采取的行动，比如我们通过`AC_MSG_ERROR`报出错误信息。
 5. 用于指定额外的编译器和链接器标记，确保编译器知道在哪里可以找到共享对象。
 `PHP_ADD_LIBRARY_WITH_PATH（）`一共三个参数：
 1. 库名
 2. 路径。
 3. 存储信息的变量名字。


----------


***`PHP_NEW_EXTENSION（）`***

    PHP_NEW_EXTENSION(php_knowledge, php_knowledge.c, $ext_shared)

>声明了这个扩展的名称、需要的源文件名、此扩展的编译形式。一共有六个参数：
 1. 扩展的名称。
 2. 用于构建扩展的源或文件的列表。
 3. 扩展应该动态加载还是静态编译。`$ext_shared`变量会为这点设置适当de 值。
 4. 指定一个`SAPI类`，仅用于专门需要 `CGI` 或 `CLI SAPI` 的扩展。
 5. 指定构建时要加`CFLAGS` 的标志列表。
 6. 是否使用`$CXX` 代替`$CC` 。
 
>熟悉了`config.m4`的大体内容后我们来做一下修改，这个例子不需要依赖第三方所以用`PHP_ARG_ENABLE`方式，找到`PHP_ARG_ENABLE`并去掉前面的dnl注释：

    [root@iZ8 php_knowledge]# vi config.m4
    PHP_ARG_ENABLE(php_knowledge, whether to enable php_knowledge support,
    Make sure that the comment is aligned:
    [  --enable-php_knowledge           Enable php_knowledge support])

编写扩展函数
------

>下一步来实现扩展逻辑的主源`php_knowledge.c`，编写一个PHP函数`php_knowledge（）`用来输出欢迎信息和打印mint、rint的调用时间来看看有什么区别。

 >在`php_knowledge.c`中增加实现用来PHP调用的函数：

    PHP_FUNCTION(php_knowledge)
    {
        php_printf("Welcomme to PHP_KNOWLEDGE <br/>");
        php_printf("%d<br/>",minit_time);
        php_printf("%d<br/>",rinit_time);
        RETURN_TRUE;
    }
>获取mint的调用时间，修改`PHP_MINIT_FUNCTION`为：

    int minit_time;
    PHP_MINIT_FUNCTION(php_knowledge)
    {
             minit_time = time(NULL);
            /*REGISTER_INI_ENTRIES();*/
            return SUCCESS;
    }

>获取rinit的调用时间，修改`PHP_RINIT_FUNCTION`为：

    int rinit_time;
    PHP_RINIT_FUNCTION(php_knowledge)
    {       
            rinit_time = time(NULL);
            return SUCCESS;
    }

>修改`PHP_MINFO（）`使`phpinfo`有友好的信息，使用`php_info_print_table`绘制表格：

    PHP_MINFO_FUNCTION(php_knowledge)
    {
            php_info_print_table_start();
            php_info_print_table_header(2, "Support", "enabled");
            php_info_print_table_row(2, "Extension Version",PHP_PHP_KNOWLEDGE_VERSION);
            php_info_print_table_row(2, "Author","zhangminsong@cmcm.com");
            php_info_print_table_end();
            /* Remove comments if you have entries in php.ini
            DISPLAY_INI_ENTRIES();
            */
    }

>在`php_knowledge_functions`关联我们刚实现的C函数php_knowledge：

    const zend_function_entry php_knowledge_functions[] = {
            PHP_FE(confirm_php_knowledge_compiled,  NULL)
            PHP_FE(php_knowledge,  NULL)
            PHP_FE_END      /* Must be the last line in php_knowledge_functions[] */
    };

>保存`php_knowledge.c`，一个简单的扩展实例源码应该写完了，下一步该编译生成我们的模块，挂在PHP上来验证一下。

    [root@iZ8 php_knowledge]# /data/opt/php7/bin/php ./php_knowledge.php  
    Functions available in the test extension:
    confirm_php_knowledge_compiled
    php_knowledge
    Congratulations! You have successfully modified ext/php_knowledge/config.m4. Module php_knowledge is now compiled into PHP.
    [root@iZ8vb6lwwf2zunsijp58jgZ php_knowledge]# /data/opt/php7/bin/phpize 
    Configuring for:
    PHP Api Version:         20151012
    Zend Module Api No:      20151012
    Zend Extension Api No:   320151012
    [root@iZ8 php_knowledge]# ./configure --with-php-config=/data/opt/php7/bin/php-config
    checking for grep that handles long lines and -e... /bin/grep
    checking for egrep... /bin/grep -E
    ...
    [root@iZ8 php_knowledge]# make
    ...
    [root@iZ8 php_knowledge]# make install
    Installing shared extensions:     /data/opt/php7/lib/php/extensions/no-debug-non-zts-20151012/

>修改`php.ini` ,增加`extension = "php_knowledge.so"` 执行一下我们刚实现的扩展函数：

    [root@iZ8 php_knowledge]#/data/opt/php7/bin/php -r 'php_knowledge();'
    Welcomme to PHP_KNOWLEDGE <br/>1517307034<br/>1517307034<br/>

>重启fpm，打印一下phpinfo()，页面出现一下信息：

    Welcomme to PHP_KNOWLEDGE <br/>1517300371<br/>1517307092<br/>

![](https://github.com/huluzhang/doc/blob/master/img/php_knowledge_phpinfo.png)
>说明我们的扩展正常运行了。我们回过头来分析一下`php_knowledge.c`的结构，其实一部分是PHP的模块的生命周期所经历的一个流程实现，另一部分是我们用C实现的函数和PHP的关联：

       MINT->RINI->RSHUTDOWN->MSHUTDOWN
       PHP_FUNCTION->zend_function_entry-ZEND_GET_MODULE

   
>回过头来我们来拆解一下`php_knowledge.c`：

***`ZEND_DECLARE_MODULE_GLOBALS(php_knowledge)`***
 >模块中的全局变量 


----------


***`PHP_INI_BEGIN()和PHP_INI_END()`***

    PHP_INI_BEGIN()
    //注册自定义的ini指令,放这里夹着。
    STD_PHP_INI_ENTRY("php_knowledge.global_value","42", PHP_INI_ALL, OnUpdateLong, global_value, zend_php_knowledge_globals, php_knowledge_globals)
    PHP_INI_END()


----------


***`PHP_FUNCTION() `***

    PHP_FUNCTION(php_knowledge)
    {
        php_printf("Welcomme to PHP_KNOWLEDGE <br/>");
          //打印mint的时间和rinit的时间有什么差异
        php_printf("%d<br/>",minit_time);
        php_printf("%d<br/>",rinit_time);
        RETURN_TRUE;
    }
    PHP_FUNCTION(confirm_php_knowledge_compiled)
    {
            char *arg = NULL;
            int arg_len, len;
            char *strg;
            if (zend_parse_parameters(ZEND_NUM_ARGS() TSRMLS_CC, "s", &arg, &arg_len) == FAILURE) { return; }
            len = spprintf(&strg, 0, "Congratulations! You have successfully modified ext/%.78s/config.m4. Module %.78s is now compiled into PHP.", "php_knowledge", arg);
            RETURN_STRINGL(strg, len, 0);
    }
>定义宏函数，就是我们在PHP语言中用来的调取的函数名字。也可以写成`ZEND_FUNCTION()`，编译器会将此函数转换成如下函数：

    void zif_php_knowledg(int ht, zval *return_value, zval **return_value_ptr, zval *this_ptr, int return_value_used TSRMLS_DC){}

>为预防输出至用户层的函数与其他的同名函数间的命名冲突，输出函数的 C 符号用 `zif_` 作为前缀，一共有如下参数：

 1. `int ht` 用户实际传递参数的数量 一般用宏ZEND_NUM_ARGS()。
 2. `zval *return_value` PHP变量的指针，可填充返回值传递给用户。默认值是IS_NULL。访问宏RETVAL_*, RETURN_*。
 3. `zval **return_value_ptr`当返回引用时，PHP将其设为变量的指针。不建议返回引用。 	
 4. `zval *this_ptr` 假如这是一个方法调用，其指向存放 $this 对象的 PHP 变量。访问宏getThis()。
 5. `int return_value_used` 指示返回值是否会被调用者使用的标志。

***`zend_parse_parameters(int num_args TSRMLS_CC, char *type_spec, ...)`***
>函数负责读取用户从参数堆栈传递来参数，并将其适当地转换后放入局部 C 语言变量。如果用户传递的参数个数有误或类型不可被转换，函数会发出一个冗长的错误信息，并返回 FAILURE。参数如下：

 1. `num_args`获取的参数数目，一般会先用`ZEND_NUM_ARGS()`判断，为了线程安全等用 `TSRMLS_CC`传递线程上下文。
 2. `type_spec`是格式化字符串，参数字符表示其类型。

>`zend_parse_parameters()` 其类型如下：
 1. b->zend_bool
 2. l->long 
 3. d->double 
 4. s->char*, int
 5. h->HashTable*

***`static void php_php_knowledge_init_globals(){}`***

    static void php_php_knowledge_init_globals(zend_php_knowledge_globals *php_knowledge_globals)
    {
            php_knowledge_globals->global_value = 0;
            php_knowledge_globals->global_string = NULL;
    }
>模块初始化全局变量默认值。


----------


***`PHP_MINIT_FUNCTION(){}`***

    int minit_time;
    PHP_MINIT_FUNCTION(php_knowledge)
    {
            /* If you have INI entries, uncomment these lines
             minit_time = time(NULL);
             //注册ini变量
            REGISTER_INI_ENTRIES();
            */
            return SUCCESS;
    }

>php的MINIT，模块第一次加载时被调用，比如mod_php跟随apahe的启动，模块的初始化工作。


----------


***`PHP_MSHUTDOWN_FUNCTION(){}`***

    PHP_MSHUTDOWN_FUNCTION(php_knowledge){
    //销毁ini变量
    UNREGISTER_INI_ENTRIES();
    }
    模块关闭时调用，模块的销毁工作。
    PHP_RINIT_FUNCTION(){}
    int rinit_time;
    PHP_RINIT_FUNCTION(php_knowledge)
    {      
            rinit_time = time(NULL);
            return SUCCESS;
    }

>每次request请求前调用，请求的初始化。


----------


***`PHP_RSHUTDOWN_FUNCTION(php_knowledge){}`***

    PHP_RSHUTDOWN_FUNCTION(php_knowledge)
    {
            return SUCCESS;
    }

>每次request请求结束时调用，请求的销毁工作。


----------


***`PHP_MINFO_FUNCTION(php_knowledge){}`***

    PHP_MINFO_FUNCTION(php_knowledge)
    {       //输出html
            php_info_print_table_start();
            //绘制表格
            php_info_print_table_header(2, "Support", "enabled");
            php_info_print_table_row(2, "Extension Version",PHP_PHP_KNOWLEDGE_VERSION);
            php_info_print_table_row(2, "Author","zhangminsong@cmcm.com");
            php_info_print_table_end();
                   是否显示php.ini的配置信息
            /* Remove comments if you have entries in php.ini
               DISPLAY_INI_ENTRIES();
            */
    }

>phpinfo()输出扩展信息。


----------


***`const zend_function_entry php_knowledge_functions[] = {}***`

    const zend_function_entry php_knowledge_functions[] = {
            //PHP_FE()宏函数是对我们编写的扩展函数的声明
            PHP_FE(confirm_php_knowledge_compiled,  NULL)      
            PHP_FE(php_knowledge,NULL)
            PHP_FE_END
    };

>声明函数数组，提供给PHP使用，C扩展与PHP语言的纽带。通过`PHP_FE`关联C函数与php的函数


----------


***`zend_module_entry php_knowledge_module_entry = {};***`

    zend_module_entry php_knowledge_module_entry = {
            STANDARD_MODULE_HEADER,//结构的首
            "php_knowledge",//模块名字，随便写
            php_knowledge_functions,//导出函数
            PHP_MINIT(php_knowledge),
            PHP_MSHUTDOWN(php_knowledge),
            PHP_RINIT(php_knowledge),           
            PHP_RSHUTDOWN(php_knowledge),   
            PHP_MINFO(php_knowledge),
            PHP_PHP_KNOWLEDGE_VERSION,
            STANDARD_MODULE_PROPERTIES//结构的尾
    };

>模块结构，声明了生命周期模块、模块名及`phpinfo`打印时的`MINFO`


----------


***`ZEND_GET_MODULE()***`

    ZEND_GET_MODULE(php_knowledge)

>扔给zend


----------


>关于PHP的扩展就讲到这里了，只是讲了一点皮毛，希望能给大家一些启发。

> 本节参考：
> ext_skel http://php.net/manual/zh/internals2.buildsys.skeleton.php
> PECL_Gen http://pecl.php.net/package/PECL_Gen
>《GNU autoconf》http://www.gnu.org/software/autoconf/manual/
>《PHP官方手册》http://php.net/manual/zh/internals2.structure.php
>《用C/C++扩展你的PHP》http://www.laruence.com/2009/04/28/719.html
>《PHP扩展开发及内核应用》http://www.cunmou.com

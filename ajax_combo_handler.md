#Ajax Combo Handler（ajax 合并请求处理）
##缘起
在项目开发中，有同事提出了一个问题，前端页面加载过程中，建立了过多的ajax请求，且每次ajax请求都是一些少量数据，这严重影响到web页面加载性能。我在看到这个问题的时候，第一时间想到了淘宝cdn的combo服务。以前只是在看一些前端技术文章的时候，或多或少听到过combo服务的概念，于是我查阅了一些资料（[参考](http://yuiblog.com/blog/2008/07/16/combohandler/)），其实它主要的目的就是通过合并javascript脚本文件请求，减少客户端的请求连接数，实现提高性能。同时也搜到过css combo，其目的与javascript一致，就是为了减少css文件的请求数。也许是受到启发，我突然想到了ajax数据请求应该也可以实现“combo化”，姑且起个中文名：*ajax 合并请求处理*。接下来说说其具体的设计思路。
##规则设计
1. 按照combo的设计思路，ajax请求也可以实现combo服务。当前端有多个数据接口请求时，可以设计一种url格式（类似combo的“??”),告诉后台要请求的具体多个url；
2. 后台按照约定解析url，得到前端提交的实际url列表。然后分别调用对应url的处理程序。待所有数据获取到后，再按照约定的格式，返回给前端；
3. 前端收到后台的返回数据后，根据约定的格式，解析返回的数据，分别调用不同的处理逻辑。至此，完成了ajax的combo化请求。

##设计示例
###请求url格式
这里只是提供了一种方式，具体的细节，可以再根据实际情况优化。
```
http://api.xxx.com/v1/ajaxcombo??user/getUserInfo?userId=1,setting/getSystemConfig.

```
暂用??实现url的分隔，后面是具体的url请求列表，用逗号分隔。为了避免url里也有逗号，需要对url进行编码。
###返回数据格式
```
{code:0,combo:1,data:[{code:0,data:{name:"houyhea",sex:0,old:32},errMsg:""},{code:0,data:{param:1,page:2},errMsg:""}],errMsg:""}
```
1. code:返回的错误码；
2. combo:当前data是否是combo化数据。如果是，则data是combo请求对应的返回数据数组，其索引顺序根据url中的顺序一致。如果不是，则data就是实际的业务数据
3. data:相关数据。
4.errMsg：对应的错误信息描述。

##其他
1. ajax combo handler 有点类似与promise的when方法（一次执行多个请求，当所有请求都有返回时，触发promise的resolve）。两者的本质区别在于：promise本质还是建立多个连接；而ajax combo handler却是只有一个请求连接。
2. ajax combo handler除了可以合并数据获取请求外，还可以合并提交请求（post、put、delete）。后续还可以考虑利用请求url的顺序，让后台顺序执行各url对应的后台程序，实现一些意想不到的特殊功能（比如：事务、数据操作的链式写法等）。
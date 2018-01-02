在具体的业务中,我们需要处理:  

- 请求方法的判断 : req.method (判断请求方法)
- URL的路径解析 :
- URL中查询字符串解析
- Cookie的解析
- Basic认证
- 表单数据的解析
- 任意格式文件的上传处理
- session(会话)

###请求方法
常见GET.POST,另外有HEAD,DELETE,PUT,CONNECT  
HTTP_Parse模块在解析请求报文的时候,通过req.method获取请求方法.
###路径解析
HTTP_Parse模块解析路径为req.url.  
静态文件服务器,会根据路径去查找磁盘的文件,并响应给客户端.   
其他分发场景,根据路径选择控制器(controller),控制器触发对应的action
###查询字符串
node提供了queryString模块处理请求参数 
###cookie
Cookie的处理步骤:  

- 服务器向客户端发送Cookie
- 浏览器将Cookie保存
- 之后每次浏览器发送请求都会携带Cookie 

HTTP_Parser会将所有的报文字段解析到req.heades上,cookie可以通过req.headers.cookie来获取.    
响应头中添加cookie,通过Set-Cookie字段,示例:  
Set-Cookie: name=value; Path=/; Expires=Sun, 23-Apr-23 09:01:35 GMT; Domain=.domain.com;

- name=value是必须包含的部分,其余部分是可选参数
- path表示这个cookie影响到的路径,当前访问的路径不满足该匹配时候,浏览器不发送这个cookie
- Exprires和Max-Age用来告知浏览器这个Cookie何过期,如果不设置这个选项,在关闭浏览器的时候就会丢失这个Cookie. 如果设立过期时间,浏览器会把Cookie内容写到磁盘中并保存,下次打开是依然有效. Expires的值是个UTC格式的时间字符串(如果服务器端和客户端的时间不一致,可能会存在偏差),Max-Age告知浏览器Cookie多久后过期. 
- HttpOnly告知浏览器不允许通过document.cookie去更改这个cookie值,设置后,本次响应的cookie在doucment.cookie不可见.但是在后续请求中,依然会携带
- Secure. 对HTTP请求无效.作用于HTTPS,表示创建的Cookie只能在HTTPS连接中传递.

cookie的性能:  
1.减小cookie的大小,大多数的cookie并不需要每次都用上   
2.为静态文件配置不同的域名,静态文件服务器   
###Session
为了解决Cookie敏感数据的问题,Seesion应运而生,Session的数据只保留在服务器端,客户端无法修改.      
虽然将所有的数据放在Cookie中不可取,但是将口令放在Cookie中还是可以的.   
服务器端启用session,会约定一个键值作为session的口令,这个值可以随意约定.一旦服务器查到用户请求cookie中没有携带,会为其生成一个唯一值,并设定超时时间.示例:   

	var sessionList = {};
	var key = 'session_id';
	var EXPIRES = 20 * 60 * 1000;
	var generate = function () {
		var session = {};
		session.id = (new Date()).getTime() + Math.random();
		session.cookie = {
			expire: (new Date()).getTime() + EXPIRES
		};
		sessionList[session.id] = session;
		return session;
	};

每个请求到来的时候,先检查cookie中的口令于服务器端的数据,如果过期,就重新生成,如下: 

	function (req, res) {
		var id = req.cookies[key];
		if (!id) {
			req.session = generate();
		} else {
			var session = sessionList[id];
			if (session) {
				if (session.cookie.expire > (new Date()).getTime()) {
					// 更新超时时间
					session.cookie.expire = (new Date()).getTime() + EXPIRES;
					req.session = session;
				} else {
					// 超时了,删除旧的数据,重新生成
					delete sessions[id];
				req.session = generate();
				}
			} else {
				// 如果session过期或口令不对,重新生成session
				req.session = generate();
			}
		}
		handle(req, res);
	}

在响应给客户端设置新的值,以便下次请求的时候能够对应服务端的数据.
	var wirte = res.writeHead;
	// 重写响应对象的wirteHead方法
	res.writeHead = function () {
		var cookie = res.getHeader('Set-Cookie');
		var session = serialize('Set-Cookie',req.session.id);
		cookies = Array.isArray(cookies)?cookies.concat(session):[cookies,session];
		res.setHeader('Set-Cookie'.cookies);
		retrun wirte.apply(this,argument);
	}

###session与内存
上面的例子中,session数据都存储在内存中,如果用户过多,可能会接触到node内存限制的上限,频繁的垃圾回收也会引起性能问题. 此外,多个进程之间不能共享内存,所以session的数据无法共享.    
常见的方案是将session集中化,统一到集中的数据存储中,常见工具有Redis,Memcached等,通过这些高效的缓存,node进程无须在内部维护数据对象.优点:  

- node与缓存服务保持长连接,握手导致的延迟只会影响初始化
- 高速缓存直接在内存中进行数据的存储和访问
- 缓存服务通常与node进程运行在相同的机器上或者相同的机房里面,网络速度收到影响小

为了让session更加安全,对其进行加密.
###缓存
常见的缓存的规则:  

- 添加expires或者cache-control到报文头中
- 配置etags
- 让ajax可以缓存

通常,post.delete,put这类请求不做任何缓存,大多数缓存应用在get请求中.
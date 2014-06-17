---
layout: post
title: "基于C的Socket聊天服务器"
date: 2014-05-05 16:26
comments: true
categories: Tec
---

前段时间使用C借助libevent，glib实现了一个简单的可以单聊，群聊的服务器，今天因为需求有些改动所以又翻出来改了一下。果然一日不见如隔三秋，虽说是自己写的东西，但基本上已经忘得七七八八了。觉得有必要在这里记录一下，省得以后又悲剧。

##一，工具类

###1.libunp.a

用到的第一个库便是它，因为它是《UNIX网络编程》的示例代码的工具库...开头写的测试程序基本都是照着示例代码改来改去，自然也是用的一样的函数来实现。觉得对于一般的读写和各种包裹函数都是很有用的。具体的不用细说，还是认真翻书来得实在。

###2.GLib

####GHashTable

这里基本的数据结构如哈希表之类的使用了GLib来做主角，另外它的GString也很好用，可以很方便的初始化与格式化字符串，个人感觉比C风格的字符串要好用一些。

GLib哈希表支持各种不同的结构，如果觉得`int`,`string`不够用可以直接使用指针，对于我来说已经相当足够。

```c
GHashTable *user_to_bev_map = g_hash_table_new(g_direct_hash,g_direct_equal);
g_hash_table_insert(user_to_bev_map, GINT_TO_POINTER(u_id), GINT_TO_POINTER(bev));
struct bufferevent *bev = g_hash_table_lookup(user_to_bev_map,GINT_TO_POINTER(to_id));
```

`g_direct_hash`,`g_direct_equal`代表初始化的类型，详细的介绍如下:

>Hash values returned by hash_func are used to determine where keys are stored within the GHashTable data structure.

> The `g_direct_hash()`, `g_int_hash()`, `g_int64_hash()`, `g_double_hash()` and `g_str_hash()` functions are provided for some common types of keys.
  
>If hash_func is NULL, g_direct_hash() is used.

<!-- more -->

####GString

相比于C风格的字符串需要定长初始化，拼接赋值之类的，GString提供的字符串要人性化很多，还内置了长度的属性，可以很方便的调用。

```c
GString *sql = g_string_sized_new(0);
g_string_printf(sql,"INSERT INTO table (id, type) VALUES (%s, %s);",id,type);
g_string_erase(sql, 0, sql->len);
g_string_free(sql,1);
```

还可以很方便的重复使用，但不要忘记最后的释放，关于释放函数的第二个参数是这样说明的：

>If free_segment is TRUE it also frees the character data. If it's FALSE, the caller gains ownership of the buffer and must free it after use with g_free().

还有个常用的函数是`g_strsplit`，可以对字符进行分割,第三个参数表明需要分出几个来，0的话则一直切分到最后。

```c
	gchar **p = g_strsplit(line,",",0);
	dispatch_request(source_id->str, p[3], p[0], p[2], p[1], p[4]);
	g_strfreev(p);
```

当然，关于GLib还有很多有用的东西，用Dash下个文档慢慢翻着看一遍相信会很有收获的。

###3.Mysql

有服务的地方就有数据库，对于这种简易的小服务，Mysql是必不可少的。

针对Mysql封了三个简单的函数方便调用：

```c
	MYSQL *db_connect(char *url, char *user_name, char *pwd, char *table_name) {
		MYSQL *conn_ptr;
		conn_ptr = mysql_init(NULL);
		if (!conn_ptr) {
			printf("mysql_init failed\n");
			return NULL;
		}
		conn_ptr = mysql_real_connect(conn_ptr, url, user_name, pwd, table_name, 0, NULL, 0);
		mysql_set_character_set(conn_ptr,"utf8");
		if (conn_ptr) {
			return conn_ptr;
		} else {
			return NULL;
		}
	}
```

```c
	MYSQL_RES *db_query(MYSQL *conn_ptr, char *sql) {
		int res = mysql_query(conn_ptr, sql); //查询语句
		if (res) {
			printf("error:%s\n",mysql_error(conn_ptr));
			return NULL;
		} else {
			MYSQL_RES *res_ptr = mysql_store_result(conn_ptr);				//取出结果集
			printf("affected %lu rows\n",(unsigned long)mysql_affected_rows(conn_ptr));
			return res_ptr;
		}
	}
```

返回的`res_ptr`需要手动释放：

```c
	mysql_free_result(res);
```
还有一个关闭函数：

```c
	void db_close(MYSQL *connfd) {
		mysql_close(connfd);
	}
```
这样，在服务器起来时建立数据库的连接存为全局变量，每次直接拿来用就好了，下面是一个比较典型的使用场景：

```c
	GString *sql = g_string_sized_new(0);
	g_string_printf(sql,"INSERT INTO table (id, type) VALUES (%s, %s);",id,type);
	db_query(conn_ptr, sql->str);
	g_string_erase(sql, 0, sql->len);
	g_string_printf(sql,"SELECT uid FROM table WHERE pid = %s",target_id);
	printf("find users sql2:%s\n", sql->str);
	MYSQL_RES *res = db_query(conn_ptr, sql->str);
	MYSQL_ROW sqlrow;
	if (res) {
		while((sqlrow = mysql_fetch_row(res)))  {
			long to_id = strtol(sqlrow[0], NULL, 10);
			if (g_hash_table_contains(user_to_bev_map,GINT_TO_POINTER(to_id)) && strcmp(sqlrow[0],source_id)) {
				struct bufferevent *bev = g_hash_table_lookup(user_to_bev_map,GINT_TO_POINTER(to_id));
				evbuffer_add_printf ((struct evbuffer *)bufferevent_get_output(bev), "%s\n",send_msg->str);
			}
		}
	}
	mysql_free_result(res);
	g_string_free(sql,1);
	g_string_free(send_msg,1);
```

###4.libevent

####介绍

libevent是个好东西，有了它一般的数量级的连接都不在话下了。关于这种大数量的连接是有专门的话题来讨论的-[C10K](http://www.kegel.com/c10k.html)。

最开始尝试了多进程多线程，select阻塞之类的方法，最后才找到这里来，也算是按着故事的发展逻辑走了一遍符合剧情尿性吧...[这篇文章](http://daniel.haxx.se/docs/poll-vs-select.html)写得不错，比较有指导性。

关于libevent上手说不上难，狠下心来多读几遍它的[Fast portable non-blocking network programming with Libevent](http://www.wangafu.net/~nickm/libevent-book/)弄明白了个大概还是不成问题的。

它的优点是跨平台，可以针对不同的平台的阻塞实现相同的功能。对于我们来说只需要关心event这个东西就好了，至于是UNIX的select，Linux的epoll还是BSD的kqueue那是libevent的事情，它会在底层帮我们选择[libevent Documentation](http://monkey.org/~provos/libevent/doxygen-2.0.1/)。

####原理

原始的socket的连接是我们建立了连接，获得一个套接字，然后对这个套接字进行多路复用的读写。

而现在我们可以使用libevent提供的event将这个套接字包裹起来，针对这个event编写它特定的读写函数。因为libevent是事件驱动的，所以当读写缓冲区达到特定条件时便会自动调用我们事先定义好的函数进行逻辑处理。大大简化了编码人员的工作量，可以让我们将更多的精力集中到逻辑代码的编写上面来（恰恰是最无聊的部分...），所以这么看来，也算是对程序员傻瓜化了一下吧。

因为对event的读写涉及到缓冲区的东西，需要我们去按字节的读出来，这里libevent也很贴心的又帮我们简化了一下工作。除了event外还提供了[bufferevent](http://www.wangafu.net/~nickm/libevent-book/Ref6_bufferevent.html)，看名字便知道这是专门针对读写字符准备的。这是网站上对它的介绍：

>Most of the time, an application wants to perform some amount of data buffering in addition to just responding to events.
 
>When we want to write data, for example, the usual pattern runs something like:

>* Decide that we want to write some data to a connection;

>* Put that data in a buffer.Wait for the connection to become writable;

>* Write as much of the data as we can;

>* Remember how much we wrote, and if we still have more data to write, wait for the connection to become writable again.

>This buffered IO pattern is common enough that Libevent provides a generic mechanism for it. A "bufferevent" consists of an underlying transport (like a socket), a read buffer, and a write buffer. Instead of regular events, which give callbacks when the underlying transport is ready to be read or written, a bufferevent invokes its user-supplied callbacks when it has read or written enough data.

####使用

我们可以将新建的监听套接字绑定在event上帮我们处理后续的事件：

```c
	struct event_base *base;
	struct event *listener_event;
	serveListen(&listener);
	evutil_make_socket_nonblocking(listener);
	listener_event = event_new(base, listener, EV_READ|EV_PERSIST, do_accept, (void *)base);
	event_add(listener_event, NULL);
	event_base_dispatch(base);
```

当有事件进入时它会主动调用`do_accept`：

```c
void do_accept(evutil_socket_t listener, short event, void *arg)
{
	 struct event_base *base = arg;
    struct sockaddr_storage ss;
    socklen_t slen = sizeof(ss);
    int fd = accept(listener, (struct sockaddr*)&ss, &slen);
    if (fd < 0) {
        perror("accept");
    } else if (fd > FD_SETSIZE) {
        close(fd);
    } else {
        struct bufferevent *bev;
        evutil_make_socket_nonblocking(fd);
        bev = bufferevent_socket_new(base, fd, BEV_OPT_CLOSE_ON_FREE);
        bufferevent_setcb(bev, readcb, writecb, errorcb, GINT_TO_POINTER(fd));
        bufferevent_setwatermark(bev, EV_READ, 0, MAX_LINE);
        bufferevent_enable(bev, EV_READ|EV_WRITE);
    }
}
```

从上面代码可以看出，我们从`accept`中接收回一个套接字，并将该套接字绑定在一个`bufferevent`，设置好相应的读写函数与读写触发的水位线加入到`base`里便可以从容的等待事件的触发了。

关于水位线，这里也有段详细的描述：

>Every bufferevent has two data-related callbacks: a read callback and a write callback. By default, the read callback is called whenever any data is read from the underlying transport, and the write callback is called whenever enough data from the output buffer is emptied to the underlying transport. You can override the behavior of these functions by adjusting the read and write "watermarks" of the bufferevent.

>Every bufferevent has four watermarks:

>* `Read low-water mark`

>Whenever a read occurs that leaves the bufferevent’s input buffer at this level or higher, the bufferevent’s read callback is invoked. Defaults to 0, so that every read results in the read callback being invoked.

>* `Read high-water mark`

>If the bufferevent’s input buffer ever gets to this level, the bufferevent stops reading until enough data is drained from the input buffer to take us below it again. Defaults to unlimited, so that we never stop reading because of the size of the input buffer.

>* `Write low-water mark`

>Whenever a write occurs that takes us to this level or below, we invoke the write callback. Defaults to 0, so that a write callback is not invoked unless the output buffer is emptied.

>* `Write high-water mark`

>Not used by a bufferevent directly, this watermark can have special meaning when a bufferevent is used as the underlying transport of another bufferevent. See notes on filtering bufferevents below.

细读一遍还是蛮获益匪浅的，大体意思为通过水位线的设置来触发读写的回调函数。

* 对于读取水位，有读低水位与高水位，读低水位默认为0，即当buffer里数据量高于0时便会调用读回调，也就是一有数据便会回调，另一方面，当超过读的高水位时，buffer便会停止接受数据，这个值默认被置为`unlimited`，所以可以理解为永远不会停止接受数据。

* 对于写入水位，写的低水位表示当写出数据后buffer里剩余的数据量小于该水位时调用写函数，默认为0，即只有buffer被清空后该函数才会被回调。写的高水位比较特殊，一般情况下没有使用。

对于水位线的设置是通过下面的函数实现的

```c
void bufferevent_setwatermark(struct bufferevent *bufev, short events,size_t lowmark, size_t highmark);
ex:bufferevent_setwatermark(bev, EV_READ, 0, MAX_LINE);
```

在回调函数里这样获取数据：

```c
	struct evbuffer *input = bufferevent_get_input(bev);
	if ((line = evbuffer_readln(input, &n, EVBUFFER_EOL_LF))) {
		gchar **p;
		p = g_strsplit(line,",",0);
		dispatch_request(u_data->u_id->str, p[3], p[0], p[2], p[1], p[4]);
		g_strfreev(p);
	}
```

需要注意的是，提供的`evbuffer_readln`可以将接受到的一行数据自动去除`\n`，方便了我们的后期使用。

##二，实现思路

具体的思路因为暂时在需求上不是很复杂所以比较简单。

首先实现了`do_accept`函数，阻塞接收请求建立连接的`socket`，当有新的`socket`进来后使用`bufferevent`将其包装好，并设置好它的首次读写回调函数。

```c
struct bufferevent *bev;
evutil_make_socket_nonblocking(fd);
bev = bufferevent_socket_new(base, fd, BEV_OPT_CLOSE_ON_FREE);
bufferevent_setcb(bev, readcb, writecb, errorcb, GINT_TO_POINTER(fd));
bufferevent_setwatermark(bev, EV_READ, 0, MAX_LINE);
bufferevent_enable(bev, EV_READ|EV_WRITE);
```

首次客户端的通信用于标示此次通信的目的，当为登录时，进入`reg_client`将该用户注册至服务器，并修改其回调函数用于具体的逻辑处理。


































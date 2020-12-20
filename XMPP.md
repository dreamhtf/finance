# 第一章

### 1.5.1 通用属性

```xml
<stream:stream>	// 创建了一个XMPP流
    <iq type='get'>
        <query xmlns='jabber:iq:roster'/>
    </iq>
	<presence/>

	<message to='darcy@pemberley.lit'
		from='elizabaeth@longbourn.lit/ballroom' type='chat'>
		<body>I cannot talk of books in a ball-room; my head is always full of
			something else.
		</body>
	</message>

	<presence type='unavailable' />
</stream:stream>
```

// 通用属性： from/to/type/id

### 1.5.2 presence节（控制并报告实体的可访问性）

​	子元素：<show>/<status>/<priority>
​	<show>away/chat/dnd/xa</show> // 用来传达该用户离开、有意聊天、不希望被打扰、长期离开
​	负优先级不会接收到通过裸JID寻址方式传送过来的消息

1. 普通presence节

2. 扩展presence节：可以扩展，但没必要

3. 出席订阅：

   - 用户的服务器会自动地将出席信息广播给那些订阅该用户出席信息的联系人。类似地，用户从所有他已经进行出席订阅的联系人那里，接收到出席更新信息。

   - subscribe/unsubscribe：请求建立新的出席订阅或取消一个现有的订阅
     subscribed/unsubscribed：对这类请求的应答

4. 定向出席：通常用于多人聊天。定向出席节是一种直接发给另一个用户或其他某个实体的普通<presence>节。这些节可以用来向那些没有进行出席订阅（通常因为只是临时需要出席信息）的实体传达出席状态信息。

### 1.5.3 message节（一个实体向另一个实体发送消息）

1. 消息类型：

   type='chat/error/normal/groupchat/headline'

2. 消息内容：

   通过向<message>节中添加一个<thead>元素来创建线索，
   <thead>元素的内容是一个用来区分不同线索的唯一标识符。

### 1.5.4 IQ节

​	表示的是Info/Query（信息查询），它为XMPP通信提供请求与相迎机制。
​	它与HTTP协议的基本工作原理非常相似，允许获取和设置查询，与HTTP的get和post动作非常相似。

### 1.5.5 error节

- <error>子元素自身必须携带必要的type属性，取值：
- cancel：不重试该动作，因为它总会失败
- continue：通常代表一条警告信息，很少用
- modify：表示发送的数据需要一些修改才会被接受
- auth：通知实体在以某种方式进行身份验证之后重试该动作
- wait：报告服务器临时遇到问题，应该在稍后将原节原封不动地重新发送

## 1.6 连接生命周期

### 1.6.1 连接

​	在XMPP流存在之前，必须建立通往XMPP服务器的连接。

### 1.6.2 流的建立

​	一旦建立通往给定XMPP服务器的连接，XMPP流就启动了。通过向服务器发送起始元素<stream:stream> ，就可以打开XMPP流。服务器通过发送响应流的起始标记<stream:stream>进行响应。

### 1.6.3  身份验证

​	XMPP允许进行TLS（Transport Layer Security，传输层安全）加密，而且大多数客户端默认使用该功能。

### 1.6.4 连接断开

```xml
<persence type='unavailable'/>
</stream:stream>
```

# 第三章

## 3.5 建立连接

​	Strophe.Status.CONNECTED等等，有7个阶段

### 3.5.1 连接生命周期

### 3.5.2 创建连接

```js
Var conn = new Strophe.Connection("http://bosh.metajack.im:5280/xmpp-httpbind");
conn.connect("user@example.com", "mypassword", my_callback);
```

connectI()和disconnect()来启动和关闭与服务器的通信。

## 3.6 创建节

### 3.6.1 Strophe 构建器

```js
// 创建<presence/>节
var pres1 = new Strophe.Builder("presence");
// 创建<presence to='example.com'/>节
var pres2 = new Strophe.Builder("presence", {to: "example.com"});
```

Strophe为XMPP节的创建提供了四个全局别名：$build()、$msg()、$pres()、$iq()

简化写法：

```js
var pres1 = $build("presence");
var pres2 = $build("presence", {to: "example.com"});
var pres3 = $pres();
var pres4 = $pres({to: "example.com"});
```

c()和cnode()方法向XMPP节中添加新的子元素

```js
var stanza = $build("foo").c("bar").c("baz");
stanza.toString();
// 返回：<foo><bar><baz/></bar></foo>
var stanza = $build("foo").c("bar").up().c("baz");
// 返回：<foo><bar/><baz/></foo>
```

t()可以添加文本子元素

```js
var message = $msg({to: "darcy@pemberley.lit", type: "chat"})
	.c("body").t("How do you do?");
```

```xml
<!-- 这个构建器生成的XMPP节-->
<message to='darcy@pemberley.lit' type='chat'>
    <body>How do you do?</body>
</message>
```



```js
var iq = $iq({to: "pemberley.lit", type: "get", id: "discol"})
	.c("query", {xmlns: "http://jabber.org/protocol/disco#info"});
```

```xml
<!-- 这个构建器生成的XMPP节-->
<iq to='pemberley.lit' type='get' id='discol'>
	<query xmlns='http://jabber.org/protocol/disco#info'></query>
</iq>
```

up()：每当添加一个子元素时，构建器的当前元素就会变成这个新的子元素。如果希望在同一个元素上创建多个子元素，那么必须在每次调用c()之后在元素树中向上后退一级。可以使用up()来完成。

```js
var presence = $pres().c("show").t("away").up().c("status").t("Off to Meryton");
```

```xml
<!-- 这个构建器生成的XMPP节-->
<presence>
	<show>away</show>
    <status>Off to Meryton</status>
</presence>
```

attrs()：它携带一个属性集合。如果部分XMPP节是由其他的代码构建的，而且我们在通过连接将其发送出去之前需要再添加一些最终属性，就可以使用这个方法。

## 3.7 处理事件

### 3.7.1 添加和删除处理程序

​	可以使用addHandler() 来添加新的XMPP节处理程序，而使用deleteHandler()则可将其删除。

```js
var ref = conn.addHandler(my_handler_function, null, "message");
connection.deleteHandler(ref);
```

### 3.7.2 节匹配

addHandler()函数携带一个或多个参数。第一个参数适当接收到匹配的XMPP节时调用的函数。剩余的参数是匹配列表。

```js
addHandler: function(handler, ns, name, type, id, from){
    
}
```

```js
// 每当接收到位于urn:xmpp:ping命名空间下的子元素的IQ节时，都会调用my_ping_handler()函数，
// 因为type/id/from这些条件未被指定
conn.addHandler(my_ping_handler, "urn:xmpp:ping", "iq");
```

### 3.7.3 节处理程序函数

```js
function my_ping_handler(iq) {
    // 返回true，每当找到匹配节时，都会调用该处理函数
    // 返回false，调用之后就会被删除
    // 如果没有返回值，会返回undefined,跟return false一样
	return false;
}
```

### 3.7.4 处理Hello响应

```js
// hello.js
var Hello = {
    connection: null,
    start_time: null,

    log: function (msg) {
        $('#log').append("<p>" + msg + "</p>");
    },

    send_ping: function (to) {
        var ping = $iq({
            to: to,
            type: "get",
            id: "ping1"
        }).c("ping", {xmlns: "urn:xmpp:ping"});

        Hello.log("Sending ping to " + to + ".");
        Hello.start_time = (new Date()).getTime() - Hello.start_time;
        Hello.connection.send(ping);
    },

    handle_pong: function (iq) {
        var elapsed = (new Date()).getTime - Hello.start_time;
        Hello.log("Received pong from server in " + elapsed + "ms.");
        Hello.connection.disconnect();

        return false;
    }
};

$(document).ready(function () {
    $('#login_').dialog({
        autoOpen: true,
        draggable: false,
        modal: true,
        title: 'Connect to XMPP',
        buttons: {
            "Connect": function () {
                $(document).trigger('connect', {
                    jid: $('#jid').val(),
                    password: $('#password').val()
                });
                $('#password').val('');
                $(this).dialog('close');
            }
        }
    });
});

$(document).bind('connect', function (ev, data) {
    var conn = new Strophe.Connection("http://bosh.metajack.im:5280/xmpp-ttpbind");
    conn.connect(data.jid, data.password, function (status) {
        if (status === Strophe.Status.CONNECTED) {
            $(document).trigger('connected');
        } else if (status === Strophe.Status.DISCONNECTED) {
            $(document).trigger('disconnected');
        }
    });

    Hello.connection = conn;
});

$(document).bind('connected', function () {
    Hello.log("Connection established.");
    Hello.connection.addHandler(Hello.handle_pong, null, "iq", null, "ping1");
    var domain = Strophe.getDomainFromJid(Hello.connection.jid);
    Hello.send_ping(domain);
});

$(document).bind('disconnected', function () {
    Hello.log("Connection terminated.");

    Hello.connection = null;
})
```

# 第六章 与好友交谈：一对一聊天

## 6.2 Gab的设计

​	XMPP即时通信使用了大量的<message>节和<presence>节	

​	每当一个用户与另一个用户通信时都要发送<message>节

​	每当联系人上线、状态变为离开或者离线时，都会发送<presence>节。<persence>节通告用户的聊天可访问性。

### 6.2.1 出席

​	为了让A接收到来自B的出席更新信息，A必须首先订阅这些更新信息。此外，B必须批准A的订阅请求。大多数情况下，A向B发送订阅请求，A会自动批准来自B的订阅请求。

```xml
<!-- 下面的XMPP节用来从服务器那里请求Elizabeth的花名册 -->
<iq from='elizabeth@longbourn.lit/library' type='get' id='roster1'>
    <query xmlns='jabber:iq:roster'/>
</iq>

<!-- 她的服务器将响应如下内容-->
<iq to='elizabeth@longbourn.lit/library' type='result' id='roster1'>
	<query xmlns='jabber:iq:roster'>
    	<item jid='darcy@pemberley.lit' name='Mr.Darcy' subscription='both'/>
        <item jid='jane@longbourn.lit' name='Jane' subscription='both'/>
    </query>
</iq>
```

jid：联系人的地址	name：该联系人的指定别名	subscription：根据联系人的出席状态设置的。

如果双发都订阅，那么它的值为both；

如果只有Elizabeth订阅，那么该值为to；

如果Elizabeth没有订阅，Jane订阅了她的出席信息，那么该值为from；

一般而言，用户希望看到花名册中subscription值为both或to的联系人。

```js


// 将下面代码添加到connected事件处理程序中以检索花名册
$(document).bind('connected',fucntion () {
    var iq = $iq({type: 'get'}).c('query', {xmlns: 'jabber:iq:roster'});
	Gab.connection.sendIQ(iq, Gab.on_roster);
});

// 将on_roster()的实现添加到Gab对象中
on_roster: function (iq) {
    $(iq).find('item').each(function () {
    	var jid = $(this).attr('jid');
        var name = $(this).attr('name') || jid;
        var jid_id = Gab.jid_to_id(jid);
        var contact = $("<li id' " + jid_id + "'>" +
                       "<div class='roster-contact offline'>" +
                       "<div class='roster-name'>" +
                       name +
                       "</div><div class='roster-jid'>" +
                       jid +
                       "</div></div></li>");
        // insert_contact()函数用来让联系人列表保持正确的排序
        Gab.insert_contact(contact);
    });
}

// 使用联系人的裸JID的稍作变形的版本作为id
jid_to_id: function (jid) {
    return Strophe.getBareJidFromJid(jid)
    	.replace("@", "-")
    	.replace(".", "-");
}

// 将insert_contact()实现以及它的辅助函数presence_value()添加到Gab对象中
presence_value: function (elem) {
    if (elem.hasClass('online')) {
        return 2;
    } else if (elem.hasClass('away')) {
        return 1;
    }
    
    return 0;
},
    
insert_contact: function (elem) {
    var jid = elem.find('.roster-jid').text();
    var pres = Gab.presence_value(elem.find('.roster-contact'));
    var contacts = $('#roster-area li');
    
    if (contacts.length > 0) {
        var inserted = false;
        contacts.each(function () {
            var cmp_pres = Gab.presence_value($(this).find('.roster-contact'));
            var cmp_jid = $(this).find('.roster-jid').text();
            
            if (pres > cmp_pres) {
                $(this).before(elem);
                inserted = true;
                return false;
            } else {
                if (jid < cmp_jid) {
                    $(this).before(elem);
                    inserted = true;
                    return false;
                }
            }
        });
        if (!inserted) {
            $('#rester-area ul').append(elem);
        }
    } else {
            $('#roster-area ul').append(elem);
    }
}
// 如果运行当前状态的Gab程序，那么我们应该能够登录到服务器并会看到花名册显示出来并且已经排序
```

### 6.4.2 处理IQ

​	<iq>节时唯一需要有响应的XMPP节。每个IQ-get或IQ-set节都必须接收到相应的IQ-set或IQ-error节，类似GET或POST一样。

### 6.4.3 更新出席状态

​	为了不错过任何更新，我们总是应该在发送那些将要触发我们感兴趣的事件之前设置好合适的处理程序

```js
on_roster: function (iq) {
    $(iq).find('item').each(function () {
    	var jid = $(this).attr('jid');
        var name = $(this).attr('name') || jid;
        var jid_id = Gab.jid_to_id(jid);
        var contact = $("<li id' " + jid_id + "'>" +
                       "<div class='roster-contact offline'>" +
                       "<div class='roster-name'>" +
                       name +
                       "</div><div class='roster-jid'>" +
                       jid +
                       "</div></div></li>");
        // insert_contact()函数用来让联系人列表保持正确的排序
        Gab.insert_contact(contact);
        // 设置状态并发送初始状态
        Gab.connection.addHandler(Gab.on_presence, null, "presence");
        Gab.connection.send($pres());
    });
}
```


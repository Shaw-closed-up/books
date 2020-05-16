# Tornado 第七章：外部服务认证

第六章的例子像我们展示了如何使用安全cookies和tornado.web.authenticated装饰器来实现一个简单的用户验证表单。在本章中，我们将着眼于如何对第三方服务进行身份验证。流行的Web API，比如Facebbok和Twitter，使用OAuth协议安全验证某人的身份，同时允许他们的用户保持第三方应用访问他们个人信息的控制权。Tornado提供了一些Python mix-in来帮助开发者验证外部服务，既包括显式地支持流行服务，也包括通过通用的OAuth支持。在本章中，我们将探讨两个使用Tornado的auth模块的示例应用：一个连接Twitter，另一个连接Facebook。

## 7.1 Tornado的auth模块

作为一个Web应用开发者，你可能想让用户直接通过你的应用在Twitter上发表更新或读取最新的Facebook状态。大多数社交网络和单一登录的API为验证你应用中的用户提供了一个标准的流程。Tornado的auth模块为OpenID、OAuth、OAuth 2.0、Twitter、FriendFeed、Google OpenID、Facebook REST API和Facebook Graph API提供了相应的类。尽管你可以自己实现对于特定外部服务认证过程的处理，不过Tornado的auth模块为连接任何支持的服务开发应用提供了简单的工作流程。

### 7.1.1 认证流程

这些认证方法的工作流程虽然有一些轻微的不同，但对于大多数而言，都使用了authorize_redirect和get_authenticated_user方法。authorize_rediect方法用来将一个未授权用户重定向到外部服务的验证页面。在验证页面中，用户登录服务，并让你的应用拥有访问他账户的权限。通常情况下，你会在用户带着一个临时访问码返回你的应用时使用get_authenticated_user方法。调用get_authenticated_user方法会把授权跳转过程提供的临时凭证替换成属于用户的长期凭证。Twitter、Facebook、FriendFeed和Google的具体验证类提供了他们自己的函数来使API调用它们的服务。

### 7.1.2 异步请求

关于auth模块需要注意的一件事是它使用了Tornado的异步HTTP请求。正如我们在第五章所看到的，异步HTTP请求允许Tornado服务器在一个挂起的请求等待传出请求返回时处理传入的请求。

我们将简单的看下如何使用异步请求，然后在一个例子中使用它们进行深入。每个发起异步调用的处理方法必须在它前面加上@tornado.web.asynchronous装饰器。

## 7.2 示例：登录Twitter

让我们来看一个使用Twitter API验证用户的例子。这个应用将重定向一个没有登录的用户到Twitter的验证页面，提示用户输入用户名和密码。然后Twitter会将用户重定向到你在Twitter应用设置页指定的URL。

首先，你必须在Twitter注册一个新应用。如果你还没有应用，可以从[Twitter开发者网站](https://dev.twitter.com/)的"Create a new application"链接开始。一旦你创建了你的Twitter应用，你将被指定一个access token和一个secret来标识你在Twitter上的应用。你需要在本节下面代码的合适位置填充那些值。

现在让我们看看代码清单7-1中的代码。

代码清单7-1 查看Twitter时间轴：twitter.py

```
import tornado.web
import tornado.httpserver
import tornado.auth
import tornado.ioloop

class TwitterHandler(tornado.web.RequestHandler, tornado.auth.TwitterMixin):
    @tornado.web.asynchronous
    def get(self):
        oAuthToken = self.get_secure_cookie('oauth_token')
        oAuthSecret = self.get_secure_cookie('oauth_secret')
        userID = self.get_secure_cookie('user_id')

        if self.get_argument('oauth_token', None):
            self.get_authenticated_user(self.async_callback(self._twitter_on_auth))
            return

        elif oAuthToken and oAuthSecret:
            accessToken = {
                'key': oAuthToken,
                'secret': oAuthSecret
            }
            self.twitter_request('/users/show',
                access_token=accessToken,
                user_id=userID,
                callback=self.async_callback(self._twitter_on_user)
            )
            return

        self.authorize_redirect()

    def _twitter_on_auth(self, user):
        if not user:
            self.clear_all_cookies()
            raise tornado.web.HTTPError(500, 'Twitter authentication failed')

        self.set_secure_cookie('user_id', str(user['id']))
        self.set_secure_cookie('oauth_token', user['access_token']['key'])
        self.set_secure_cookie('oauth_secret', user['access_token']['secret'])

        self.redirect('/')

    def _twitter_on_user(self, user):
        if not user:
            self.clear_all_cookies()
            raise tornado.web.HTTPError(500, "Couldn't retrieve user information")

        self.render('home.html', user=user)

class LogoutHandler(tornado.web.RequestHandler):
    def get(self):
        self.clear_all_cookies()
        self.render('logout.html')

class Application(tornado.web.Application):
    def __init__(self):
        handlers = [
            (r'/', TwitterHandler),
            (r'/logout', LogoutHandler)
        ]

        settings = {
            'twitter_consumer_key': 'cWc3 ... d3yg',
            'twitter_consumer_secret': 'nEoT ... cCXB4',
            'cookie_secret': 'NTliOTY5NzJkYTVlMTU0OTAwMTdlNjgzMTA5M2U3OGQ5NDIxZmU3Mg==',
            'template_path': 'templates',
        }

        tornado.web.Application.__init__(self, handlers, **settings)

if __name__ == '__main__':
    app = Application()
    server = tornado.httpserver.HTTPServer(app)
    server.listen(8000)
    tornado.ioloop.IOLoop.instance().start()
```

代码清单7-2和7-3的模板文件应该被放在应用的templates目录下。

代码清单7-2 Twitter时间轴：home.html

```
<html>
    <head>
        <title>{{ user['name'] }} ({{ user['screen_name'] }}) on Twitter</title>
    </head>

    <body>
        <div>
            <a href="/logout">Sign out</a>
        </div>
        <div>
            <img src="{{ user['profile_image_url'] }}" style="float:left" />
            <h2>About @{{ user['screen_name'] }}</h2>
            <p style="clear:both"><em>{{ user['description'] }}</em></p>
        </div>
        <div>
            <ul>
                <li>{{ user['statuses_count'] }} tweets.</li>
                <li>{{ user['followers_count'] }} followers.</li>
                <li>Following {{ user['friends_count'] }} users.</li>
            </ul>
        </div>
        {% if 'status' in user %}
            <hr />
            <div>
                <p>
                    <strong>{{ user['screen_name'] }}</strong>
                    <em>on {{ ' '.join(user['status']['created_at'].split()[:2]) }}
                        at {{ user['status']['created_at'].split()[3] }}</em>
                </p>
                <p>{{ user['status']['text'] }}</p>
            </div>
        {% end %}
    </body>
</html>
```

代码清单7-3 Twitter时间轴：logout.html

```
<html>
    <head>
        <title>Tornadoes on Twitter</title>
    </head>

    <body>
        <div>
            <h2>You have successfully signed out.</h2>
            <a href="/">Sign in</a>
        </div>
    </body>
</html>
```

让我们分块进行分析，首先从twitter.py开始。在Application类的**init**方法中，你将注意到有两个新的键出现在设置字典中：twitter_consumer_key和twitter_consumer_secret。它们需要被设置为你的Twitter应用详细设置页面中列出的值。同样，你还会注意到我们声明了两个处理程序：TwitterHandler和LogoutHandler。让我们立刻看看这两个类吧。

TwitterHandler类包含我们应用逻辑的主要部分。有两件事情需要立刻引起我们的注意，其一是这个类继承自能给我们提供Twitter功能的tornado.auth.TwitterMixin类，其二是get方法使用了我们在[第五章](http://dockerpool.com/static/books/introduction_to_tornado_cn/ch5.html)中讨论的@tornado.web.asynchronous装饰器。现在让我们看看第一个异步调用：

```
if self.get_argument('oauth_token', None):
    self.get_authenticated_user(self.async_callback(self._twitter_on_auth))
    return
```

当一个用户请求我们应用的根目录时，我们首先检查请求是否包括一个oauth_token查询字符串参数。如果有，我们把这个请求看作是一个来自Twitter验证过程的回调。

然后，我们使用auth模块的get_authenticated方法把给我们的临时令牌换为用户的访问令牌。这个方法期待一个回调函数作为参数，在这里是self._teitter_on_auth方法。当到Twitter的API请求返回时，执行回调函数，我们在代码更靠下的地方对其进行了定义。

如果oauth_token参数没有被发现，我们继续测试是否之前已经看到过这个特定用户了。

```
elif oAuthToken and oAuthSecret:
    accessToken = {
        'key': oAuthToken,
        'secret': oAuthSecret
    }
    self.twitter_request('/users/show',
        access_token=accessToken,
        user_id=userID,
        callback=self.async_callback(self._twitter_on_user)
    )
    return
```

这段代码片段寻找我们应用在Twitter给定一个合法用户时设置的access_key和access_secret cookies。如何这个值被设置了，我们就用key和secret组装访问令牌，然后使用self.twitter_request方法来向Twitter API的/users/show发出请求。在这里，你会再一次看到异步回调函数，这次是我们稍后将要定义的self._twitter_on_user方法。

twitter_quest方法期待一个路径地址作为它的第一个参数，另外还有一些可选的关键字参数，如access_token、post_args和callback。access_token参数应该是一个字典，包括用户OAuth访问令牌的key键，和用户OAuth secret的secret键。

如果API调用使用了POST方法，请求参数需要绑定一个传递post_args参数的字典。查询字符串参数在方法调用时只需指定为一个额外的关键字参数。在/users/show API调用时，我们使用了HTTP GET请求，所以这里不需要post_args参数，而所需的user_id API参数被作为关键字参数传递进来。

如果上面我们讨论的情况都没有发生，这说明用户是首次访问我们的应用（或者已经注销或删除了cookies），此时我们想将其重定向到Twitter的验证页面。调用self.authorize_redirect()来完成这项工作。

```
def _twitter_on_auth(self, user):
    if not user:
        self.clear_all_cookies()
        raise tornado.web.HTTPError(500, 'Twitter authentication failed')

    self.set_secure_cookie('user_id', str(user['id']))
    self.set_secure_cookie('oauth_token', user['access_token']['key'])
    self.set_secure_cookie('oauth_secret', user['access_token']['secret'])

    self.redirect('/')
```

我们的Twitter请求的回调方法非常的直接。_twitter_on_auth使用一个user参数进行调用，这个参数是已授权用户的用户数据字典。我们的方法实现只需要验证我们接收到的用户是否合法，并设置应有的cookies。一旦cookies被设置好，我们将用户重定向到根目录，即我们之前谈论的发起请求到/users/show API方法。

```
def _twitter_on_user(self, user):
    if not user:
        self.clear_all_cookies()
        raise tornado.web.HTTPError(500, "Couldn't retrieve user information")

    self.render('home.html', user=user)
```

_twitter_on_user方法是我们在twitter_request方法中指定调用的回调函数。当Twitter响应用户的个人信息时，我们的回调函数使用响应的数据渲染home.html模板。这个模板展示了用户的个人图像、用户名、详细信息、一些关注和粉丝的统计信息以及用户最新的状态更新。

LogoutHandler方法只是清除了我们为应用用户存储的cookies。它渲染了logout.html模板，来给用户提供反馈，并跳转到Twitter验证页面允许其重新登录。就是这些！

我们刚才看到的Twitter应用只是为一个授权用户展示了用户信息，但它同时也说明了Tornado的auth模块是如何使开发社交应用更简单的。创建一个在Twitter上发表状态的应用作为一个练习留给读者。

## 7.3 示例：Facebook认证和Graph API

Facebook的这个例子在结构上和刚才看到的Twitter的例子非常相似。Facebook有两种不同的API标准，原始的REST API和Facebook Graph API。目前两种API都被支持，但Graph API被推荐作为开发新Facebook应用的方式。Tornado在auth模块中支持这两种API，但在这个例子中我们将关注Graph API。

为了开始这个例子，你需要登录到Facebook的[开发者网站](https://developers.facebook.com/)，并创建一个新的应用。你将需要填写应用的名称，并证明你不是一个机器人。为了从你自己的域名中验证用户，你还需要指定你应用的域名。然后点击"Select how your app integrates with Facbook"下的"Website"。同时你需要输入你网站的URL。要获得完整的创建Facebook应用的手册，可以从https://developers.facebook.com/docs/guides/web/开始。

你的应用建立好之后，你将使用基本设置页面的应用ID和secret来连接Facebook Graph API。

回想一下上一节的提到的单一登录工作流程，它将引导用户到Facebook平台验证应用，Facebook将使用一个HTTP重定向将一个带有验证码的用户返回给你的服务器。一旦你接收到含有这个认证码的请求，你必须请求用于标识API请求用户身份的验证令牌。

这个例子将渲染用户的时间轴，并允许用户通过我们的接口更新她的Facebook状态。让我们看下代码清单7-4。

代码清单7-4 Facebook验证：facebook.py

```
import tornado.web
import tornado.httpserver
import tornado.auth
import tornado.ioloop
import tornado.options
from datetime import datetime

class FeedHandler(tornado.web.RequestHandler, tornado.auth.FacebookGraphMixin):
    @tornado.web.asynchronous
    def get(self):
        accessToken = self.get_secure_cookie('access_token')
        if not accessToken:
            self.redirect('/auth/login')
            return

        self.facebook_request(
            "/me/feed",
            access_token=accessToken,
            callback=self.async_callback(self._on_facebook_user_feed))

    def _on_facebook_user_feed(self, response):
        name = self.get_secure_cookie('user_name')
        self.render('home.html', feed=response['data'] if response else [], name=name)

    @tornado.web.asynchronous
    def post(self):
        accessToken = self.get_secure_cookie('access_token')
        if not accessToken:
            self.redirect('/auth/login')

        userInput = self.get_argument('message')

        self.facebook_request(
            "/me/feed",
            post_args={'message': userInput},
            access_token=accessToken,
            callback=self.async_callback(self._on_facebook_post_status))

    def _on_facebook_post_status(self, response):
        self.redirect('/')

class LoginHandler(tornado.web.RequestHandler, tornado.auth.FacebookGraphMixin):
    @tornado.web.asynchronous
    def get(self):
        userID = self.get_secure_cookie('user_id')

        if self.get_argument('code', None):
            self.get_authenticated_user(
                redirect_uri='http://example.com/auth/login',
                client_id=self.settings['facebook_api_key'],
                client_secret=self.settings['facebook_secret'],
                code=self.get_argument('code'),
                callback=self.async_callback(self._on_facebook_login))
            return
        elif self.get_secure_cookie('access_token'):
            self.redirect('/')
            return

        self.authorize_redirect(
            redirect_uri='http://example.com/auth/login',
            client_id=self.settings['facebook_api_key'],
            extra_params={'scope': 'read_stream,publish_stream'}
        )

    def _on_facebook_login(self, user):
        if not user:
            self.clear_all_cookies()
            raise tornado.web.HTTPError(500, 'Facebook authentication failed')

        self.set_secure_cookie('user_id', str(user['id']))
        self.set_secure_cookie('user_name', str(user['name']))
        self.set_secure_cookie('access_token', str(user['access_token']))
        self.redirect('/')

class LogoutHandler(tornado.web.RequestHandler):
    def get(self):
        self.clear_all_cookies()
        self.render('logout.html')

class FeedListItem(tornado.web.UIModule):
    def render(self, statusItem):
        dateFormatter = lambda x: datetime.
strptime(x,'%Y-%m-%dT%H:%M:%S+0000').strftime('%c')
        return self.render_string('entry.html', item=statusItem, format=dateFormatter)

class Application(tornado.web.Application):
    def __init__(self):
        handlers = [
            (r'/', FeedHandler),
            (r'/auth/login', LoginHandler),
                (r'/auth/logout', LogoutHandler)
            ]

            settings = {
                'facebook_api_key': '2040 ... 8759',
                'facebook_secret': 'eae0 ... 2f08',
                'cookie_secret': 'NTliOTY5NzJkYTVlMTU0OTAwMTdlNjgzMTA5M2U3OGQ5NDIxZmU3Mg==',
                'template_path': 'templates',
                'ui_modules': {'FeedListItem': FeedListItem}
            }

            tornado.web.Application.__init__(self, handlers, **settings)

    if __name__ == '__main__':
        tornado.options.parse_command_line()

        app = Application()
        server = tornado.httpserver.HTTPServer(app)
        server.listen(8000)
        tornado.ioloop.IOLoop.instance().start()
```

我们将按照访客与应用交互的顺序来讲解这些处理。当请求根URL时，FeedHandler将寻找access_token cookie。如果这个cookie不存在，用户会被重定向到/auth/login URL。

登录页面使用了authorize_redirect方法来讲用户重定向到Facebook的验证对话框，如果需要的话，用户在这里登录Facebook，审查应用程序请求的权限，并批准应用。在点击"Approve"之后，她将被跳转回应用在authorize_redirect调用中redirect_uri指定的URL。

当从Facebook验证页面返回后，到/auth/login的请求将包括一个code参数作为查询字符串参数。这个码是一个用于换取永久凭证的临时令牌。如果发现了code参数，应用将发出一个Facebook Graph API请求来取得认证的用户，并存储她的用户ID、全名和访问令牌，以便在应用发起Graph API调用时标识该用户。

存储了这些值之后，用户被重定向到根URL。用户这次回到根页面时，将取得最新Facebook消息列表。应用查看access_cookie是否被设置，并使用facebook_request方法向Graph API请求用户订阅。我们把OAuth令牌传递给facebook_request方法，此外，这个方法还需要一个回调函数参数--在代码清单7-4中，它是_on_facebook_user_feed方法。

代码清单7-5 Facebook验证：home.html

```
<html>
    <head>
        <title>{{ name }} on Facebook</title>
    </head>

    <body>
        <div>
            <a href="/auth/logout">Sign out</a>
            <h1>{{ name }}</h1>
        </div>
        <div>
            <form action="/facebook/" method="POST">
                <textarea rows="3" cols="50" name="message"></textarea>
                <input type="submit" value="Update Status" />
            </form>
        </div>
        <hr />
        {% for item in feed %}
            {% module FeedListItem(item) %}
        {% end %}
    </body>
</html>
```

当包含来自Facebook的用户订阅消息的响应的回调函数被调用时，应用渲染home.html模板，其中使用了FeedListItem这个UI模块来渲染列表中的每个条目。在模板开始处，我们渲染了一个表单，可以用message参数post到我们服务器的/resource。应用发送这个调用给Graph API来发表一个更新。

为了发表更新，我们再次使用了facebook_request方法。这次，除了access_token参数之外，我们还包括了一个post_args参数，这个参数是一个成为Graph请求post主体的参数字典。当调用成功时，我们将用户重定向回首页，并请求更新后的时间轴。

正如你所看到的，Tornado的auth模块提供的Facebook验证类包括很多构建Facebook应用时非常有用的功能。这不仅在原型设计中是一笔巨大的财富，同时也非常适合是生产中的应用。
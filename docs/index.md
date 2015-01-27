<p class="badges" height=20px>
<iframe src="http://ghbtns.com/github-btn.html?user=tomchristie&amp;repo=django-rest-framework&amp;type=watch&amp;count=true" class="github-star-button" allowtransparency="true" frameborder="0" scrolling="0" width="110px" height="20px"></iframe>

<a href="https://twitter.com/share" class="twitter-share-button" data-url="django-rest-framework.org" data-text="Checking out the totally awesome Django REST framework! http://www.django-rest-framework.org" data-count="none"></a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="http://platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>

<img src="https://secure.travis-ci.org/tomchristie/django-rest-framework.svg?branch=master" class="travis-build-image">
</p>

---

**注意**: 这篇文档针对的是**3.0版本**的REST framework. [2.4版本](http://tomchristie.github.io/rest-framework-2-docs/) 的文档依然有效.

更多的细节请参考[3.0发布说明][3.0-announcement].

---

<p>
<h1 style="position: absolute;
    width: 1px;
    height: 1px;
    padding: 0;
    margin: -1px;
    overflow: hidden;
    clip: rect(0,0,0,0);
    border: 0;">Django REST Framework</h1>

<img alt="Django REST Framework" title="Logo by Jake 'Sid' Smith" src="img/logo.png" width="600px" style="display: block; margin: 0 auto 0 auto">
</p>


Django REST framework是一个功能强大、扩展性非常好的工具库，它可以让你很轻松的来构建Web API.

下面可能是你想要使用REST framework的一些原因:

* [Web预览API][sandbox] 对开发人员有巨大的帮助.
* [认证策略][authentication]，包含[OAuth1a][oauth1-section]和[OAuth2][oauth2-section]的开箱即用.
* [Serialization][serializers]同时支持[ORM][modelserializer-section]和[non-ORM][serializer-section]数据源.
* 可针对所有场景进行定制 - 如果你不需要[更多][generic-views] [功能强大的][viewsets] [特性][routers]，可以仅仅使用[标准的 基于函数的 views][functionview-section].
* [大量的文档][index], 以及[强大的社区支持][group].
* 很多大公司信任而且正在使用,例如[Mozilla][mozilla]和[Eventbrite][eventbrite].

---

![Screenshot][image]

**上图**: *Web预览API的截图*

## 必要的依赖项

REST framework 需要以下支持:

* Python (2.6.5+, 2.7, 3.2, 3.3, 3.4)
* Django (1.4.11+, 1.5.6+, 1.6.3+, 1.7)

下列的包是可选的:

* [Markdown][markdown] (2.1.0+) - 对Web预览API的Markdown语法支持.
* [PyYAML][yaml] (3.10+) - YAML格式内容支持.
* [defusedxml][defusedxml] (0.3+) - XML格式内容支持.
* [django-filter][django-filter] (0.9.2+) - 过滤器支持.
* [django-oauth-plus][django-oauth-plus] (2.0+)和[oauth2][oauth2] (1.5.211+) - OAuth 1.0a支持.
* [django-oauth2-provider][django-oauth2-provider] (0.2.3+) - OAuth 2.0支持.
* [django-guardian][django-guardian] (1.1.1+) - 对象级别的权限支持.

**注意**: Python包`oauth2`严重的名不副实, 其实提供的是OAuth 1.0a的支持. 还要注意的是这个包同时需要包OAuth 1.0a, 而OAuth 2.0 还为与Python 3兼容.

## 安装

使用`pip`命令安装, 包含你想要的任意可选包...

    pip install djangorestframework
    pip install markdown       # 对Web预览API的Markdown语法支持.
    pip install django-filter  # 过滤器支持

...或者从github上克隆一个项目.

    git clone git@github.com:tomchristie/django-rest-framework.git

然后在你的项目配置文件中的 `INSTALLED_APPS`里配置`'rest_framework'`.

    INSTALLED_APPS = (
        ...
        'rest_framework',
    )

如果你打算使用Web预览API，很可能还需要添加REST framework的登陆和登出views.  在你的`urls.py`文件中添加下面内容.

    urlpatterns = [
        ...
        url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
    ]

注意，URL路径随意你配置, 但是你必须通过`'rest_framework'`命名空间引入`'rest_framework.urls'`.

## 示例

让我们看看如何使用REST framework快速的构建一个简单的模型支持(model-backed)API.

我们会创建一个读写API，用来访问我们项目里面的users的信息.

REST framework API的所有的全局配置都在一个叫做`REST_FRAMEWORK`的单独的词典中.  从将下面添加到你的`settings.py`模块中开始:

    REST_FRAMEWORK = {
        # 使用Django标准的`django.contrib.auth`权限,
        # 或者允许将只读权限给未鉴权的用户.
        'DEFAULT_PERMISSION_CLASSES': [
            'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'
        ]
    }

别忘记确认一下你已经将`rest_framework`添加到了`INSTALLED_APPS`中.

现在，我们已经做好了创建API的准备.
下面是我们的项目根目录里的`urls.py`模块:

	from django.conf.urls import url, include
	from django.contrib.auth.models import User
	from rest_framework import routers, serializers, viewsets
	
	# 序列化器(Serializers)定义了API的表现形式.
	class UserSerializer(serializers.HyperlinkedModelSerializer):
	    class Meta:
	        model = User
	        fields = ('url', 'username', 'email', 'is_staff')
	
	# 视图集(ViewSets)定义了视图(view)的行为.
	class UserViewSet(viewsets.ModelViewSet):
	    queryset = User.objects.all()
	    serializer_class = UserSerializer
	
	# 路由器(Routers)提供了一个简单的方式来自动适配URL配置.
	router = routers.DefaultRouter()
	router.register(r'users', UserViewSet)
	
	# 让我们的API使用自动URL路由.
	# 此外, 我们也添加了Web预览API的登陆URL.
	urlpatterns = [
	    url(r'^', include(router.urls)),
	    url(r'^api-auth/', include('rest_framework.urls', namespace='rest_framework'))
	]

现在你可以在浏览器中打开这个API [http://127.0.0.1:8000/](http://127.0.0.1:8000/), 浏览你新建立的'users' API. 如果你使用了位于右上角的登陆控件进行了登陆，你就可以从系统中添加和删除user.

## 快速开始

现在你已经等不及的想开始了吧? 这份[快速开始指南][quickstart]会帮助你最快速的学会使用REST framework来构建并运行你的API.

## 教程

本教程会引导你一步一步的完成构建REST framework. 强烈建议阅读，虽然这需要耗费一些时间，但是它可以帮你全面了解每个特性是如何配合在一起工作的.

* [1 - 序列化(Serialization)][tut-1]
* [2 - 请求和响应(Requests & Responses)][tut-2]
* [3 - 基于类的视图(Class based views)][tut-3]
* [4 - 认证和权限(Authentication & permissions)][tut-4]
* [5 - 关系和超链API(Relationships & hyperlinked APIs)][tut-5]
* [6 - 视图集和路由器(Viewsets & routers)][tut-6]

这里有一个用来测试目的完成的一个活生生的API例子, [猛击这里][sandbox].

## API指南

这份API指南是给你参考的完整手册，包含REST framework提供的所有功能.

* [Requests][request]
* [Responses][response]
* [Views][views]
* [Generic views][generic-views]
* [Viewsets][viewsets]
* [Routers][routers]
* [Parsers][parsers]
* [Renderers][renderers]
* [Serializers][serializers]
* [Serializer fields][fields]
* [Serializer relations][relations]
* [Validators][validators]
* [Authentication][authentication]
* [Permissions][permissions]
* [Throttling][throttling]
* [Filtering][filtering]
* [Pagination][pagination]
* [Content negotiation][contentnegotiation]
* [Metadata][metadata]
* [Format suffixes][formatsuffixes]
* [Returning URLs][reverse]
* [Exceptions][exceptions]
* [Status codes][status]
* [Testing][testing]
* [Settings][settings]

## 主题

使用REST framework的通用指南.

* [创建你的API文档][documenting-your-api]
* [AJAX, CSRF & CORS][ajax-csrf-cors]
* [浏览器功能增强][browser-enhancements]
* [Web预览API][browsableapi]
* [REST, Hypermedia & HATEOAS][rest-hypermedia-hateoas]
* [第三方资源][third-party-resources]
* [为REST framework做贡献][contributing]
* [项目管理][project-management]
* [2.0 发布公告][rest-framework-2-announcement]
* [2.2 发布公告][2.2-announcement]
* [2.3 发布公告][2.3-announcement]
* [2.4 发布公告][2.4-announcement]
* [3.0 发布公告][3.0-announcement]
* [Kickstarter 公告][kickstarter-announcement]
* [发布说明][release-notes]
* [Credits][credits]

## 开发

请参考[贡献准则][contributing]提供的信息，了解如何克隆REST Framework的资源，运行测试以及贡献修正.

## 支持

请参考[REST framework讨论组][group], 尝试位于`irc.freenode.net`上的`#restframework`频道, 查找[IRC档案][botbot], 或者直接在[Stack Overflow][stack-overflow]上提出问题, 确保将该问题添加到标签['django-rest-framework'][django-rest-framework-tag]下.

来自[DabApps][dabapps]的[付费支持是可用的][paid-support], 包含REST framework核心的工作, 或者为构建你自己的REST framework API提供支持.  如果你想讨论商业支持方面的话题，请[联系DabApps][contact-dabapps].

想随时了解最新的REST framework开发信息，你可以关注[作者][twitter]的推特.

<a style="padding-top: 10px" href="https://twitter.com/_tomchristie" class="twitter-follow-button" data-show-count="false">关注 @_tomchristie</a>
<script>!function(d,s,id){var js,fjs=d.getElementsByTagName(s)[0];if(!d.getElementById(id)){js=d.createElement(s);js.id=id;js.src="//platform.twitter.com/widgets.js";fjs.parentNode.insertBefore(js,fjs);}}(document,"script","twitter-wjs");</script>

## 安全

如果你确信你在Django REST framework中找到了一些安全隐患, 请**不要在公共场合公开这些问题**.

请通过email发送问题的描述给[rest-framework-security@googlegroups.com][security-mail].  必要时，在公布之前，项目的维护者会和你一起来解决这些问题.

## 许可

Copyright (c) 2011-2015, Tom Christie
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

Redistributions of source code must retain the above copyright notice, this
list of conditions and the following disclaimer.
Redistributions in binary form must reproduce the above copyright notice, this
list of conditions and the following disclaimer in the documentation and/or
other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

[travis]: http://travis-ci.org/tomchristie/django-rest-framework?branch=master
[travis-build-image]: https://secure.travis-ci.org/tomchristie/django-rest-framework.png?branch=master
[mozilla]: http://www.mozilla.org/en-US/about/
[eventbrite]: https://www.eventbrite.co.uk/about/
[markdown]: http://pypi.python.org/pypi/Markdown/
[yaml]: http://pypi.python.org/pypi/PyYAML
[defusedxml]: https://pypi.python.org/pypi/defusedxml
[django-filter]: http://pypi.python.org/pypi/django-filter
[oauth2]: https://github.com/simplegeo/python-oauth2
[django-oauth-plus]: https://bitbucket.org/david/django-oauth-plus/wiki/Home
[django-oauth2-provider]: https://github.com/caffeinehit/django-oauth2-provider
[django-guardian]: https://github.com/lukaszb/django-guardian
[0.4]: https://github.com/tomchristie/django-rest-framework/tree/0.4.X
[image]: img/quickstart.png
[index]: .
[oauth1-section]: api-guide/authentication#oauthauthentication
[oauth2-section]: api-guide/authentication#oauth2authentication
[serializer-section]: api-guide/serializers#serializers
[modelserializer-section]: api-guide/serializers#modelserializer
[functionview-section]: api-guide/views#function-based-views
[sandbox]: http://restframework.herokuapp.com/

[quickstart]: tutorial/quickstart.md
[tut-1]: tutorial/1-serialization.md
[tut-2]: tutorial/2-requests-and-responses.md
[tut-3]: tutorial/3-class-based-views.md
[tut-4]: tutorial/4-authentication-and-permissions.md
[tut-5]: tutorial/5-relationships-and-hyperlinked-apis.md
[tut-6]: tutorial/6-viewsets-and-routers.md

[request]: api-guide/requests.md
[response]: api-guide/responses.md
[views]: api-guide/views.md
[generic-views]: api-guide/generic-views.md
[viewsets]: api-guide/viewsets.md
[routers]: api-guide/routers.md
[parsers]: api-guide/parsers.md
[renderers]: api-guide/renderers.md
[serializers]: api-guide/serializers.md
[fields]: api-guide/fields.md
[relations]: api-guide/relations.md
[validators]: api-guide/validators.md
[authentication]: api-guide/authentication.md
[permissions]: api-guide/permissions.md
[throttling]: api-guide/throttling.md
[filtering]: api-guide/filtering.md
[pagination]: api-guide/pagination.md
[contentnegotiation]: api-guide/content-negotiation.md
[metadata]: api-guide/metadata.md
[formatsuffixes]: api-guide/format-suffixes.md
[reverse]: api-guide/reverse.md
[exceptions]: api-guide/exceptions.md
[status]: api-guide/status-codes.md
[testing]: api-guide/testing.md
[settings]: api-guide/settings.md

[documenting-your-api]: topics/documenting-your-api.md
[ajax-csrf-cors]: topics/ajax-csrf-cors.md
[browser-enhancements]: topics/browser-enhancements.md
[browsableapi]: topics/browsable-api.md
[rest-hypermedia-hateoas]: topics/rest-hypermedia-hateoas.md
[contributing]: topics/contributing.md
[project-management]: topics/project-management.md
[third-party-resources]: topics/third-party-resources.md
[rest-framework-2-announcement]: topics/rest-framework-2-announcement.md
[2.2-announcement]: topics/2.2-announcement.md
[2.3-announcement]: topics/2.3-announcement.md
[2.4-announcement]: topics/2.4-announcement.md
[3.0-announcement]: topics/3.0-announcement.md
[kickstarter-announcement]: topics/kickstarter-announcement.md
[release-notes]: topics/release-notes.md
[credits]: topics/credits.md

[tox]: http://testrun.org/tox/latest/

[group]: https://groups.google.com/forum/?fromgroups#!forum/django-rest-framework
[botbot]: https://botbot.me/freenode/restframework/
[stack-overflow]: http://stackoverflow.com/
[django-rest-framework-tag]: http://stackoverflow.com/questions/tagged/django-rest-framework
[django-tag]: http://stackoverflow.com/questions/tagged/django
[security-mail]: mailto:rest-framework-security@googlegroups.com
[paid-support]: http://dabapps.com/services/build/api-development/
[dabapps]: http://dabapps.com
[contact-dabapps]: http://dabapps.com/contact/
[twitter]: https://twitter.com/_tomchristie

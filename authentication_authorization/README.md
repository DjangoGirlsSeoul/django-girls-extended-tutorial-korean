# 숙제: 당신의 웹사이트에 보안 추가하기

지금까지 우리는 관리자 화면을 사용하는 경우를 제외하고 당신이 설정한 비밀번호를 사용하지 않았음을 눈치챘을 수도 있습니다. 당신의 블로그에 포스트를 아무나 새로 작성하거나 수정할 수 있다는 것 또한 눈치챘을 수도 있어요. 내 블로그에 아무나 (당신 대해 잘 모르지만..) 글을 쓰는 걸 원하지 않아! 그래서 아무나 글을 쓰지 못하도록 뭔가를 할 수 있어요.

## 포스트의 새로 작성/편집 허가하기

첫번째로 보안을 위해 무언가를 할거에요. 우리는 로그인 한 사용자만 포스트를 접근할 수 있도록 `post_new`, `post_edit`, `post_draft_list`, `post_remove` and `post_publish`의 뷰들을 보호할 거에요. 장고는 고급 주제중 하나 *decorators*를 사용하여 몇 가지의 유용한 헬퍼들을 제공합니다. 지금 기술적인 것에 걱정하지 마세요. 이것들은 나중에 이해하게 될 거에요. 장고에서 제공하는 `django.contrib.auth.decorators`모듈 안의 decorator를 사용고, 이것을 `login_required`라고 부릅니다..

그리고, 우리는 `blog/views.py`을 수정하고 파일 위쪽의 import를 하는 영역에 아래와 같이 한 줄을 추가합니다.

```python
from django.contrib.auth.decorators import login_required
```

그리고, 각각의 `post_new`, `post_edit`, `post_draft_list`, `post_remove` and `post_publish` 뷰의 앞에 아래와 같이 줄을 추가합니다.

```python
@login_required
def post_new(request):
    [...]
```

잘했어요, 이제 `http://localhost:8000/post/new/`로 접속해보아요. 다른점을 찾으셨나요?

> 만일 당신이 빈 폼의 페이지를 보고 있다면, 이미 관리자 화면만들기를 통하여 로그인이 되어 있는 상태일 겁니다. `http://localhost:8000/admin/logout/` 로 로그아웃을 한 다음, `http://localhost:8000/post/new` 다시 접속해보세요. 

사랑스런 에러들 중 하나를 보게 될겁니다. 정말 흥미로운 사실 중 하나: 우리가 추가한 Decorator는 로그인 페이지로 이동하게 할겁니다. 아직 사용할 수 없지만, "페이지를 찾을 수 없습니다.(404)"의 에러를 발생합니다. 

`post_edit`, `post_remove`, `post_draft_list` , `post_publish`의 위에도 decorator를 추가해야되는 것을 잊지마세요.

아싸!, 우리의 목표지점에 도착! 다른 사람들이 우리의 블로그에 더이상 포스트를 새로 작성하지 못합니다. 불행하게도 우리 또한 더이상 포스트를 작성할 수 없군요. 그럼, 작성할 수 있도록 다음을 수정해봅시다.

## 로그인 사용자들

이제 우리는 사용자와 패스워드 그리고 인증을 구현할 수 있는 많은 종류의 마법의 도구를 사용할 수 있습니다. 그러나 이러한 종류의 도구들은 정확하게 사용하는 것은 다소 복잡합니다. 우리를 위해 열심히 일하고 있는 "배터리 완충" 된 장고는, 제공되는 인증 도구를 좀 더 사용할 수 있게 만들겁니다.

`Mysite/urls.py`에서 `url(r'^accounts/login/$', 'django.contrib.auth.views.login')` url을 추가 합니다. 그래서 파일은 아래와 같이 비슷하게 됩니다:

```python
from django.conf.urls import patterns, include, url

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', 'django.contrib.auth.views.login'),
    url(r'', include('blog.urls')),
)
```

이제 우리는 로그인 페이지의 템플릿이 필요하니, `blog/templates/registration` 의 디렉토리를 생성하고, `login.html`로 명명한 파일을 생성했던 디렉토리 안에 생성합니다.

```django
{% extends "blog/base.html" %}

{% block content %}

{% if form.errors %}
<p>이름과 비밀번호가 일치하지 않습니다. 다시 시도해주세요.</p>
{% endif %}

<form method="post" action="{% url 'django.contrib.auth.views.login' %}">
{% csrf_token %}
<table>
<tr>
    <td>{{ form.username.label_tag }}</td>
    <td>{{ form.username }}</td>
</tr>
<tr>
    <td>{{ form.password.label_tag }}</td>
    <td>{{ form.password }}</td>
</tr>
</table>

<input type="submit" value="login" />
<input type="hidden" name="next" value="{{ next }}" />
</form>

{% endblock %}
```

우리는 블로그의 전체적인 모양과 느낌을 기본 템플릿을 사용해서 나타낼수 있습니다.

여기서 좋은 점은 *just works[TM]*이라는 것입니다. 우리는 폼을 제출하거나 패스워드와 보완을 처리할 필요가 없습니다. 단지 여기서 한가지, 우리는 `mysite/settings.py`에 설정을 추가하는 것만 하면됩니다.:

```python
LOGIN_REDIRECT_URL = '/'
```

이제 로그인에 직접 접근할때, 최상위 index레벨에서 성곡적으로 로그인으로 이동됩니다.

## 레이아웃 개선하기

우리는 현재 인증된 유저에 한해서 포스팅을 추가/수정/발행시킬 수 있습니다. 그런데 아직까지 인증되지 않은 유저에게도 추가/수정 버튼이 노출되고 있는 데요. 이를 숨겨보려 합니다. 템플릿을 수정이 필요한데요. `blog/templates/blog/base.html` 파일부터 수정해봅시다.

```django
<body>
    <div class="page-header">
        {% if user.is_authenticated %}
        <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
        <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
        {% else %}
        <a href="{% url 'django.contrib.auth.views.login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
        {% endif %}
        <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
    </div>
    <div class="content">
        <div class="row">
            <div class="col-md-8">
            {% block content %}
            {% endblock %}
            </div>
        </div>
    </div>
</body>
```

어떤 패턴이 있음을 눈치채셨나요? 템플릿 내에서 if 문을 통해 인증여부를 체크하고 있습니다. 인증이 되었을 시에는 추가/수정 버튼을 노출시키고, 인증이 되지 않았을 시에는 로그인 버튼을 노출시켰습니다.

*Homework*: 인증된 유저에게만 수정 버튼만 노출되도록 `blog/templates/blog/post_detail.html` 파일을 수정해보세요.

## 인증된 사용자에 대해 더 기능 추가하기

우리의 템플릿에 좋은 설탕을 추가해봅시다. 먼저 현재 우리가 로그인 중이라는 표시를 나타내주는 몇가지 도구들을 추가할 것입니다. `blog/templates/blog/base.html`를 아래와 같이 수정해봅시다.:

```django
<div class="page-header">
    {% if user.is_authenticated %}
    <a href="{% url 'post_new' %}" class="top-menu"><span class="glyphicon glyphicon-plus"></span></a>
    <a href="{% url 'post_draft_list' %}" class="top-menu"><span class="glyphicon glyphicon-edit"></span></a>
    <p class="top-menu">Hello {{ user.username }}<small>(<a href="{% url 'django.contrib.auth.views.logout' %}">Log out</a>)</small></p>
    {% else %}
    <a href="{% url 'django.contrib.auth.views.login' %}" class="top-menu"><span class="glyphicon glyphicon-lock"></span></a>
    {% endif %}
    <h1><a href="{% url 'blog.views.post_list' %}">Django Girls</a></h1>
</div>
```

멋진 "안녕하세요. <이름>"구문을 추가함으로써 우리가 현재 인증되어 있다는 것을 알려줍니다. 또한, 블로그를 로그아웃하는 링크를 추가해봅시다. 하지만, 아직 작동하지 않습니다. 이럴수가, 우리가 인터넷을 망가뜨렸나봐요. 자, 고쳐봅시다!

우리는 로그인의 처리를 장고에게 맡기기로 결정했던 것처럼, 장고가 우리를 위해 로그아웃까지 처리해줄 수 있는지 살펴봅시다. https://docs.djangoproject.com/en/1.8/topics/auth/default/ 에서 무언가를 찾을 수 있을지 확인해봅시다.

다 살펴보았나요?`django.contrib.auth.views.logout`뷰을 가르키고 있는 URL (`mysite/urls.py` 안)을 추가하는 것에 대해 계속 생각했을 겁니다. 아래처럼요.:

```python
from django.conf.urls import patterns, include, url

from django.contrib import admin
admin.autodiscover()

urlpatterns = patterns('',
    url(r'^admin/', include(admin.site.urls)),
    url(r'^accounts/login/$', 'django.contrib.auth.views.login'),
    url(r'^accounts/logout/$', 'django.contrib.auth.views.logout', {'next_page': '/'}),
    url(r'', include('blog.urls')),
)
```

끝났어요! 만약에 당신이 여기까지 잘 따라왔다면(그리고 숙제를 했다면), 당신은 지금 아래와 같은 블로그를 갖게 되었습니다.

  * 로그인을 위해 사용자 이름과 패스워드가 필요합니다
  * 포스트를 생성/수정/게시(/삭제)를 위해 로그인을 해야하는 것이 필요합니다
  * 다시 로그아웃도 할 수 있습니다.
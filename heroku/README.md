# Heroku 배포하기 (PythonAnyhere 포함)

개발자들이 다양한 배포 방법을 알고 있다면 좋은 일이랍니다. PythonAnywhere 말고도 Heroku(헤로쿠)를 사용해 사이트를 배포해 볼까요?

[Heroku](http://heroku.com/) 는 접속자가 적은 소규모 애플리케이션을 위해 무료로 제공하고 있는데요. 하지만 배포 방법이 조금 까다롭습니다.

우리는 다음 튜토리얼을 따라서 진행해 볼 거에요. : https://devcenter.heroku.com/articles/getting-started-with-django. 여러분들이 더욱 쉽게 간편하게 배울 수 있도록 이곳에 내용을 붙여놓았습니다.

## `requirements.txt`

`requirements.txt` 파일을 만들어 Heroku에게 어떤 파이썬 패키지가 서버에 설치되어야 하는지 알려줍시다.

먼저 Heroku를 이용하기 위해서는 새로운 패키지를 설치해야합니다. 콘솔에 `virtualenv` 을 입력해 활성화시키고 다음과 같이 입력하세요. :

    (myvenv) $ pip install dj-database-url gunicorn whitenoise
    

설치를 마친 후, `djangogirls`디렉토리로 가서 다음 명령어를 실행하세요. :

    (myvenv) $ pip freeze > requirements.txt
    

이 명령어는 설치된 패키지 목록 안에 `requirements.txt` 파일을 생성합니다. (예를 들면 우리가 사용하고 있는 Django가 파이썬 라이브러리이죠. :))

> **Note**: `pip freeze` 는 virtualenv에 설치된 파이썬 라이브러리 목록을 보여줍니다. `>` 은 `pip freeze` 결과물을 가져와 파일에 넣어줍니다. `> requirements.txt` 없이 `pip freeze` 를 실행해 어떤 일이 일어나는지 확인해보세요!

파일을 열고 맨 마지막 줄에 아래 내용을 추가하세요. :

    psycopg2==2.5.4
    

Heroku에서 애플리케이션이 실행되기 위해 필요한 코드입니다.

## Procfile

Heroku가 원하는 또 다른 하나는 Procfile입니다. Procfile는 웹사이트 시작을 위해 Heroku가 명령할 수 있게 이를 알려줍니다. 코드 에디터를 열어 `djangogirls` 디렉토리 안에 `Procfile` 파일을 생성해 아래 코드를 추가하세요. :

    web: gunicorn mysite.wsgi
    

이 코드는 `web` 애플리케이션을 배포할 예정이라는 것을 뜻합니다. 이제 `gunicorn mysite.wsgi` 을 실행해 봅시다. (`gunicorn` 은 `runserver` 명령어와 비슷한 프로그램이지만 더 강력합니다)

그 다음 저장하세요. 끝!

## `runtime.txt`

이제 Heroku에게 내가 사용할 파이썬 버전을 말해줘야 합니다. 코드 에디터의 "new file (새 파일 만들기)"를 실행해 `djangogirls` 디렉토리 안에 `runtime.txt` 를 만들고, 안에 아래 내용을 채우면 됩니다. (이외에 다른 내용을 절대로 넣지 마세요) :

    python-3.4.2
    

## `mysite/local_settings.py`

Heroku는 PythonAnywhere보다 더 제한적이기 때문에, Heroku는 로컬 컴퓨터(내 컴퓨터)와 다른 설정을 원합니다. Heroku는 SQLite를 사용하면서 동시에 Postgres를 사용합니다. 때문에 로컬 환경에만 사용가능한 설정을 만들기 위해 따로 파일을 생성해야 합니다.

이제 `mysite/settings.py` 파일을 만들어 봅시다. 이 파일은 `mysite/settings.py` 로부터 `DATABASE` 설정이 포함되어야 합니다. 이렇게요. :

```python
import os
BASE_DIR = os.path.dirname(os.path.dirname(__file__))

DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}

DEBUG = True
```

위의 내용을 넣고 저장하세요! :)

## mysite/settings.py

이제 웹사이트의 `settings.py` 파일을 수정해야합니다. 에디터에서 `mysite/settings.py` 를 열고 파일 맨 마지막에 아래 내용을 추가하세요. :

```python
import dj_database_url
DATABASES['default'] = dj_database_url.config()

SECURE_PROXY_SSL_HEADER = ('HTTP_X_FORWARDED_PROTO', 'https')

ALLOWED_HOSTS = ['*']

DEBUG = False

try:
    from .local_settings import *
except ImportError:
    pass
```

이 코드는 Heroku에게 꼭 필요한 구성(configuration) 으로 `mysite/local_settings.py` 파일이 있는 경우 모든 로컬 설정을 불러옵니다.

파일을 저장하세요.

## mysite/wsgi.py

`mysite/wsgi.py` 을 열고 맨 마지막에 아래 내용을 추가하세요. :

```python
from whitenoise.django import DjangoWhiteNoise
application = DjangoWhiteNoise(application)
```

잘했어요!

## Heroku 계정

이 곳에서 Heroku toolbet를 설치하세요. (만약 설치가 되었다면 이 부분을 넘어가세요) : https://toolbelt.heroku.com/

> Heroku toolbet 프로그램 설치 시, 설치 구성 요소 화면에 "Custom Installation(사용자 정의 설치)"를 선택했는지 확인합니다. 그 다음 나타나는 구성요소 목록에서 "Git and SSH" 에 체크합니다.
> 
> 윈도우에서 Git과 SSH 명령 프롬프트의 `PATH` (경로)를 추가 하려면 다음 명령을 실행 해야 합니다. : `setx PATH "%PATH%;C:\Program Files\Git\bin"` 변경된 사항이 적용될 수 있도록 명령 프롬프트 프로그램을 다시 시작합니다.
> 
> 명령 프롬프트를 다시 시작한 후 `djangogirls` 폴더에 가서 virtualenv 를 활성화 하는 것을 잊지 마세요! (힌트: [Django 설치하기](../django_installation/README.md#working-with-virtualenv) 부분에서 확인하세요))

설치가 끝났다면 무료 Heroku 계정을 이 곳에서 만드세요. : https://id.heroku.com/signup/www-home-top

그 다음 컴퓨터에서 아래 명령을 실행하여 Heroku 계정을 인증하세요:

    $ heroku login
    

만일 SSH 키가 없는 경우, 이 명령어는 자동으로 키를 생성해줍니다. Heroku에 push 코드를 사용하기 위해서는 SSH 키가 필요합니다.

## Git commit

Heroku는 배포할 때 Git을 사용합니다. PythonAnywhere와 달리, Github을 통하지 않고 직접 Heroku에 push를 할 수 있습니다. 하지만 그 전에 몇 가지 할 일이 있어요.

`Djangogirls` 디렉토리 안에 있는 `.gitignore` 파일을 열어 `local_settings.py`를 추가합니다. `local_settings` 을 무시하기 위해 git이 필요합니다. 그래야 로컬 컴퓨터가 유지되면서 Heroko를 종료하지 않습니다.

    *.pyc
    db.sqlite3
    myvenv
    __pycache__
    local_settings.py
    

그리고 변경된 내용을 커밋합시다.

    $ git status
    $ git add -A .
    $ git commit -m "additional files and changes for Heroku"
    

## 어플리케이션 이름 선택하기

`[내 블로그 이름].herokuapp.com`을 만들어 내 블로그기 웹에 보이게 할 거에요. 내가 사용하고 싶은 이름을 정하면 됩니다. 이름은 Django `blog` app 또는 `mysite` 와 같이 기존에 만든 명칭과 상관없이 정하면 됩니다. 어떤 이름이든 상관없지만, 사용할 수 있는 문자는 제한적입니다. : 소문자(대문자와 악센트 제외), 숫자, 대쉬(-)만 사용 가능합니다.`-`).

이름(본명 또는 별명)을 정했다면, 아래 명령어 `djangogirlsblog` 부분을 사용할 이름으로 바꾸고 실행하세요. :

    $ heroku create djangogirlsblog
    

> **Note**: `djangogirlsblog` 부분을 내 Heroku 어플리케이션 이름으로 바꾸는 것을 잊지마세요.

아직 이름을 못 정했다면, 제외하고 그대로 실행하면 됩니다.

    $ heroku create
    

Heroku는 `enigmatic-cove-2527` 처럼 미사용 중인 아이디를 선택할 거에요.).

Heroku 어플레케이션 이름을 바꾸고 싶다면, 아래와 같이 실행하면 됩니다. (바꿀 이름을 `the-new-name (새 이름)`에 넣으세요)

    $ heroku apps:rename the-new-name
    

> **Note**: 어플리케이션 이름을 바꾼 후에, `[the-new-name(새 이름)].herokuapp.com`으로 접속하는 것을 잊지마세요.

## Heroku 배포하기!

지금까지 했던 설치와 구성 내용이 참 많았죠? 하나만 더하면 이제 배포할 수 있습니다!

`heroku create` 를 실행했을 때, 우리 앱은 저장소로 자동으로 옮겨져 저장되었습니다. 이제 간단한 git push로 어플리케이션을 배포할 수 있습니다.

    $ git push heroku master
    

> **Note**: 처음 실행 시 Heroku가이 컴파일되고 psycopg가 설치되면서 화면에 출력 결과물이 *아주* 많이 보일 거에요. 출력이 거의 끝날 때쯤 `https://yourapplicationname.herokuapp.com/ deployed to Heroku` 가 보이면 성공적으로 마친 거랍니다.

## 어플리케이션 접속하기

우리는 Heroku에 코드를 배포했고, Procfile에서 Process Type을 web으로 지정했었습니다. (역자 주 : Process Type 은 web, worker, urgentworker, clock 등이 있습니다) 이제 우리는 Heroku에게 `web process` 를 시작하라고 지시할 수 있습니다..

이를 위해 아래 명령어를 실행합니다. :

    $ heroku ps:scale web=1
    

이 명령어는 Heroku에게 `web` 프로세스의 한 인스턴스(instance)만 실행하라고 말합니다. 블로그 어플리케이션이 매우 간단하기 때문에, 너무 많은 프로세스를 실행할 필요가 없답니다. Heroku에게 더 많은 프로세스를 실행하도록 요청할 수 있지만 (만약에 Heroku가 "Dynos"라는 메세지를 호출하는 것이니 봐도 놀라지 마세요), 더 이상 무료로 사용할 수 없습니다. 

` heroku open` 으로 브라우저에서 앱을 볼 수 있습니다..

    $ heroku open
    

> **Note**: 에러 페이지가 보일 거예요! 이것에 대해서는 조금 있다가 이야기하도록 해요.

브라우저에서 [https://djangogirlsblog.herokuapp.com/]()로 url을 입력하면, 잠시 에러 페이지가 표시될 거에요.

지금 보이는 오류는 Heroku로 배포될 때 보여지는 것인데요. 새로운 데이터 베이스를 생성했지만 비어있기 때문에 오류가 보이는 것입니다. PythonAnywhere에서 했던 것 처럼, `migrate` 와 `createsuperuser` 명령어를 실행해 봅시다. 이번에는 커맨드 라인에 `heroku run` 을 입력해 볼게요.

    $ heroku run python manage.py migrate
    
    $ heroku run python manage.py createsuperuser
    

이 명령 프롬프트는 사용자 이름과 암호 선택을 다시 물을 거에요. 이는 웹 사이트의 관리자 페이지에 로그인 정보가 될 거에요.

브라우저를 새로고침 하면, 다 된거에요! 이제 여러분은 두 가지 호스팅 플랫폼을 배포하는 방법을 알게 되었어요. 앞으로 좋아하는 것을 선택하면 되겠죠. :)
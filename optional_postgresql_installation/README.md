# 옵션: PostgreSQL 설치하기

> 이 장의 일부는 Geek Girls Carrots (http://django.carrots.pl/)의 튜토리얼을 기반으로 작성되었습니다.
> 
> Creative Commons Attribution-ShareAlike 4.0 International License 에 따라 [django-marcador tutorial](http://django-marcador.keimlink.de/) 튜토리얼를 바탕으로 작성되었습니다. django-marcador 튜토리얼은 Markus Zapke-Gründemann et al에게 저작권이 있습니다.

## 윈도우

아래 링크에 접속해 프로그램을 다운 받으면, 윈도우에 Postgres를 쉽게 설치가 가능합니다. : http://www.enterprisedb.com/products-services-training/pgdownload#windows

운영체제에 맞는 최신버전을 선택하세요. 설치 프로그램을 다운받고 다음 가이드를 따라 설치하세요. : http://www.postgresqltutorial.com/install-postgresql/ 그 다음 단계에서 설치경로가 필요하니 주의깊게 봐두세요. (특히 `c:\Program files\PostgreSQL\9.3`부분입니다)).

## 맥 OS X

가장 쉬운 방법은 [Postgres.app](http://postgresapp.com/)을 다운받아 설치하는 것입니다.

다운받은 후, 응용 프로그램 디렉토리로 드래그하시면 설치가 끝났습니다! 클릭하여 실행하세요.

[이 곳](http://postgresapp.com/documentation/cli-tools.html) 안내에 따라 환경 변수 `PATH` 에 Postgre 경로를 추가해서 명령행에서 실행시킬 수도 있습니다..

## 리눅스

리눅스 배포판에 따라 설치방법이 상이합니다. 우분투 리눅스, 페도라 리눅스를 기준으로 설명하겠습니다. 다른 배포판은 [PostgresSQL 공식문서](https://wiki.postgresql.org/wiki/Detailed_installation_guides#General_Linux)를 참고하세요..

### Ubuntu

아래 명령을 실행해주세요.

    sudo apt-get install postgresql postgresql-contrib
    

### Fedora

아래 명령을 실행해주세요.

    sudo yum install postgresql93-server
    

# 데이터베이스 생성하기

이제 데이터베이스를 생성하고, 그 데이터베이스에 접근할 유저계정을 생성해야합니다. PostgreSQL에서는 원하는 만큼의 데이터베이스와 유저계정을 생성할 수 있습니다. 그래서 다수의 사이트를 운영할 경우, 각 사이트 별 데이터베이스를 생성할 수 있습니다.

## 윈도우

윈도우를 쓴다면, 몇 가지 설정작업이 더 필요합니다. 지금 다루는 설정들이 중요한 내용은 아닙니다. 하지만 이 설정들이 궁금하다면, 코치들에게 자유롭게 물어보세요.

  1. 명령프롬프트를 실행하세요 (시작메뉴 → 모든 프로그램 → 악세사리 → 명령 프롬프트)
  2. 다음 명령을 입력하고 엔터키를 입력해주세요: `setx PATH "%PATH%;C:\Program Files\PostgreSQL\9.3\bin"`. 명령 프롬프트에서 코드를 붙여넣기하려면, 마우스 오른쪽 버튼을 누른 후 `붙여넣기(Paste)` 를 선택하세요. 경로 끝에 `\bin` 이 있는지 확인하세요. `SUCCESS: Specified value was saved.` 라는 메세지가 출력되어야 잘 된 것입니다..
  3. 명령 프롬프트 창을 닫고 다시 열어주세요.

## 데이터베이스 생성하기

먼저, `psql` 명령을 입력해서 Postgres 콘솔을 실행하겠습니다. 콘솔을 어떻게 실행시키는 지 모두 알고 있죠?

> 맥 OS X 에서는 `Terminal(터미널)` 애플리케이션(application)을 실행시켜주세요. (애플리케이션(application) → 유틸리티(Unitilites)안에 있어요) 리눅스에서는 애플리케이션(Applications) → 악세사리(Accessories) → 터미널 경로입니다. 윈도우에서는 시작 메뉴 → 모든 프로그램 → 악세사리 → 명령 프롬프트 경로입니다. 간혹 윈도우에서는 `psql` 실행 시에, Postgres 설치 시에 선택했던 username과 password가 필요할 수 있습니다. 만약 `psql` 에서 암호를 입력했는데 인증이 되지 않는다면, `psql -U <username> -W` 명령을 입력하고 나서 암호를 입력해보세요.

    $ psql
    psql (9.3.4)
    Type "help" for help.
    #
    

프롬프트가 `$` 에서 `#` 으로 바뀌었습니다. 이제 PostgreSQL 에 명령을 보낼 수 있습니다. 유저계정을 생성해봅시다.

    # CREATE USER name;
    CREATE ROLE
    

`name` 부분을 로그인 아이디 (혹은 다른 name) 로 변경해주세요. 악센트가 들어간 글자나 공백문자는 쓸 수 없습니다. `bożena maria` 는 `bozena_maria` 로 변경해서 적용해주세요).

이제 Django 프로젝트에서 쓸 데이터베이스를 생성할 시간입니다.

    # CREATE DATABASE djangogirls OWNER name;
    CREATE DATABASE
    

`name` 에는 위 ROLE 에서 입력했던 name 을 써야함을 기억해주세요. (예 - `bozena_maria`)).

좋습니다. 필요한 데이터베이스 정리가 끝났습니다.

# Django 프로젝트 설정 (settings.py) 수정하기

`mysite/settings.py` 파일에서 DATABASES 부분을 찾아서

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
    }
}
```

아래와 같이 수정해주세요.

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': '<FIXME>',
        'USER': '<FIXME>',
        'PASSWORD': '',
        'HOST': 'localhost',
        'PORT': '',
    }
}
```

USER와 NAME에는 방금 전 생성했던 ROLE/DATABASE `name` 을 입력해주세요.

# PostgreSQL 파이썬 패키지 설치하기

먼저 https://toolbelt.heroku.com 에서 Heroku Toolbelt 을 다운받아 설치해주세요. 이 Toolbelt 에는 Git 과 더불어 배포에 필요한 모든 유틸리티들이 들어있습니다.

다음으로 PostgreSQL 패키지를 설치해야합니다. 이 패키지 이름은 `psycopg2` 입니다. 설치방법은 윈도우와 리눅스/맥 간에 다소 차이가 있습니다.

## 윈도우

윈도우에서는 http://www.stickpeople.com/projects/python/win-psycopg/ 에서 바이너리 패키지를 다운받아주세요.

현재 설치되어있는 파이썬 버전과 아키텍처 (왼쪽 칸 32비트, 오른쪽 칸 64비트) 에 맞는 패키지를 다운받아주세요.

다운받은 파일을 c:\ 경로에 psycopg2.exe 파일명으로 옮겨주세요. `C:\psycopg2.exe` 경로가 됩니다..

터미널에서 `virtualenv` 을 활성화하고, 다음 명령을 입력해주세요.

    easy_install C:\psycopg2.exe
    

## 리눅스 / OS X

터미널에서 아래 명령을 입력해주세요.

    (myvenv) ~/djangogirls$ pip install psycopg2
    

잘 수행이 되었다면, 다음과 같은 출력이 보여집니다.

    Downloading/unpacking psycopg2
    Installing collected packages: psycopg2
    Successfully installed psycopg2
    Cleaning up...
    

* * *

완료되면 `python -c "import psycopg2"`를 실행합니다. 모든 오류가 없다면 성공적으로 설치가 끝난 것입니다.
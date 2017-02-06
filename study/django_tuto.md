#Django Tutorial
##1.가상환경 설정&장고설치
* pyenv virtualenv 3.4.3 django_tuto
* pyenv local django_tuto
	* pyenv versions
* pip install django

##.2 첫번째 장고앱 작성 시작
#### Django버전 확인

		pythom -m django --version
	
#### 프로젝트 만들기

		django-admin startproject mysite
	>프로젝트를 생성 할 때 python이나 django 에서 사용중인 이름은 피애야한다.
	특히 django나 test같은 이름.


####프로젝트에서 생성 된 것
![](/home/hong/projects/django/django_tutorial/study/django_tree.png) 

* mysite/ : 디렉토리 밖의 디렉토리는 단순히 프로젝트를 담는 공간. 이 이름은 django와 아무 상관이 없으니 원하는 이름으로 변경가능.
	* 디렉토리 내부에는 project를 위한 실제 Python 패키지들이 저장
	* __init__.py : Python으로 하여금 이 디렉토리를 패키지 처럼 다루라고 알려주는 용도의 단순한 빈 파일.
	* urls.py : 현재 Django projects의 URL선언을 저장
	* wsgi.py : 현재 프로젝트를 서비스하기 위한 WSGI 호환 웹서버의 진입점
* manage.py : Django 프로젝트와 다양한 방법으로 상호작용하는 커맨드라인의 유틸리티

#### 개발서버
Django project가 제대로 동작하는지 확인

	python manage.py runserver

#### 설문조사 앱 만들기

	python manage.py startapp polls
	
	polls란 디렉토리가 생성

#### 첫 번째 view 작성하기
	polls/view.py
	
	from django.http import HttpResponse
	
	def index(request):
	return HttpResponse("Hello, world. You're at the polls index.")
__	
	
	polls/urls.py
	
	from django.conf.urls import url
	from . import views

	urlpatterns = [
    		url(r'^$', views.index, name='index'),
	]
__
	
	mysite/urls.py
	
	from django.conf.urls import url, include
	from django.contrib import admin

	urlpatterns = [
    		url(r'^polls/', include('polls.urls')),
    		url(r'^admin/', admin.site.urls),
	]

include()함수는 다른 URLconf를 참조 할 수 있도록 도와줍니다. include()함수를 위한 정규 표현식에서 끝을 의미하는 기호로 $대신에 /가 붙는다.
> include()는 admin.site.urls를 제외한 다른 URL패턴을 include 할 때마다 include()를 사용해야 합니다.

index view 가 URLconf 에서 연결됐고, 잘 동작하는지 확인하기 위해
	
	python mange.py runserver을 입력하면 index view로 정의한 'Hello, world. You're at the polls index'가 보인다.
	
#####ulr() 인수 regex
regex는 보통 정규 표현식을 짧게 줄여 쓰는 표현. 문자열의 패턴을 일치시키는 문법을 말하며, 이 경우에는 url의 패턴을 찾아내는데 사용
#####url() 인수 view
django에서 일치하는 정규표현식을 찾아내면 HttpRequest 객체를 첫번째 인수로 하고, 정규 표현식에서 잡힌 값들은 나머지 인수로 하여 특정한 view 함수를 호출호
#####url() 인수 kwargs
임의의 키워드 인수들은 목표한 view에 사전형으로 전달됩니다. 그러나 이 튜토리얼에서는 사용하지 않는다.
#####url() 인수 name
URL에 이름을 지으면, 탬플릿을 포함한 Django 어디에서나 명확하게 참조 할 수 있다. 이 강력한 기능을 이용하여, 단 하나의 파일만 수정해도 project 내의 모든 URL패턴을 바꿀 수 있도록 도와줌.

####데이터베이스 설치
mysite/settings.py 파일을 열어본다. Django 설정을 모듈 변스로 표현한 보통의 Python 모듈.
기본적으로는 SQLite를 사용하돌고 구성.
TIMEZONE설정 - Asia/Seoul

* django.contrib.admin - 관리용 사이트, 곧 사용.
* django.contrib.auth - 인증시스템
* django.contrib.contenttypes - 컨텐츠 타입을 위한 프레임워크
* django.contrib.sessions - 세션 프레임워크
* django.contrib.messages - 메시징 프레임워크
* django.contrib.staticfiles - 정적파일을 관리하는 프레임워크 

이러한 기본 어플리케이션들 중 몇몇은 최소한 하나의 데이터베이스 테이블을 사용하는데, 그러기 위해서는 데이터베이스에서 테이블을 미리 만들 필요가 있다.

	python mange.py migrate
	
	migrate 명령은 INSTALLED_APPS의 설정을 탐색하여, mysite/setting.py의 데이터베이스 설정과 app과 함께 제공되는 데이터베이스 migrations에 따라 필요한 데이터베이스 테이블을 생성.
	
####모델 만들기

	polls/models.py
	
	from django.db import models

	class Question(models.Model):
		question_text = models.CharField(max_length=200)
		pub_date = models.DateTimeField(date published)

	class Choice(models.Model):
		question = models.ForignKey(Question, on_date=models.CASCADE)
		choice_text = models.CharField(max_length=200)
		votes = models.IntegerField(default=0)
	
	각 모델은 django.db.models.Model이라는 클래스의 서브클래스로 표현. 각 모델은 몇개의 클래스 변수를 가지고 있으며, 각각의 클래스 변수들은 모델의 데이터베이스 필드를 나타냅니다.
	
	mysite/setting.py 
	
	INSTALLED_APPS = [
		'polls.apps.PollsConfig', 를 추가
	]
	
shell에서
	
	python mange.py makemigrations polls
	
	Migrations for 'polls':
 	 polls/migrations/0001_initial.py:
 	 	- Create model Choice
 	 	- Create model Question
 	 	- Add field question to choice
 	 
 	 makemigration 명령은 model을 수정햇으며 model의 변경을 migration으로 저장하고 싶다고 장고에 알립니다.


<hr>
###Django Admin사이트
 
 	python manage.py createsuperuser
	Username
	Email address
	Password
	에 원하는 내용을 넣습니다.
	
	python manage.py runserver
	로 서버를 실행시킵니다
	
	localhost:8000/admin 으로 접속
	
####Admin 사이트에서 설문 app 수정가능하게 만들기

	polls/admin.py
	
	from django.contrib import admin
	
	from .models import Question
	
	admin.site.register(Question)
	
	에지 화면에 Qustion을 등록했으므로 admin사이트 첫화면에 나타납니다.
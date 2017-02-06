###View
view는 일반적으로 특정 기능을 수행하거나 특정한 템플릿을 제공하기 위해 장고 어플리케이션에 들어 있는 웹페이지의 한 유형. 예를들어 blog 어플리케이션에는 다음과 같음 view가 존재

* 블로그 홈페이지 - 가장 최신의 몇몇 항목 보야줌
*  세부 항목 페이지 - 하나의 항목을 위한 permalink 페이지
* 연도별 아카이브 페이지 - 주어진 년도의 모든 달을 항목과 함께 보여줌
* 월별 아카이브 페이지 - 주어진 달의 모든 일을 항목과 함계 보여줌
* 일별 아카이브 페이지 - 주어진 날의 모든 항목을 보여줌
* 댓글 달기 - 주어진 항목에 댓글을 올리는 작업 담당

투표 어플리케이션에는 다음과 같은 view가 존재

* Question "index" page - 가장 최신의 몇몇 질문들을 보여줌
* Question "detail" page - 투표를 위한 폼과 질문 내용 보여줌. 결과는 보여주지 않음
* Question "results" page - 특정 질문에 대한 결과 보여줌
* Vote action - 특정 질문에 대해 특정 선택을 할 수 있게 투표를 관리

####더 많은 view 만들기

	polls/views.py
	
	def detail(request, question_id):
		return HttpResponse("You're looking at question %s." % question_id)
		
	def results(request, question_id):
		response = "You're looking at the results of question %s."
		return HttpResponse(response % question_id)
		 
	def vote(request, question_id):
		return HttpResponse("You're voting on question %s." % question_id)

--

	polls/urls.py
	
	from django.conf.urls import url

	from . import views

		urlpatterns = [
		# ex: /polls/
		url(r'^$', views.index, name='index'),
		# ex: /polls/5/
		url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
		# ex: /polls/5/results/
		url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
		# ex: /polls/5/vote/
		url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
		]
		
		브라우저에서 polls/34 입력 결과를 살펴보면 detail() 메소드가 실행되고 
		URL에 제공한 ID를 출력함

누가 웹사이트에서 페이지를 요청하면 장고가 mysite.urls 파이썬 모듈을 로드하는 이유는 ROOT_URLCONF 설정이 이 모듈을 가리키기 때문.
이 모듈은 urlpatterns라는 이름의 변수를 찾고, 정규 표현식을 순서대로 탐색. 
polls/에 맞는 정규표현식을 찾고 나서, 다음 처리를 위해 일치된 텍스트 뒤에 남은 "34/" 텍스트를 'polls,urls'URLconf에 전송. 파이썬 모듈이 r'^(?P<question_id>[0-9]+)/$'에 일치하는 정규표현식을 찾으면 detail() 뷰를 다음처럼 요청

	detail(request=<HttpRequest object>, question_id='34')

question_id=’34’는 (?P<question_id>[0-9]+)와 일치. 패턴을 둘러싼 괄호를 사용해 해당 패턴과 일치하는 텍스트를 포착하고 이를 뷰 함수에게 인자로 보냄. ?P<question_id>는 일치한 패턴을 확인하는데 사용할 이름을 정의. 그리고 [0-9]+는 0부터 9까지의 연쇄(즉 숫자)와 일치하는 정규 표현식.
####실제로 무엇인가하는 view작성하기
각 view는 요청된 페이지를 위한 컨텐츠를 담은 HttpResponse 객체를 반환하거나, Http404와 같은 예외를 던집니다.
만들어진 view는 데이터베이스에 있는 레코드를 읽거나 읽지 않을 수도 있다.

	polls/views.py
	
	from django.http import HttpResponse
	from .models import Question

	def index(request):
		latest_question_list = Question.objects.order_by('-pub_date')[:5]
		output = ', '.join([q.question_text for q in latest_question_list])
		return HttpResponse(output)

	페이지의 디자인이 뷰안에 고정되있다는 문제가 존재. 만일 페이지의 디자인을 변경하고 싶다면, 이 파이썬 코드를 수정해야함. 따라서 템플릿 시스템을 사용해 뷰가 사용할 수 있는 템플릿을 생성하는 방법으로 파이썬에서 디자인을 분리
	
	먼저 polls디렉토리에 templates라는 디렉토리 생성(장고는 이 디렉토리에서 템플릿을 찾음)
	프로젝트의 TEMPLATES 설정은 장고가 어떻게 템플릿을 로드하고 화면을 표시할지에 대해 기술. 초기 설정 파일은 Django Templates 백엔드를 구성하고 이것의 APP_DIRS 옵션은 True라고 설정. 관례적으로 Django Templates은 각각의 INSTALLED_APPS안에 templates라는 하위 디렉토리 탐색.
	
	polls/templates/polls/index.html
	
	{% if latest_question_list %}
  		<ul>
    		{% for question in latest_question_list %}
    			<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>
    		{% endfor %}
      	</ul>
	{% else %}
   		<p>No polls are available.</p>
	{% endif %}

	polls/views.py 수정
	
	from django.http import HttpResponse
	from django.template import loader

	from .models import Question


	def index(request):
		latest_question_list = Question.objects.order_by('-pub_date')[:5]
		template = loader.get_template('polls/index.html')
		context = {
			'latest_question_list': latest_question_list,
		}
		return HttpResponse(template.render(context, request))
	
	위 코드는 polls/index.html이라는 템플릿을 로드하며, 이를 context에 전달.(contexts는 템플릿 변수 이름을 파이썬 객체에 매핑하는 dict)
	
####지름길 render()
템플릿을 로드하고, context를 채우고 완성된 템플릿의 결과와 함께 HttpResponse 객체를 반환하기 위한 코드 묶음은 매우 흔한 관용구이다. 

	polls/views.py
	
	from django.shortcuts import render
	from .models import Question
	
	def index(request):
		lastes_question_list = Question.objects.order_by('-pub_date')[:5]
		context = {'latest_question_list': latest_question_list}
		return render(request, 'polls/index.html', context)
	
	한번만 각 view들에서 이런 작업을 하고 나면 더이상 loader나 HttpResponse를 임포트할 필요가 없다는 사실에 주목하자.
	render()함수는 첫번재 인자로 요청하는 객체를 두번째 인자로 템플릿 이름을 부가적인 세번재 인자로 딕셔너리를 받는다.
	render()함수는 받은 context로 완성된 해당 템플릿의 HttpResponse 객체를 리턴
	
	
	
### 404error

	polls/views.py
	
	from django.http import Http404
	from django.shortcuts import render
	
	from .models import Qustion
	
	def detail(request, question_id):
		try:
			question = Question.objects.get(pk=question_id)
		except Question.DoesNotExist:
			raise Http404("Question does not exist")
		return render(request, 'polls/detail.html', {'question': question})
		
	만일 요청된 ID의 질문이 존재하지 않는다면, 뷰는 Http404예외를 던짐.

####지름길 get_object_or_404()
get_object_or_404()는 get()을 사용하며 만일 객체가 존재하지 않으면 Http404를 던지기 위해 사용하는 아주 흔한 관용구로 장고가 제공하는 지름길. 재작성된 detail()뷰는 다음과 같다.

	polls/views.py
	
	from django.shortcuts import get_object_or_404, render
	
	from .models import Question
	
	def detail(request, question_id):
		question = get_object_or_404(Question, pk=question_id)
		return render(request, 'polls/detail.html', {'questuon': question})
		
	get_object_or_404() 함수는 장고 모델을 첫 번쨰 인자로 받고, 키워드 인자 여러 개를 모델 관리자의 get() 함수에 전달. 해당 객체가 존재하지 않는 경우라면 HTtp404()를 던집니다.
	
###템플릿 시스템 사용하기
컨텍스트 변수인 question이 주어지면 polls/detail.html 템플릿은 다음과 같다.

	polls/templates/polls/detail.html
	
	<h1>{{ question.question_text }}</h1>
	<ul>
	{% for choice in question.choice_set.all %}
		 <li>{{ choice.choice_text }}</li>
	{% endfor %}
	</ul>

	템플릿 시스템은 변수 속성에 접근하기 위해 점을 기준으로 찾는 구문법을 사용. {{ question.question_text }}의 예시를 보면, 장고는 우선 question 객체에서 사전을 조히합니다. 만일 여기서 찾지 못하면 장고는 속성을 조회합니다. 만일 속성 조회도 실패하면 리스트-색인 조회를 시도
	
	매소드 호출은 {% for %} 루프에서 발생합니다. question.choice_set.all은 파이썬 코드 question.choice_set.all()로 해석됨. question.choice_set.all()순회할 수 있는 Choice 객체 모듬을 반환하므로 {% 랙 %} 태그안에서 사용 가능해짐
	
###템플릿에 고정되어 있는 URL 제거하기
polls/index.html 템플릿에서 질문에 대해 연결하는 코드를 작성할 때, 링크는 다음처럼 부분적으로 코드 안에 고정된다는 사실을 기억

	<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>

	이렇게 고정되고 결합이 강한 접근 방법을 사용하면 템플릿이 엄청 많음 프로젝트에서 URL을 변경할 때 난관에 부딪히는 문제를 초래.
	
	하지만 polls.urls 모듈의 url() 함수에 name인수를 정의했으므로, URL 구성에 정의햇던 특정 URL 결로 의존성을 {% url %} 템플릿 태그를 사용해 제거할 수 있음.
	
	<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
	
	이는 polls.urls 모듈에 URL 정의를 탐색하는 방식으로 동작. 'detail'의 URL 이름이 정확히 어디에 정의되었는지 확인해보자
	
	url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),

	만약 투표 세부 뷰의 URL을  (polls/specifics/12/처럼) 다른 URL로 변경하고 싶다면, 템플릿에 고정시키는 방식 대신 polls/urls.py에서 다음처럼 변경
	
	url(r'^specifics/(?P<question_id>[0-9]+)/$', views.detail, name='detail'),

###URL 이름을 이름 영역으로 분리하기
이 튜토리얼에는 오직 'polls'라는 어플리케이션 하나만 존재. 실제로 장고 프로젝트에서는 5개, 10개, 20개 또는 그보다 더많은 앱을 포함. 그렇다면 장고는 어떻게 그 많은 URL 이름을 구분할까? 

URLconf에 이름영역을 추가하는 방법이 정답이다. polls/urls.py 파일에서, 어플리케이션 이름영역을 설정하기 위해 app_name을 추가

	polls/urls.py
	
	from django.conf.urls import url
	from .import views
	
	app_name = 'polls'
	urlpatterns = [
		url(r'^(?P<question_id>[0-9]+/$', views.detail, name='detail'),
		url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
		url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
	]
	
	이름영역이 없는 polls/index.html 템플릿을 
	polls/templates/polls/index.html
	
	<li><a href="{% url 'detail' question.id %}">{{ question.question_text }}</a></li>
	를
	<li><a href="{% url 'polls:detail' question.id %}">{{ question.question_text }}</a></li>
	
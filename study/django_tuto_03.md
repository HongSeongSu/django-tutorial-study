###간단한 폼 만들기

	polls/templates/polls/detail.html
	에 form요소 추가
	
	<h1>{{ question.question_text }}</h1>

	{% if error_message %}<p><strong>{{ error_message }}</strong></p>{% endif %}

	<form action="{% url 'polls:vote' question.id %}" method="post">
	{% csrf_token %}
	{% for choice in question.choice_set.all %}
		<input type="radio" name="choice" id="choice{{ forloop.counter }}" value="{{ choice.id }}" />
		<label for="choice{{ forloop.counter }}">{{ choice.choice_text }}</label><br />
	{% endfor %}
	<input type="submit" value="Vote" />
	</form>
	
polls/views.py 에 다음을 추가

	from django.shortcuts import get_object_or_404, render
	from django.http import HttpResponseRedirect, HttpResponse
	from django.urls import reverse

	from .models import Choice, Question
	# ...
	def vote(request, question_id):
		question = get_object_or_404(Question, pk=question_id)
		try:
			selected_choice = question.choice_set.get(pk=request.POST['choice'])
		except (KeyError, Choice.DoesNotExist):
			return render(request, 'polls/detail.html', {
				'question': question,
				'error_message': "You didn't select a choice.",
			 })
		else:
			selected_choice.votes += 1
			selected_choice.save()
		# POST 데이터를 성공적으로 처리한 후에는 항상 HttpResponseRedirect를
		# 반환합니다. 그래야 사용자가 뒤로가가(Back) 버튼을 클릭했을 때 폼이
		# 두 번 제출되는 현상이 생기지 않습니다.
		return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
		
* request.POST는 제출된 데이터를 키 이름으로 접근할 수 시는 사전 같은 객체.
이 예제에서 request.POST['choice']는 선택된 항목의 ID를 문자열로 반환.
request.POST 값은 항상 문자열.

###코드는 적을수록 좋다
result()뷰와 detail() 뷰는 아주 단순하지만 코드가 중복됨. 설문 목록을 보여주는 index()뷰도 마찬가지
이들 뷰는 웹개발에서 흔히 볼수 있다. URL로 전달된 매개변수에 따라 데이터베이스에서 데이터를 가져온 후 템플릿을 로드하고 템플릿에 데이터를 채워 반환한다. 장고는 제너릭 뷰라는 시스템을 제공한다.

####URLconf 수정하기

	polls/urls.py
	
	from django.conf.urls import url
	
	from . import views
	
	app_name = 'polls'
	urlpatterns = [
		url(r'^$', views.IndexView.as_view(), name='index'),
		url(r'^(?P<pk>[0-9]+)/$', views.DetailView.as_view(), name='detail'),
		url(r'^(?P<pk>[0-9]+)/results/$', views.ResultsView.as_view(), name='results'),
		url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
	]
	
	두번째와 세번째 정규 표현식에서 일치하는 패턴 이름을 <question_id> 에서 <pk>로 변경한 사실에 주목
	
#####뷰 수정하기
 기존의 낡은 indexm detail, results 뷰를 제거하고 장고 제너릭 뷰를 사용합니다
 
 	polls/views.py
 	
 	from django.shortcuts import get_object_or_404, render
 	from django.htpp import HttpResponseRedirect
 	from django.urls import reverse
 	from django.views import generic
 	
 	from .models import Choice, Question
 	
 	class IndexView(generic.ListView):
 		template_name = 'polls/index.html'
 		context_object_name = 'latest_question_list'
 		
 		def get_queryset(self):
 		"""Return the last five published questions."""
		return Question.objects.order_by('-pub_date')[:5]

	class DetialView(generic.DetailView):
		model = Question
		template_name = 'polls/results.html'
		
	def vote(request, question_id):
	...#변경사항 없음

설문 앱에서는 제너릭 뷰 두개를 사용. 하나는 ListView이고 다른 하나는 DetailView입니다. 
ListView는 객체 목록 보여주기 개념을 추상화하고
DetailView는 특정 타입의 객체 정보를 상세하게 보여주기 개념을 추상화합니다.

DetailView 제너릭 뷰는 URL에서 포착된 기본 키의 이름이 pk라고 간주한다(그래서 제너릭뷰를 위해 question_id를 pk로 변경)

기본적으로 DetailView는 <.app name>/<.modle name>_detail.html 템플릿을 사용하게 됩니다.
이때 template_name 속성을 지정하면 장고는 자동으로 생성된 기본 템플싱이 아니라 지정된 템플릿 이름을 사용. 위에서 results 목록 뷰에도 template_name을 지정. 결과 뷰와 상세 뷰가 모두 뒤에서는 DetailView이지만 최종 모양은 달라짐.

ListView도 <.app name>/<.model name>_list.html라는 기본 템플릿을 사용하지만 위에서는 template_name 속성을 이용해 ListView에게 우리가 미리 만든 'polls/index.html' 템플릿을 사용하라고 지정.

question과 latest_question_list라는 컨텍스트 변수가 담긴 컨텍스트를 템플릿에 제공. DetailView에서 question 변수가 자동을 제공됨. 장고 모델 Question을 사용하므로 장고가 컨텍스트 변수 이름을 스스로 파악할 수 잇음. ListView는 자동으로 생성되는 컨텍스트 변수가 question_list입니다.

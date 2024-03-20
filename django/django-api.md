## django API (database API)
interactive python shell에서 django API를 실행할 수 있다.

python shell을 실행할 때 현재 django project의 DJANGO_SETTINS_MODULE 환경변수를 적용하기 위해서 manage.py를 통해 shell을 실행한다.
```shell
$ poetry run python manage.py shell
```

이제 shell에서 [database API](https://docs.djangoproject.com/en/5.0/topics/db/queries/) 를 사용하여 [이전 글](https://itsurvival.tistory.com/21)에서 작성한 model들을 가지고 오브젝트를 만들거나 database에 쿼리를 실행할 수 있다.

```python
# 이전 글에서 작성한 Question, Choice 모델들을 import
>>> from polls.models import Choice, Question
>>> Question.objects.all()
<QuerySet []>  # 아직 저장된 Question이 없음

# 새 Question 객체 생성
# - 기본적으로 timezone 설정이 활성화 되어 있으므로
#  pub_date 필드에 저장되하는 datetime 객체에는 timezone 포함되므로
#  datetime.datetime.now() 보다 timezone.now() 를 사용해서
#  tzinfo를 포함 시키는 것이 좋다. 
>>> from django.utils import timezone
>>> q = Question(question_text="What's new?", pub_date=timezone.now())

# Question 객체를 database에 저장
>>> q.save()

# database에 저장 하고 나면 id(pk)가 할당된다. 
>>> q.id
1

# python 객체의 속성으로서 모델의 필드 값을 참조할 수 있다.
>>> q.question_text
"What's new?"
>>> q.pub_date
datetime.datetime(2024, 3, 19, 7, 21, 14, 138222, tzinfo=datetime.timezone.utc)

# 속성의 값을 변경하고 save() 함수를 실행하여 database의 필드 값을 변경할 수 있다.
>>> q.question_text = "What's up?"
>>> q.save()

# objects.all() 함수로 database에 있는 모든 데이터를 조회
>>> Question.objects.all()
<QuerySet [<Question: Question object (1)>]>
```

조회 결과로 Question 리스트를 받아보면 방금 저장한 객체 하나를 볼 수 있는데, `<Question: Question object (1)>` 는 id 값만 보여진다. 모델 클래스에 `__str__()` 메소드를 만들어서 객체가 str 타입으로 형변환 했을 때의 결과 값을 정해줄 수 있다.

```python
from django.db import models


class Question(models.Model):
    question_text = models.CharField(max_length=200)
    # ...
    def __str__(self):
        return self.question_text

class Choice(models.Model):
    choice_text = models.CharField(max_length=200)
    # ...
    def __str__(self):
        return self.choice_text
```

이제 python shell에서 객체를 조회하면 다음과 같이 객체의 question_text 필드의 값을 확인할 수 있다. 뿐만 아니라 django admin에서 모델의 객체를 표현할 때도 적용된다.
```
>>> from polls.models import Choice, Question

>>> Question.objects.all()
<QuerySet [<Question: What's up?>]>
```

django는 풍부한 데이터베이스 조회 AP를 제공한다.
```
# id가 1인 question 조회
>>> Question.objects.filter(id=1)
<QuerySet [<Question: What's up?>]>

# question_text 값이 "What"으로 시작하는 question 조회
>>> Question.objects.filter(question_text__startswith="What")
<QuerySet [<Question: What's up?>]>

# 올해 등록된 question 조회
# settings.py 에 설정한 timezone을 반영하기위해 django.utils.timezone 사용
>>> from django.utils import timezone
>>> current_year = timezone.now().year
>>> Question.objects.get(pub_date__year=current_year)
<Question: What's up?>
```

객체를 id(primary key)로 조회하는 것은 가장 일반적인 방식이며 primary key(id)는 pk라는 키워드로 줄여서 표현할 수 있다.
```
# 다음은 "Question.objects.get(id=1)"과 같다.
>>> Question.objects.get(pk=1)
<Question: What's up?>

# 만약 존재하지 않는 id를 조회할 경우 exception을 발생시킨다.
>>> Question.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Question matching query does not exist.
```

Question과 Choice는 1:N 관계이다. 따라서 Choice의 필드엔 question id를 foreign key를 갖는다. 이렇 foreign key를 통해 참조할 때엔 다음과 같이 표현할 수 있다.
```
# question 조회
>>> q = Question.objects.get(pk=1)

# question와 관계된 choice들을 조회
# {object}.{related_model_name}_set
>>> q.choice_set.all()
<QuerySet []>

# question object와 관계된 choice들을 생성(저장)
>>> q.choice_set.create(choice_text="Not much", votes=0)
<Choice: Not much>
>>> q.choice_set.create(choice_text="The sky", votes=0)
<Choice: The sky>
>>> c = q.choice_set.create(choice_text="Just hacking again", votes=0)

# choice object와 관계된 question을 참조할 수 있다.
>>> c.question
<Question: What's up?>

# 그 반대로 question object에서 관계된 choice object들을 참조할 수 있다.
>>> q.choice_set.all()
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
>>> q.choice_set.count()
3
```

object와 관계된 object의 필드에 대한 쿼리도 가능하다.
```
>>> Choice.objects.filter(question__pub_date__year=current_year)
<QuerySet [<Choice: Not much>, <Choice: The sky>, <Choice: Just hacking again>]>
```

끝.

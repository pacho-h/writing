## django API
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




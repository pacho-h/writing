# DRF(Django REST Framework)

https://www.django-rest-framework.org/

DRF는 django에서 Web API를 구현할때 필요한 인증/인가, 직렬화 등 유용한 기능을 미리 구현해둔 프레임워크이다.

## Installation

```
# 먼저 Django가 설치 되어 있어야 한다.
(django-practice-py3.12) $ poetry add djangorestframework
Using version ^3.15.0 for djangorestframework

Updating dependencies
Resolving dependencies... (0.1s)

Package operations: 1 install, 0 updates, 0 removals

  - Installing djangorestframework (3.15.0)

Writing lock file

```

DRF 실습을 위한 API app 생성
```
(django-practice-py3.12) $ django-admin startapp api
(django-practice-py3.12) $ tree api
api
├── __init__.py
├── admin.py
├── apps.py
├── migrations
│   └── __init__.py
├── models.py
├── tests.py
└── views.py

2 directories, 7 files
```

## Serializer 작성

`api/serializers.py`

```python
from django.contrib.auth.models import Group, User
from rest_framework import serializers


class UserSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = User
        fields = ['url', 'username', 'email', 'groups']


class GroupSerializer(serializers.HyperlinkedModelSerializer):
    class Meta:
        model = Group
        fields = ['url', 'name']
```

### HyperlinkedModelSerializer
DRF에서는 HyperlinkedModelSerializer를 사용하는 것을 권장하고 있다. 
HyperlinkedModelSerializer는 장고레스트 프레임워크의 클래스로 기본 키 대신 하이퍼링크를 사용하여 관계를 표현하는데, 이 클래스는 ModelSerializer 클래스와 비슷하지만 다음과 같은 차이점이 있다.

- id(pk) 필드 대신 url 필드가 포함된다.(필요시 추가할 수 있음)
- 다른 인스턴스와의 관계는 기본 키가 아닌 하이퍼링크로 갖는다.
- 기본적으로 id 필드를 포함하지 않는다.
- 관계는 PrimaryKeyRelatedField 대신 HyperlinkedRelatedField를 사용 한다.

## View 작성

`api/views.py`

```python
from django.contrib.auth.models import Group, User
from rest_framework import permissions, viewsets

from .serializers import GroupSerializer, UserSerializer


class UserViewSet(viewsets.ModelViewSet):
    """
    User들을 보거나 편집할 수 있는 API endpoint
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]


class GroupViewSet(viewsets.ModelViewSet):
    """
    Group들을 보거나 편집할 수 있는 API endpoint
    """
    queryset = Group.objects.all().order_by('name')
    serializer_class = GroupSerializer
    permission_classes = [permissions.IsAuthenticated]
```
모든 일반적인 동작을 처리하기 위한 여러 뷰를 작성하는 대신  `ViewSets` 클래스로 그룹화한다.

필요한 경우 이를 개별 View로 쉽게 나눌 수 있지만 `ViewSets`를 사용하면 View 로직이 매우 간결하게 정리된다.

## URL 작성

`django_practice/urls.py`
```python
from django.contrib import admin
from django.urls import include, path
from rest_framework import routers

from api import views

router = routers.DefaultRouter()
router.register(r"users", views.UserViewSet)
router.register(r"groups", views.GroupViewSet)
urlpatterns = [
    path("polls/", include("polls.urls")),
    path("admin/", admin.site.urls),
    path("", include(router.urls)),
    path("api-auth/",
         include("rest_framework.urls", namespace="rest_framework")),
]
```

## Settings

`django_practice/settins.py`
```python
# ...
INSTALLED_APPS = [
    # ...
    "rest_framework",
]
```

## Test

서버 실행
```
(django-practice-py3.12) $ python manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
March 21, 2024 - 09:04:21
Django version 5.0.3, using settings 'django_practice.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.

```

API 호출 테스트
```
$ curl -u pacho -H 'Accept: application/json; indent=4' http://localhost:8000/users/
Enter host password for user 'pacho':
[
    {
        "url": "http://localhost:8000/users/1/",
        "username": "pacho",
        "email": "pacho@example.com",
        "groups": []
    }
]
```

끝.

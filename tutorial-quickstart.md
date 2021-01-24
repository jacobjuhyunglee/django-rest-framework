## 프로젝트 셋업

새 장고 프로젝트 `tutorial`을 생성하고 `quickstart`라는 새로운 앱을 생성하자.

```bash
# 새로운 프로젝트 디렉토리 생성
mkdir tutorial
cd tutorial

# 패키지 의존성을 로컬에서 분리하기 위해 가상환경 생성
python3 -m venv env
source env/bin/activate # 윈도우에서는 `env\Scripts\activate` 사용

# 가상환경에 Django와 Django REST Framework 설치
pip install django
pip install djangorestframework

# 단일의 앱으로 프로젝트 셋업
django-admin startproject tutorial .  # 점 사용시 프로젝트 폴더를 하나만 만들어줌
cd tutorial
django-admin startapp quickstart
```

셋업을 끝내면 아래와 같은 파일 구조가 갖춰졌을 것이다.

```bash
$ tree

├── manage.py
└── tutorial
    ├── __init__.py
    ├── __pycache__
    │   ├── __init__.cpython-39.pyc
    │   ├── settings.cpython-39.pyc
    │   ├── urls.cpython-39.pyc
    │   └── wsgi.cpython-39.pyc
    ├── asgi.py
    ├── quickstart
    │   ├── __init__.py
    │   ├── __pycache__
    │   │   ├── __init__.cpython-39.pyc
    │   │   ├── serializers.cpython-39.pyc
    │   │   └── views.cpython-39.pyc
    │   ├── admin.py
    │   ├── apps.py
    │   ├── migrations
    │   │   └── __init__.py
    │   ├── models.py
    │   ├── serializers.py
    │   ├── tests.py
    │   └── views.py
    ├── settings.py
    ├── urls.py
    └── wsgi.py
```

그 다음, 데이터베이스를 동기화 해준다

```bash
python manage.py migrate
```

admin을 위한 superuser를 생성해준다.

```bash
python manage.py createsuperuser --email admin@example.com --username admin
```

## Serializers

**`Serializer`**는 쿼리셋과 모델 인스턴스같은 복잡한 데이터들을 `JSON`, `XML` 형태나 다른 컨텐츠 타입으로 쉽게 렌더링 해주는 특유의 파이썬 데이터타입으로 전환시켜준다.

```python
from django.contrib.auth.models import User, Group
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

이 예제에서는 `HyperlinkedModelSerializer`로 `hyperlinked relation`를 사용했는데, 이 방법이 RESTful한 방법이라고 한다.

## Views

`Viewsets`를 사용함으로 로직을 간결하게 정리해줄 수 있다.

```python
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from rest_framework import permissions
from tutorial.quickstart.serializers import UserSerializer, GroupSerializer

class UserViewSet(viewsets.ModelViewSet):
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]

class GroupViewSet(viewsets.ModelViewSet):
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
    permission_classes = [permissions.IsAuthenticated]
```

## URLs

장고에서 쓰던 view가 아니라 `viewsets`를 사용하기 때문에, 간단히 `viewsts`를 **라우터 클래스로 등록**해줌으로 자동적으로 `URL conf`를 생성할 수 있다 `(tutorial/urls.py)`.

```python
from django.urls import include, path
from rest_framework import routers
from tutorial.quickstart import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# 자동 URL 라우팅을 사용해서 API를 연결해준다.
# 추가적으로 browsable API를 사용하기 위해 로그인 URl을 포함한다.

urlpatterns = [
    path('', include(router.urls)),
    path('api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

만약 API URL이 많은 제어를 필요시 한다면, 기존 `class-based views`를 사용하고 `URL conf`를 명시해줌으로 쉽게 교체할 수 있다.

## Pagination

---

`Pagination`은 사용자가 몇개의 객체가 페이지 안에 리턴 될지 제어할 수 있게 해준다.

Pagination은 추가하려면 아래의 코드를 `settings.py`에 추가해주면 된다.

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}
```

## Settings

---

`'rest_framework'`를 `settings.py`에서 `INSTALLED_APPS`에 추가해준다.

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

## API 테스트하기

---

이제 테스트를 할 준비가 되었으니, `runserver`를 해주자.

```python
python manage.py runserver
```

모든 준비를 마쳤다면, 작성한 API의 접근이 가능하게 되는데, curl 또는, httpie를 이용하거나, 브라우저에서 접속이 가능하다.

**curl**

```python
curl -H 'Accept: application/json; indent=4' -u admin:wngud123 http://127.0.0.1:8000/users/
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
						"url": "http://127.0.0.1:8000/users/1/",
            "username": "admin",
            "email": "jacob.juhyung.lee@gmail.com",
            "groups": []
        }
    ]
}
```

**httpie**

```python
http -a admin:wngud123 http://127.0.0.1:8000/users/

HTTP/1.1 200 OK
...
{
    "count": 1,
    "next": null,
    "previous": null,
    "results": [
        {
            "email": "jacob.juhyung.lee@gmail.com",
            "groups": [],
            "url": "http://127.0.0.1:8000/users/1/",
            "username": "admin"
        }
    ]
}
```

**brower**

![](https://s3.us-west-2.amazonaws.com/secure.notion-static.com/c0b1d2b3-8c6d-4a02-858a-f81905589183/_2021-01-20__3.18.05.png?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Credential=AKIAT73L2G45O3KS52Y5%2F20210124%2Fus-west-2%2Fs3%2Faws4_request&X-Amz-Date=20210124T033536Z&X-Amz-Expires=86400&X-Amz-Signature=efce40e4c67d47c4e4f6b3b70980580cca9aa1e098b71f03b082b6ffb515ac2a&X-Amz-SignedHeaders=host&response-content-disposition=filename%20%3D%22_2021-01-20__3.18.05.png%22)

![](https://www.notion.so/Quickstart-dd206814f9214605949e2a908d3c2b10#3d354ecc8e6e460f92e4946dbee411bd)

## Admin

튜토리얼을 따라했는데 튜토리얼과는 다르게 브라우저에서 `User List`를 접근할 수가 없었다.

`"detail": "Authentication credentials were not provided."`와 같은 에러메시지가 떠서 뭔가 admin에 연관된 문제인 것 같아, 기본 장고 세팅처럼 admin을 추가해주었다.

```python
# urls.py
from django.contrib import admin
from django.urls    import path

urlpatterns = [
	...
	path('admin/', admin.site.urls)
]
```

세팅을 저장해준 후 `127.0.0.1:8000/admin`으로  접속해 아까 만들어주었던 username과 password로 로그인해주고 다시 `127.0.0.1:8000/users`로 접속하니 위 화면처럼 잘 나오는 것을 확인했다!

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b0a21d2e-2cff-46e3-b239-5127a9347bc0/_2021-01-20__3.50.33.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/b0a21d2e-2cff-46e3-b239-5127a9347bc0/_2021-01-20__3.50.33.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ee350c6-5497-48b2-9546-3c5df5732641/_2021-01-20__3.51.25.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/3ee350c6-5497-48b2-9546-3c5df5732641/_2021-01-20__3.51.25.png)

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea2db998-53a8-40c1-8aae-d8c33dd0c937/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/ea2db998-53a8-40c1-8aae-d8c33dd0c937/Untitled.png)

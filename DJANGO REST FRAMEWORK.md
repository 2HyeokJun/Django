# DJANGO RESTFRAMEWORK TUTORIAL

---------------------

## 1. 설치

(1) 설치 후 ```python3 manage.py migrate```

(2) 관리자 계정 생성

```python
python3 manage.py createsuperuser --email admin@example.com --username admin
```



## 2.```serializers.py```

````python
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
````

* ```HyperlinkedModelSerializer```란?

  기본키가 아닌 하이퍼링크(url)를 사용해 관계를 나타내는 것. 더 RESTful에 좋은 디자인이라고 한다.

  (Notice that we're using hyperlinked relations in this case with `HyperlinkedModelSerializer`.  You can also use primary key and various other relationships, but hyperlinking is good RESTful design.)



## 3. ```views.py```

```python
from django.contrib.auth.models import User, Group
from rest_framework import viewsets
from rest_framework import permissions
from tutorial.quickstart.serializers import UserSerializer, GroupSerializer

class UserViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows users to be viewed or edited.
    """
    queryset = User.objects.all().order_by('-date_joined')
    serializer_class = UserSerializer
    permission_classes = [permissions.IsAuthenticated]

class GroupViewSet(viewsets.ModelViewSet):
    """
    API endpoint that allows groups to be viewed or edited.
    """
    queryset = Group.objects.all()
    serializer_class = GroupSerializer
    permission_classes = [permissions.IsAuthenticated]
```

```viewset```이라 불리는 클래스로 공통 행동들을 묶어서 작성한다.

## 4. ```urls.py```

```python
from django.urls import include, path
from rest_framework import routers
from tutorial.quickstart import views

router = routers.DefaultRouter()
router.register(r'users', views.UserViewSet)
router.register(r'groups', views.GroupViewSet)

# Wire up our API using automatic URL routing.
# Additionally, we include login URLs for the browsable API.
urlpatterns = [
    path('', include(router.urls)),
    path('api-auth/', include('rest_framework.urls', namespace='rest_framework'))
]
```

```view``` 대신 ```viewset```을 사용하면, ```viewset```을 ```router```에 등록하기만 하면 URL을 자동으로 생성할 수 있다.

추가적인 API URL이 필요하면 일반적인 방식으로 URL을 명시적으로 작성하면 된다.


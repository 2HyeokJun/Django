# Django 초기 세팅

1. ```Django``` 시작하기

   ```python
   #cmd
   python -m venv (venvname)
   cd (venvname)
   .\/Scripts/activate.ps1
   ```

2. 가상환경에서 ```framework```설치

   ```python
   #cmd
   pip install django
   pip install djangorestframework
   pip install django-cors-headers
   ```

3. ```Django``` 프로젝트 생성

   ```python
   #cmd
   django-admin startproject (project)
   python manage.py startapp (app)
   ```

4. ```settings.py```에 ```app```과 ```framework``` 추가

   ```python
   #project/settings.py
   ...
   INSTALLED_APPS = [
       ...
   ‘rest_framework’,
   ‘app’,
       ...
   ]
   ...
   ```

5. ```models.py``` 만들기

   ```python
   #app/models.py
   class Board(models.Model):
       title = models.CharField(max_length = 30)
       content = models.CharField(max_length = 500)
       author = models.CharField(max_length = 30)
       created = models.DateTimeField(auto_now_add = True)
   
       def __str__(self):
           return self.title
   ```

6. ```migration```

   ```python
   #cmd
   python manage.py makemigrations
   python manage.py migrate
   ```

7. ```serializers.py``` 만들기

   ```python
   # app/serializers.py
   from rest_framework import serializers
   from .models import Client
   class BoardSerializer(serializers.ModelSerializer):
       class Meta:
           model = Board
           fields = ['title', 'content', 'author', 'created']
   ```

8. ```views.py``` 만들기

   ```python
   #app/views.py
   from django.shortcuts import render
   from django.http import HttpResponse, JsonResponse
   from .models import Board
   from .serializers import BoardSerializer
   from rest_framework.parsers import JSONParser
   
   # Create your views here.
   def address_list(request):
       if request.method == 'GET':
           query_set = Board.objects.all()
           serializer = BoardSerializer(query_set, many = True)
           return JsonResponse(serializer.data, safe = False)
   
       elif request.method == 'POST':
           data = JSONParser().parse(request)
           serializer = BoardSerializer(data = data)
           if serializer.is_valid():
               serializer.save()
               return JsonResponse(serializer.data, status = 201)
           return JsonResponse(serializer.errors, status = 400)
   ```

9. ```views.py```와 ```urls.py``` 연결하기

   ```python
   # project/urls.py
   from django.contrib import admin
   from django.urls import path
   from django.conf.urls import url, include
   from client import views
   
   urlpatterns = [
       path('admin/', admin.site.urls),
       path('board/', views.board_list),
   ]
   ```

10. ```admin```에 ```model``` 추가하기

```python
#app/admin.py
from django.contrib import admin
from .models import Client
# Register your models here.
class ClientAdmin(admin.ModelAdmin):
    pass
admin.site.register(Client, ClientAdmin)
```

11. 시작하기

    ```python
    #cmd
    python manage.py makemigrations
    python manage.py migrate
    python manage.py runserver
    ```

    

--------------------------------

# serializers.py

* 모델의 형태를 내맘대로 조절하고 싶다면?

```python
class MatchInfo(models.Model):
    index = models.AutoField(primary_key = True)
    home_team = models.CharField(max_length = 50)
    away_team = models.CharField(max_length = 50)
    win_odds = models.FloatField()
    draw_odds = models.FloatField()
    lose_odds = models.FloatField()
    def __str__(self):
        return self.home_team
```

​	이런 모델일 때

1. ```odds = ['win_odds', 'draw_odds', 'lose_odds']```로 묶어서 표현하고 싶다

2. ```home_team, away_team```을 ```home, away```로 바꿔서 보여주고 싶다

```python
class MatchInfoSerializer(serializers.ModelSerializer):
    home = serializers.CharField(source = 'home_team')
    away = serializers.CharField(source = 'away_team')
    odds = serializers.SerializerMethodField()
    class Meta:
        model = MatchInfo
        fields = ('home', 'away', 'odds', 'index')
    def get_odds(self, obj):
        odds_array = [obj.win_odds, obj.draw_odds, obj.lose_odds]
return odds_array
```

* model의 모든 값을 가져오는데 다 쓰기 귀찮다면?

```python
class CarSerializer(serializers.ModelSerializer):
	class Meta:
		model = Car
		fields = '__all__'
```

그러면 일부만 '빼고' 싶다면?

```python
class CarSerializer(serializers.ModelSerializer):
	class Meta:
		model = Car
		exclude = ('field1', ...)
```

--------------------------

# models.py

1. ```inspectdb > models.py```할 때 ```ValueError: source code string cannot contain null bytes``` 오류 (VSCODE) : [참고](https://lionontheshore.tistory.com/77)

   

-------

# views.py


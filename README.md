[<p>1.1 Getting Started with Django</p>](#one)
[<p>2.2 URLs and Views</p>](#two)

<a name="one"><h2>1.1 Django Web Framework</h2></a><br>
<a name="two"><h2>1.2 Django Web Framework</h2></a><br>

<h1>Using json Renderer</h1>
<pre>
models.py
---------
from django.db import models

class Student(models.Model):
    name=models.CharField(max_length=100)
    roll=models.IntegerField()
    city=models.CharField(max_length=100)
    
    
serializers.py
--------------
from rest_framework import serializers

class StudentSerializer(serializers.Serializer):
    id=serializers.IntegerField()
    name=serializers.CharField(max_length=100)
    roll=serializers.IntegerField()
    city=serializers.CharField(max_length=100)   
    
admin.py
--------
@admin.register(Student)
class StudentAdmin(admin.ModelAdmin):
    list_display=['id','name','roll','city']

views.py
----------
from django.shortcuts import render
from .models import Student
from .serializers import StudentSerializer
from rest_framework.renderers import JSONRenderer 
from django.http import HttpResponse, JsonResponse

def student_detail(request):
    stu=Student.objects.get(id=1)               #without primary key
    serializer=StudentSerializer(stu)
    json_data=JSONRenderer().render(serializer.data)
    return HttpResponse(json_data, content_type='application/json')

def student_detail2(request):
    stu=Student.objects.all()
    serializer=StudentSerializer(stu, many=True)
    json_data=JSONRenderer().render(serializer.data)
    return HttpResponse(json_data, content_type='application/json')
    
urls.py
-------
    path('getjson/',views.student_detail),
    path('alldata',views.student_detail2),
</pre>

![drf](https://user-images.githubusercontent.com/59610617/130356707-edd020de-678f-4a5e-b904-186241d15511.png)<br>

![drf2](https://user-images.githubusercontent.com/59610617/130356713-46c2de31-6743-4cc7-a75c-7add66ab4c1f.png)<br>

<h1>Using json Response</h1>
In the above example we did'nt use pk to fetch data when we hit getjson, Here we'll use pk. 
<pre>
same code as above just some changes in views & urls

views.py
from django.shortcuts import render
from .models import Student
from .serializers import StudentSerializer
from rest_framework.renderers import JSONRenderer 
from django.http import HttpResponse, JsonResponse

#using pk
def student_detail(request,pk):
    stu=Student.objects.get(id=pk)
    serializer=StudentSerializer(stu)
    json_data=JSONRenderer().render(serializer.data)
    return HttpResponse(json_data, content_type='application/json')

#or using json response // this will reduce the number of code
def student_detail2(request):
    stu=Student.objects.all()
    serializer=StudentSerializer(stu, many=True)
    return JsonResponse(serializer.data, safe=False)
    
urls.py
    path('getjson/<int:pk>',views.student_detail),
    path('alldata',views.student_detail2),
</pre>
getjson will fetch all the data same as above<br>

getjson/pk, this will fetch the data based on pk given.<br>
![drf3](https://user-images.githubusercontent.com/59610617/130356936-a8611b84-2078-47bc-a5eb-4c0ed636368b.png)<br>

<h1>DRF using GenericAPiView</h1>
<pre>
same models.py and admin.py as above given.

serializers.py
--------------
from rest_framework import serializers
from .models import Student

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model=Student
        fields=['id','name','roll','city']
        
views.py
--------
from rest_framework.generics import GenericAPIView
from rest_framework.mixins import ListModelMixin, CreateModelMixin, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin

#List: used to view data
class StudentList(GenericAPIView, ListModelMixin):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)

#Post: used to create data
class StudentCreate(GenericAPIView, CreateModelMixin):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)

#Retrive: to retrive based on pk
class StudentRetrive(GenericAPIView, RetrieveModelMixin):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)


#Put: update every field, if any field is blank it'll give error
class StudentUpdate(GenericAPIView, UpdateModelMixin):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    
    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

#Destroy: to delete data based on pk
class StudentDestroy(GenericAPIView, DestroyModelMixin):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    
    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)

urls.py
    path('studentapi/',views.StudentList.as_view()),
    path('studentCreateapi/',views.StudentCreate.as_view()),
    path('studentRetriveapi/<int:pk>',views.StudentRetrive.as_view()),
    path('studentUpdateapi/<int:pk>',views.StudentUpdate.as_view()),
    path('studentDeleteapi/<int:pk>',views.StudentDestroy.as_view()),
</pre>
<h1>Grouping GenericAPIView</h1>
In the above example we've created url's for every method. In this example we'll group ListModelMixin, CreateModelMixin cuz they don't need pk and RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin needs pk so we'll group them accordangly.
<pre>
views.py
--------
from rest_framework.generics import ListAPIView,CreateAPIView,ListCreateAPIView, RetrieveUpdateAPIView, RetrieveDestroyAPIView, RetrieveUpdateDestroyAPIView

# GROUPING THE METHOD TOGETHER
# pk require 
class LCStudent(GenericAPIView, ListModelMixin, CreateModelMixin):        #LC: List, Create
    queryset=Student.objects.all()
    serializer_class=StudentSerializer

    def get(self, request, *args, **kwargs):
        return self.list(request, *args, **kwargs)
    
    def post(self, request, *args, **kwargs):
        return self.create(request, *args, **kwargs)


#pk not required
class RUDStudent(GenericAPIView, RetrieveModelMixin, UpdateModelMixin, DestroyModelMixin):    #RUD: Retrive, Update, Delete
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    
    def get(self, request, *args, **kwargs):
        return self.retrieve(request, *args, **kwargs)
    
    def put(self, request, *args, **kwargs):
        return self.update(request, *args, **kwargs)

    def delete(self, request, *args, **kwargs):
        return self.destroy(request, *args, **kwargs)

urls.py
--------
path('studentapi/',views.LCStudent.as_view()),
path('studentapi/<int:pk>',views.RUDStudent.as_view()),

Here we just need 2 urls where pk is required and one where there's no need for pk
</pre>

![lc](https://user-images.githubusercontent.com/59610617/130359214-9fddaada-a043-484e-8c9c-abaf28b5e060.png)<br>

![rud](https://user-images.githubusercontent.com/59610617/130359220-911f1b70-9ca6-4748-820c-2db9186b8428.png)<br>

<h1>Grouping</h1>
<pre>
Extends: it extends GenericAPIView, ListModelMixin

views.py
from rest_framework.generics import ListAPIView, CreateAPIView ListCreateAPIView, RetrieveUpdateAPIView, RetrieveDestroyAPIView, RetrieveUpdateDestroyAPIView

Using each one seperately
--------------------------------------------------------
#this will only show a list 
class StudentList(ListAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
   
#this will give post option
class StudentList(CreateAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
   
class StudentRetrive(RetriveAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
   
class StudentUpdate(UpdateAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
   
class StudentUpdate(UpdateAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
 
class StudentDestroy(DestroyAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    
urls.py
Create URL's accordngly, but only forList, Create we don't need pk, and for rest all 3 we need pk , so we'll take <int:pk> in all 3.

Grouping together
--------------------
views.py

class StudentListCreate(ListCreateAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer

#We can either use RetriveUpdate or RetriverDestroy based on our requirement
class StudentRetriveUpdate(RetrieveUpdateAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer

class StudentRetrivDestroy(RetrieveDestroyAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer  
    
#If we need Retrive update destroy all together
class StudentRetriveUpdateDestroy(RetrieveUpdateDestroyAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
   
urls.py
    path('studentapi/',views.StudentListCreate.as_view()),
    path('studentapi/<int:pk>',views.StudentRetriveUpdateDestroy.as_view()),
</pre>
<h3>List, Create</h3>

![list,create](https://user-images.githubusercontent.com/59610617/130360537-c3730e80-dcba-4219-bfcb-81892baacc99.png)<br>

<h3>Retrive, Update, Delete</h3>

![retrive,update,delete](https://user-images.githubusercontent.com/59610617/130360547-f1be61bf-eda6-4371-877d-74b00ef2a1f9.png)<br>

<h1>DRF using ViewSet Class</h1>
list(): get all records.<br>
retrive(): get single record.<br>
create(): create/insert record.<br>
update(): update record completely.<br>
partial_update(): update record partially.<br>
destroy(): delete record.<br>
<pre>
models.py and admins.py same as above example

serializers.py
--------------
from rest_framework import serializers
from .models import Student

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model=Student
        fields=['id','name','roll','city']

views.py
--------
from django.shortcuts import render
from rest_framework.response import Response
from .models import Student
from .serializers import StudentSerializer
from rest_framework import status
from rest_framework import viewsets


urls.py
---------
from viewsetapi import views
from rest_framework.routers import DefaultRouter

class StudentViewSet(viewsets.ViewSet):
    def list(self, request):
        stu=Student.objects.all()
        serializer=StudentSerializer(stu, many=True)
        return Response(serializer.data)

    def retrieve(self, request, pk=None):
        id=pk
        if id is not None:
            stu=Student.objects.get(id=id)
            serializer=StudentSerializer(stu)
            return Response(serializer.data)

    def create(self, request):
        serializer=StudentSerializer(data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response({'msg':'Data Created'},status=status.HTTP_201_CREATED)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def update(self, request, pk):
        id=pk
        stu=Student.objects.get(pk=id)
        serializer=StudentSerializer(stu,data=request.data)
        if serializer.is_valid():
            serializer.save()
            return Response({'msg':'Compltete Data Updated'})
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

    def partial_update(self, request, pk):
        id=pk
        stu=Student.objects.get(pk=id)
        serializer=StudentSerializer(stu,data=request.data, partial=True)
        if serializer.is_valid():
            serializer.save()
            return Response({'msg':'Partial Data Updated'})
            return Response(serializer.errors)

    def destroy(self, request, pk):
        id=pk
        stu=Student.objects.get(pk=id)
        stu.delete()
        return Response({'msg':'Data Deleted'})

urls.py
--------
from django.urls import path, include
from viewsetapi import views
from rest_framework.routers import DefaultRouter

#creatting router object
router=DefaultRouter()

#register studentviewset with router
router.register('studentapi',views.StudentViewSet,basename='student')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api',include(router.urls)),
]
</pre>
<h3>On http://127.0.0.1:8000/apistudentapi/  this link it'll provide list() and create() i.e we can see list and add data:</h3>

![api](https://user-images.githubusercontent.com/59610617/130361520-62c45ffb-569c-4ba5-9b9e-88179587db8f.png)<br>

![api_link](https://user-images.githubusercontent.com/59610617/130361549-7f8f4045-bed7-4087-bf18-e1364df086d5.png)<br>

![api_link2](https://user-images.githubusercontent.com/59610617/130361550-4c981be7-e423-4e1e-97bb-b1f87560fae2.png)<br>

<h3>On http://127.0.0.1:8000/apistudentapi/3  this link it'll provide retrive(), update(), partial_update(), destroy() all based on pk. Here even if we did'nt give <int:pk> still it gives option for pk.</h3>

![api_link3](https://user-images.githubusercontent.com/59610617/130361551-bbed80f7-2573-44eb-918f-498863cbe8c2.png)

List of attributes in viewset:
<pre>
This is only for list we can do same with every method viewset provide which we saw above

class StudentViewSet(viewsets.ViewSet):
  def list(self, request):
          print(f"LIST\n Basename:{self.basename}\n Action:{self.action}\n Detail:{self.detail}\n Suffix:{self.suffix}\n Name:{self.name}\n Description:{self.description}")

Output:
LIST
 Basename:student
 Action:list     
 Detail:False    
 Suffix:List     
 Name:None       
 Description:None
 
Using these attributes we can perform particular action which we need.
Example do something when action == List these kind of logics we can do.
</pre>

PENDING
#GETTING ALL METHODS USING MODEL VIEWSET
# class StudentModelViewSet(viewsets.ModelViewSet):
#     queryset=Student.objects.all()
#     serializer_class=StudentSerializer

<h1>Authentication in DRF</h1>
Django provides:<br>
1)BasicAuthentication.<br>
2)SessionAuthentication.<br>
3)TokenAuthentication.<br>
4)RemoteUserAuthentication.<br>
5)CustomAuthentication.<br>

<h1>Permission</h1>
<li>Permissions are used to grant or deny access for different classes of users to different parts of the API.</li>
<li>Permission check are always run at the very start of the view, before any other code is allowed to proceed.</li>
<li>Permission checks will typically use the authentication information in the request.user and request.auth properties to determine if the incoming request should be permitted.</li>
<li>So we can use request.user and request.auth to check the one who is loggin in.</li>
<b>Django provides:</b><br>
1)AllowAny  = have permisiion to all users and everyone without password.<br>
2)IsAuthentication = every users including superuser, normal user, staff user can loging with there username and pswd, and deny if he's unauthenticated. This is suatable if you want your API to only be accessible to registerd users.<br>
3)IsAdminUser = Only allowed to users whos user.is_staff =  TRUE, this is suatable if you want your API to be accessible to a subset of trusted administrators.<br>


<h1>Basic Authentication in ViewSet</h1>
<li>We need username, password for basic authentication. This one is generally only appropriate for testing purpose not for production.</li>
<li>Note: If you provide basic authentication in production you must ensure that your API is only available over https. </li>
<li>You should also ensure that your API clients will always re-request the username and password at login, and will never store those details to persistent storage.</li>
<pre>

</pre>

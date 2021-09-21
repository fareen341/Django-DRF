[<p>1.1 Getting Started with Django</p>](#one)
[<p>2.2 URLs and Views</p>](#two)

NOTE: for using rest_framework it must be installed and included in the installed_apps

<a name="one"><h2>1.1 Django Web Framework</h2></a><br>
<a name="two"><h2>1.2 Django Web Framework</h2></a><br>

<h1>INTRODUCTION</h1>
<b>XML</b><br>
<p>Mostly use for configuration as in db configuration. We need namespace in xml.</p>
<pre>
&lt;emps&gt;
    &lt;emp&gt;
        &lt;id&gt;1&lt;/id&gt;
        &lt;name>Fareen&lt;/name&gt;
    &lt;/emp&gt;
    &lt;emp&gt;
        &lt;id&gt;2&lt;/id&gt;
        &lt;name&gt;Annu&lt;/name&gt;
    &lt;/emp&gt;
&lt;/emps&gt;
</pre>
<p>Postman has code on right side which give code to fetch data, on code <> symbol</p>



<h1>Using json Renderer</h1>
<pre>
settings.py
-----------
INSTALLED_APPS = [
    'rest_framework',
]

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

<h1>Model ViewSet DRF</h1>
<pre>
Same model serializer, model, admin as above.

This will provides the entire CRUD DRF 
---------------------------------------
views.py
--------

from django.shortcuts import render
from rest_framework.response import Response
from .models import Student
from .serializers import StudentSerializer
from rest_framework import status
from rest_framework import viewsets

class StudentModelViewSet(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    
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

This will provide read only DRF, will only give readonly(list and retrive), can see the whole list
and when we hit with pk we can only see that particular list, we cannot post,update, delete this is real only
--------------------------------

views.py
--------
from django.shortcuts import render
from rest_framework.response import Response
from .models import Student
from .serializers import StudentSerializer
from rest_framework import status
from rest_framework import viewsets

class RetriveListStudent(viewsets.ReadOnlyModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer

urls.py
-------
same urls as above just different view name.
</pre>

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
4)IsAuthenticationOrReadOnly - authenticated user can perform write permission and other all users(anoymous) can read only.<br>
5)DjangoModelPermission - this permission is tired with django auth, like we create model we get change, add, view permission on admin interface. Giving the select permission as in just post or put or just post and put etc. Here unauthenticated user cannot view data.<br>
For django model permission we need to login on admin and give permissionon particular model which we want the particular user to access, for giving only add student permission allow that user add permission on django admin and when we login we can see that user will have read and add permission
Note: Every user after login have view data permission, which is default.
SuperUser have all permissions, but the user whos staff status is true for this user also we've to give model permission.<br>
6)DjangoModelPermissionOrAnonReadOnly - it is same as DjangoModelPermission except it allow unauthenticated user to view data.<br>
In django model permission we must give login credential even for view data but in DjangoModelPermissionOrAnonReadOnly anoynmous users can view data. So when we hit /api url anybody can view the data but cannot delete,update and add data.<br>
7)DjangoObjectPermissions - 
8)CustomPermission:

Third Party Permissions:

![thirdparty](https://user-images.githubusercontent.com/59610617/130565201-cafa439d-50e9-4422-84a9-e69a0ee27e2e.png)<br>

<h1>Authentication</h1>
<li>Basic Authentication</li>
<li>Session AUthentication</li>
<li>Token Authentication</li>
<li>RemoteUser Authentication</li>
<li>Custome Authentication</li>

Third party Authentication:

![auth](https://user-images.githubusercontent.com/59610617/130578972-9c4d4964-7e64-4ea3-ae16-2eba9bec92d6.png)<br>

In all of this we'll see "JSON Web Token Authentication"<br>

<h1>Basic Authentication in ViewSet</h1>
<li>We need username, password for basic authentication. This one is generally only appropriate for testing purpose not for production.</li>
<li>Note: If you provide basic authentication in production you must ensure that your API is only available over https. </li>
<li>You should also ensure that your API clients will always re-request the username and password at login, and will never store those details to persistent storage.</li>
<pre>
We can create API in any way using viewset, modelviewset, genericviewset etc and can add authentication and permission in all of them
Here i'll use model viewset, This authentication is for class based, for function based it's slight different

With authentication we have to give permissions as well.

urls.py
-------
same urls as above just different view name.

views.py
---------
from django.shortcuts import render
from rest_framework.response import Response
from .models import Student
from .serializers import StudentSerializer
from rest_framework import status
from rest_framework import viewsets

#For authentication
from rest_framework.authentication import BasicAuthentication

#For permission
from rest_framework.permissions import IsAuthenticated, AllowAny, IsAdminUser

class StudentModelViewSet(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    authentication_classes=[BasicAuthentication]
    permission_classes=[IsAuthenticated]
 
Globally authentication, permission for all the classes:
If we want to give this permission to every model then we can define it globally:

settings.py
REST_FRAMEWORK={
    'DEFAULT_AUTHENTICATION_CLASSES':['rest_framework.authentication.BasicAuthentication'],
    'DEFAULT_PERMISSION_CLASSES':['rest_framework.permissions.IsAuthenticated']
}

If for a perticular class we don't need the default settings.py authentication, pemission
class 1{
    //will have authentication, permission coming from settings.py
    }
 
class 2{
    //will have authentication, permission coming from settings.py
    }
    
class 3{
    authentication_classes=[BasicAuthentication]
    permission_classes=[AllowAny]
    
class 1, class2 will have global permission and class 3 will have it's own permission which is allowany.

We can also give many permission to one class:
permission_classes=[AllowAny, IsAdminUser...etc] Same goes for authentication as in [SesstionAuthentication, BasicAuthentication...etc]
</pre>

![authentication](https://user-images.githubusercontent.com/59610617/130386758-e194fd40-6f51-4bb3-828d-3b58721392a2.png)<br>

<h1>Session Authentication</h1>
<li>This authentication scheme use djangos's default session backend for authentication. This one is appropriate for AJAX clients that are running in the same session context as your website.</li> 
<li>If successfully authenticated, this authentication provides the following credential:</li>
1)request.user will be django user instance.<br>
2)request.auth will be None.<br>
3)unauthenticated responses: return HTTP 403 forbidden response.<br>
<li>Tf you're using an AJAX style API with SessionAuthentication, you'll need to make sure you include a valid CSRF token for any "unsafe" HTTP method calls, such as PUT, PATCH, POST or DELETE requests.</li>
<pre>
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
    path('auth/',include('rest_framework.urls',namespace='rest_framework'))             #will provide login/logout functionality
]

views.py
class StudentModelViewSet(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    authentication_classes=[SessionAuthentication]
    permission_classes=[DjangoModelPermissions]
</pre>

![session](https://user-images.githubusercontent.com/59610617/130560292-bcfd47da-42f5-4c89-878b-d556889c61ea.png)<br>

<h1>Custom permission example</h1>
<pre>
Same model, serializer, urls in above example:

Step 1: create .py file or any name, use CustomPermission for easily reference
from rest_framework.permissions import BasePermission

class MyPermission(BasePermission):
    def has_permission(self, request, view):
        if request.method=='GET':
            return True
        return False
        
Step 2:
views.py
from custum_permissions import MyPermission

class StudentModelViewSet(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    authentication_classes=[SessionAuthentication]
    permission_classes=[MyPermission]

We can give any logic as in, in a blog post the person who created blog have edit,delete permission etc
</pre>

<h1>Token Authentication</h1>
<li>This authentication scheme uses a simple token-based HTTP authentication scheme. Token authentication is appropriate for client-server setups, such as native desktop and mobile clients.</li>
<li>If you use this authentication in production you must ensure that your API is only available over https.</li>
<b>GENERATE TOKEN:</b>
<li>Using Admin application</li>
<li>Using django manage.py command >python manage.py drf_create_token <username>: this command will return Api token for the given user or create a token if token does'nt exist for user.</li>
<li>By exposing an API endpoint</li>
<li>Using signals</li>
<pre>
Same models.py, admin.py, serializers.py, urls.py as above

settings.py
include rest_framework, rest_framework.authtoken in installed apps.

Step1:
settings.py
INSTALLED_APPS = [
    'rest_framework',
    'rest_framework.authtoken',
]

Step2:
after including token authentication we need to run migrate commands cuz it'll create a table Token

Step3:
generate token 
---------------
1)Using Admin: go to admin ->Token ->Select user for whom we need to create token -> Save 
Now token will be generated for that particular user

2)Using manage.py: python manage.py drf_create_token fareen
If token exist will return else create for that user.

3)By exposing an API endpoint: user create token in this case:
Same models.py, admin.py, serializers.py as above

Step 1:

urls.py
from rest_framework.authtoken .views import obtain_auth_token

router=DefaultRouter()

router.register('studentapi',views.Student,basename='student')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('',include(router.urls)),
    path('auth/',include('rest_framework.urls',namespace='rest_framework')),
    path('gettoken/',obtain_auth_token)
]

Step 2:
For this we need httpie, install it using pip >pip install httpie
httpie: it is command like HTTP client, its goal is to make CLI interation with web services as human friendly as possible.

On cmd: http POST http://127.0.0.1:8000/gettoken/ username="admin" password="admin"

If the user request for token and if token exist it'll return and if not it'll create
Using this token they can access the api

We can use Custom authentication and in that we can return email, user_id etc from server and when user ask for token he'll get his email,user_id as well in return, for this we have to create another .py file and include in views and give url for the same.

4)Using signals: whenever user created generate token
Same models.py, admin.py, serializers.py as above

urls.py
router=DefaultRouter()

router.register('studentapi',views.Student,basename='student')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('',include(router.urls)),
    path('auth/',include('rest_framework.urls',namespace='rest_framework')),
]

models.py
from django.conf import settings
from django.db.models.signals import post_save
from django.dispatch import receiver
from rest_framework.authtoken.models import Token

@receiver(post_save,sender=settings.AUTH_USER_MODEL)
def create_auth_token(sender, instance=None, created=False, **kwargs):
    if created:
        Token.objects.create(user=instance)
        
Now if any new user login there token will generated automatically

Step 4:
Using this tokens t access our API
----------------------------------

Permission class will be same for all authentication(basic, session, token).

Sending request using httpie:
http http://127.0.0.1:8000/studentapi/

Will get the data cuz in views.py we didn't provide any permission so it is bydefault AllowAny everyone can access.

views.py
from rest_framework.authentication import TokenAuthentication
from rest_framework.permissions import IsAuthenticated

class Student(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    authentication_classes=[TokenAuthentication]
    permission_classes=[IsAuthenticated]
    
Now we cannot access the api using the above url it'll deny the request
To access this api use >http http://127.0.0.1:8000/studentapi/ 'Authorization:Token e7670717121b92145d0d5bcb1b3ed2ffcbfb591e'            #this is uses's key 

To send post request:
http -f POST http://127.0.0.1:8000/studentapi/  name="sam" roll=123 city="Mumbai" 'Authorization:Token e7670717121b92145d0d5bcb1b3ed2ffcbfb591e' 

Using the token provided the user is performing the POST operation

To send put request:
http PUT http://127.0.0.1:8000/studentapi/4/  name="neha" roll=12 city="Mumbai" 'Authorization:Token e7670717121b92145d0d5bcb1b3ed2ffcbfb591e'

To send delete request:
http DELETE http://127.0.0.1:8000/studentapi/4/ 'Authorization:Token e7670717121b92145d0d5bcb1b3ed2ffcbfb591e'
</pre>

<h1>Custom authentication</h1>
<pre>
Same modes.py, serializers.py, admin.py and include rest_framework in settings.py

urls.py
from rest_framework.routers import DefaultRouter
from rest_framework.authtoken .views import obtain_auth_token
# from tokenapp.auth import CustomAuthToken

router=DefaultRouter()

router.register('studentapi',views.Student,basename='student')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('',include(router.urls)),
    path('auth/',include('rest_framework.urls',namespace='rest_framework')),
    
create .py file for custom authentication(custom_auth.py) can give any name:
from rest_framework.authentication import BaseAuthentication
from django.contrib.auth.models import User
from rest_framework.exceptions import AuthenticationFailed

class CustomeAuthentication(BaseAuthentication):
    def authenticate(self, request):
        username=request.GET.get('username')
        if username is None:
            return None

        try:
            user=User.objects.get(username=username)
        except User.DoesNotExist:
            raise AuthenticationFailed('No such user')
        return (user,None)
        
views.py
from rest_framework.permissions import IsAuthenticated,AllowAny
from tokenapp.custom_auth import CustomeAuthentication

class Student(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    authentication_classes=[CustomeAuthentication]
    permission_classes=[IsAuthenticated]
 
For this we'll hit http://127.0.0.1:8000/studentapi/username=admin  OR
For delete, updtae http://127.0.0.1:8000/2/studentapi/username=admin
</pre>

<h1>JSON Web Token Authentication(JWT)</h1>
<li>JWT is a fairly new standsrd which can be used for token based authentication. Unlike the built-in token authentication scheme, JWT authentication does'nt need to use a database to validate a token.</li>
<li>So one benefit of this JWT token is in case of normal tokem authentication when there are 100 users all accessing api together then database perform 100 request together cuz token is stored in table in normal token authentication, in case of JWT token there's no database table generated. https://jwt.io/</li><br>

<b>Simple JWT</b>
We need to install it before using it:<br>
>pip install djangorestframework-simplejwt

To install it globally, meaning to make it common for all the classes the include it inside the settings.py<br>
<pre>
REST_FRAMEWORK={
    'DEFAULT_AUTHENTICATION_CLASSES':(
        'rest_framework_simplejwt.authentication.JWTAuthentication',
        )
}
</pre>
Creating jwt project
<pre>
install simple jwt >pip install djangorestframework-simplejwt

include 'rest_framework', 'jwtapp' in installed apps

same models.py, admin.py, serializers.py

urls.py
from rest_framework.routers import DefaultRouter
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshSlidingView,TokenVerifyView

router=DefaultRouter()

router.register('studentapi',views.Student,basename='student')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('',include(router.urls)),
    path('gettoken/',TokenObtainPairView.as_view(),name='tokenobtain'),
    path('refreshtoken/',TokenRefreshSlidingView.as_view(), name='tokenrefresh'),
    path('verifytoken/',TokenVerifyView.as_view(), name='tokenverify')   
]

TokenObtainPairView: return access and referesh token
TokenRefreshSlidingView: return refresh token generate

TO GENERATE TOKEN:
-------------------
we'll use gettoken url which is in urls:
user will request token like this
http POST http://127.0.0.1:8000/gettoken/ username="admin" password="admin"

Output:
{
    "access": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.ey",
    "refresh": "eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ"
}
This will return access and refresh token, access token has default validity for 5 minutes after 5 minutes it'll expire, then user have to use refresh token to generate access token to access api he don't need to give username, password again he can use refresh token. refresh token has one day validity

Using the access token to access api:
http POST http://127.0.0.1:8000/verifytoken/ token="copy paste the access token"

If the access token expired:
http POST http://127.0.0.1:8000/refreshtoken/ refresh="copy paste the refresh token"

This will return the access token

ACCESS TOKEN:
-------------
views.py
from rest_framework.permissions import IsAuthenticated
from rest_framework_simplejwt.authentication import JWTAuthentication

class Student(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    authentication_classes=[JWTAuthentication]
    permission_classes=[IsAuthenticated]

Now to access this api user need JWT Token

To access request:
http http://127.0.0.1:8000/studentapi/ 'Authorization:Bearer copy and paste access token'

To post request:
http -f POST http://127.0.0.1:8000/studentapi/ name=annu roll=2 city=Mumbai 'Authorization:Bearer paste access key'

To update:
http PUT http://127.0.0.1:8000/studentapi/2/ name=sam roll=20 city=Mumbai 'Authorization:Bearer paste access key'

http DELETE http://127.0.0.1:8000/studentapi/2/ 'Authorization:Bearer 'Authorization:Bearer paste access key'

We should create a application for this token, users will not use command line for access to know more read the documentation of simple jwt 
</pre>
To override the simplejwt methods:
To increase the lifetime of the access token:
To get refresh token every time when we use refresh token, make this option TRUE, by default it is FALSE(meaning everytime we request for access token using refresh token, it only returns the access token, but is this option is TRUE it'll return refresh token every time)
<pre>
SIMPLE_JWT = {
    'ACCESS_TOKEN_LIFETIME':timedelta(minutes=10),      #lifetime of access token
    'REFRESH_TOKEN_LIFETIME':timedelta(days=2),         #lifetime of refresh token
    'ROTATE_REFRESH_TOKEN':True                         #to generate refresh token everytime
}
</pre>

<h1>Throttling</h1>
<li>Your API might have a restrictive throttle for unauthenticated requests, and a less restrictive throttle for authentication requests.</li>
<pre>
For globally
in settings.py

REST_FRAMEWORK = {
    'DEFAULT_THROTTLE_CLASSES':[
        'rest_framework.throttling.AnonRateThrottle',
        'rest_framework.throttling.UserRateThrottle'
    ],
    'DEFAULT_THROTTLE_RATES':{
        #rates can be include in second, minute, hour or day
        'anon':'100/day',       #only unauthenticated users can only login for 100 times per day 
        'user':'1000/day'       #authenticated users can login only 1000 times per day
    }
}
</pre>

<li>AnonRateThrottle: The AnonRateThrottle will only ever throttle unauthenticated users. The IP address of the incoming request is used to generate a unique key to throttle against.</li>
<li>UserRateThrottle: The UserRateThrottle will throttle users to a given rate of requests across
the API. The user id is used to generate a unique key to throttle against. Unauthenticated requests will fall back to using the IP address of the incoming request to generate a unique key to throttle against.</li>
<li>ScopedRateThrottle: use it when we want to restrict some part of api.</li>

Practical:
<pre>
same serialisers.py, models.py, admins.py, settings.py as above example

urls.py
from viewsetapi import views
from rest_framework.routers import DefaultRouter

router=DefaultRouter()
router.register('studentapi',views.StudentModelViewSet,basename='student')

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api',include(router.urls)),
    path('auth/',include('rest_framework.urls',namespace='rest_framework'))
]

views.py
from rest_framework.throttling import AnonRateThrottle, UserRateThrottle

class StudentModelViewSet(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    authentication_classes=[SessionAuthentication]
    permission_classes=[IsAuthenticated]
    throttle_classes = [AnonRateThrottle, UserRateThrottle]
 
settings.py 
Note: this throttle is for all the classes, it's global
REST_FRAMEWORK={
    'DEFAULT_THROTTLE_RATES':{
        'anon': '2/day',
        'user': '5/day',
    }
}
</pre>
When anoynmous user request 2 times, when it is 3rd time they'll get throttle msg. Same when authenticated user request 5 times, on 6th time they'll msg. To check this throttle refresh browser 5 times and on 6th they'll get request throttle msg.
<pre>
{
    "detail": "Request was throttled. Expected available in 86388 seconds."
}
</pre>

To give different rate for different classes
<pre>
create (mythrottle.py(give any name).
from rest_framework.throttling import UserRateThrottle

class AnnuRateThrottle(UserRateThrottle):
    scope = "annu"
    
settings.py
REST_FRAMEWORK={
    'DEFAULT_THROTTLE_RATES':{
        'anon': '2/day',
        'annu': '2/minute'
    }
}

views.py
from rest_framework.throttling import AnonRateThrottle, UserRateThrottle
from viewsetapi.throttle import AnnuRateThrottle

class StudentModelViewSet(viewsets.ModelViewSet):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    authentication_classes=[SessionAuthentication]
    permission_classes=[IsAuthenticatedOrReadOnly]
    throttle_classes = [AnonRateThrottle, AnnuRateThrottle]
</pre>
On 3rd request it'll give throttle message:
<pre>
{
    "detail": "Request was throttled. Expected available in 54 seconds."
}
 
Likewise we can give throttle for different class
</pre>

<h1>ScopedRateThrottle</h1>
Using this we can give different throttle to diffrent classes.
<pre>
urls.py
    path('studentapi/',views.StudentList.as_view()),
    path('studentapipost/',views.StudentCreate.as_view()),                 
    path('studentapi/<int:pk>/',views.StudentRetrive.as_view()),
    
settings.py
REST_FRAMEWORK={
    'DEFAULT_THROTTLE_RATES':{
        'viewstudent': '2/minute',
        'poststudent': '1/minute'
    }
}

views.py
from rest_framework.generics import ListAPIView,CreateAPIView,ListCreateAPIView, RetrieveUpdateAPIView, RetrieveDestroyAPIView, RetrieveUpdateDestroyAPIView, RetrieveAPIView
from rest_framework.throttling import ScopedRateThrottle

class StudentList(ListAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    throttle_classes = [ScopedRateThrottle]
    throttle_scope = 'viewstudent'

class StudentCreate(CreateAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    throttle_classes = [ScopedRateThrottle]
    throttle_scope = 'poststudent'
    

class StudentRetrive(RetrieveAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    throttle_classes = [ScopedRateThrottle]
    throttle_scope = 'viewstudent'

after 2 request in list, retrive we'll get throttle deny msg: and for post we'll only get 1 request per minute.
Likewise we can give for delete and update
</pre>

<h1>Filtering in rest api</h1>
To filter based on user, example fiter based on admin
<pre>
serialializers.py
from rest_framework import serializers
from .models import Student

class StudentSerializer(serializers.ModelSerializer):
    class Meta:
        model=Student
        fields=['id','name','roll','city','passby']
        
models.py
from django.db import models

class Student(models.Model):
    name=models.CharField(max_length=50)
    roll=models.IntegerField()
    city=models.CharField(max_length=50)
    passby=models.CharField(max_length=10)

urls.py
from functionbasedapp import views

urlpatterns = [
    path('admin/', admin.site.urls),
    path('studentapi/',views.StudentList.as_view()),
]

views.py
class StudentList(ListAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    def get_queryset(self):
        user = self.request.user            
        return Student.objects.filter(passby=user)          #filter based on user
 
If the passby is admin, it'll filter by admin.
This scenario maybe usefull in book shop website where the users who publish the book can see only the books published by him.
</pre>
Data:

![data](https://user-images.githubusercontent.com/59610617/130651528-5aba1bf7-c307-4eaf-b061-dc4ac8d30992.png)<br>

<h3>The current user which is logged in is fareen so it'll filter based on current user which is fareen</h3>

![filter](https://user-images.githubusercontent.com/59610617/130651628-56a26338-9e02-4744-9edf-1976100927cb.png)<br>

<h2>Filter based on city</h2>
<pre>
Same serializers.py, models.py, urls.py and admin.py as above eg

Step 1: for this we need filter installed >pip install django-filter

Step 2: settings.py in installed apps
    'rest_framework',
    'django_filters'            

Step 3:giving it globally
settings.py
REST_FRAMEWORK={
    'DEFAULT_FILTER_BACKENDS':{
        'django_filters.rest_framework.DjangoFilterBackend': '2/minute'
    }
}

Step 3: views.py
class StudentList(ListAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    filterset_fields = ['city']

If we want filter on every class:
remove the code in settings.py and add it in views.py

from django_filters.rest_framework import DjangoFilterBackend
class StudentList(ListAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    filter_backends = [DjangoFilterBackend]
    filterset_fields = ['city']         
    
ilterset_fields = ['name', 'city'] if we want to filter according to name and city
We can also search using: http://127.0.0.1:8000/studentapi/?name=neha&city=Mumbai
Now on browser's filter option we'll get name and city two option
</pre>
Now when we refresh page we'll get filter option on the browser<br>

![filter](https://user-images.githubusercontent.com/59610617/130720329-f8be5e46-e386-41e1-8113-b500cf8e701a.png)<br>

<h1>Search Filter</h1>
We need char data for search filter to work.
<pre>
same urls.py, models, admins, serializers as above

views.py
from rest_framework.filters import SearchFilter

class StudentList(ListAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    filter_backends = [SearchFilter]
    search_fields = ['name','city']

Now we'll get one search filter option which we can use to search according to name OR city not like above example name & city
</pre>

![search](https://user-images.githubusercontent.com/59610617/130721572-82521106-1629-4390-a7da-f5d55021d227.png)<br>

<b>Regex in search filter</b><br>
1)'^': startswith search.<br>
2)'=': exact match.<br>
3)'@': full-text search(currently support only on postgres sql).<br>
4)'$': regex search.<br>
<pre>
class StudentList(ListAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    filter_backends = [SearchFilter]
    search_fields = ['^name']
</pre>
<b>To change the default serach= on browser</b>
<pre>
Same views.py

settings.py
REST_FRAMEWORK={
    'SEARCH_PARAM':'q'
}

Now the default http://127.0.0.1:8000/studentapi/?search=neha will change to 
http://127.0.0.1:8000/studentapi/?q=neha
</pre>

<h1>Ordering Filter</h1>
<pre>
views.py
from rest_framework.filters import OrderingFilter

class StudentList(ListAPIView):
    queryset=Student.objects.all()
    serializer_class=StudentSerializer
    filter_backends = [OrderingFilter]
</pre>
Here we did not give ordering_fields so we'll get order by every fileld which looks like below:

![ordering](https://user-images.githubusercontent.com/59610617/130893328-b5a455a3-b9fa-4cd4-ae78-733dde29adda.png)<br>

After giving "ordering_fileds = ["name"]" we'll get order by name asc and desc.<br>





<h1>Permission in Function based view</h1>
Giving Permission in function based view is slightly different than class based




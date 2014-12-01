Building a RESTful API with Django REST framework
=====================================

API's turned to be the heart of every application in our time. With the rise of social media, API's are being developed at a faster pace and gaining a lot of attention. Gone are the days where RPC architectures like CORBA and XML-RPC are used to enable the exchange of information and REST has taken its place making its mark in getting the things pretty straight forward.

In this post we will get to know how to build a Restful API for an application using a Django REST framework. Before dipping ourselves in the API building stuff, let us acquire a picture on REST.

REST
-------

In simple words REST is a web architecture which lives on top of HTTP. It need not abide to HTTP but if we use HTTP as the base, it makes the process simple and simplifies the understanding of the API. It allows the clients and servers to communicate with each other in all the possible ways. REST is a resource based on which we use an HTTP verb to dictate what operation we want to do on a URI. In REST we represent the resource data in JSON and XML format. An API is called a Restful API if it adheres to all the interaction constraints of REST.

In a nutshell, here are the six interaction constraints and what they imply:

1. Uniform interface: It is a defined interface between client and server in which URIs are resource names and http verbs are the actions to take on the resources. Eg. GET, PUT, POST, DELETE.
2. Client-server: It enforces the separation of concerns in the form of a client-server architecture.
3. Stateless: The server must not contain the client state. Each message should be self descriptive and each message must contain enough information and context.
4. Cacheable: Server representations should be cacheble. It helps the service or the consumer to reuse the response for later requests.
5. Layered system: It should comprise of multiple layers, and no one layer can "see past". It results in an increase of scalability.
6. Code on demand: This is an optional constraint. In means restful service logic should be transferable to the client. 


So, if any the constraints are not obeyed except the sixth optional constraint its no more called a RESTful API.

Django Rest framework
-----------------------
Django REST framework makes the process of building web API's simple and flexible. With its batteries included it won't be a tedious task to create an API. Before creating our own API let us look into some vital parts of the REST framework
The important parts of Django REST framework are:

* Serialization and Deserialization: 
The first part in the process of building an API is to provide a way to serialize and deserialize the instances into representations. Serialization is the process of making a streamable representation of the data which will help in the data transfer over the network. Deserialization is its reverse process. In our process of building an API we render data into JSON format. To achieve this Django REST framework provides JSONRenderer and JSONParser. 
JSONRenderer renders the request data into JSON, using utf-8 encoding and JSONParser parses the JSON request content.

Without dipping inside the technicalities let us see how it takes place.

Serializing data with JSONRenderer

        >>> serializer.data
        {'cal_id': u'2', 'username': u'tester'}
        >>> content = JSONRenderer().render(serializer.data)
        >>> content
        '{"cal_id": "2", "username": "tester"}'


Deserializing data with JSONParser:

Firstly, we parse a stream into Python native datatypes with BytesIO and then we will restore those native datatypes into to a fully populated object instance afterwards.

        >>> content
        '{"cal_id": "2", "username": "tester"}'
        >>> stream = BytesIO(content)
        >>> data = JSONParser().parse(stream)
        >>> data
        {u'username': u'tester', u'cal_id': u'2'}

* Requests and Responses

Requests and responses are the essential parts of Django REST framework which provides flexibility while parsing.


* Request objects:

The request object in REST framework is more abilities. The attribute request.DATA in Request object is similar to Request.POST added with the capabilities of handling arbitrary data which works for 'POST', 'PUT' and 'PATCH' methods. And also we can use the different attributes like request.FILES, request.QUERY_PARAMS, request.parsers, request.stream which will help in dealing with the request a hassle free task.

Response objects:

The Response object helps to render the correct content type as requested by the client unlike the normal HttpResponse.

Syntax: Response(data, status=None, template_name=None, headers=None, content_type=None)

Status Codes: 
Instead of using the normal HTTP status codes, we will make use of explicit identifiers for every status code to make the process of reading the status code much simple. Here are some status codes that we represent in a REST framework we use normally.

    HTTP_200_OK
    HTTP_201_CREATED
    HTTP_400_BAD_REQUEST
    HTTP_401_UNAUTHORIZED
    HTTP_403_FORBIDDEN

* Views:
APIView is a view provided by Django Rest framework which subclasses the Django's view class. If we use this view the requests passed to the handler methods will no more Django's HttpRequest instances and they will be REST framework's Request instances.
And while dealing with the responses they work to set the correct renderer on the response. The process of authentication and the permissions stuff also will be dealt with it.

Creating an API with Django REST framework
-------------------------

Using all the above stuff lets us build a simple API which gives the details of a user. The procedure goes the following way:

We will be using Django's built in user in this example. We will create a serializers.py file where we create serializers which are similar to Django forms. Just like model forms we have got model serializers here, which will stop replication of code. And we create a view which lists all the users, creates a new user, retrieve a user and update a user.

1. Here we go with the installation with virtual environment activated:

    pip install djangorestframework

2. Create a Django project and an app. I created it with a name rest_example and restapp

    django-admin.py startproject rest_example
    cd rest_example
    python manage.py startapp restapp

3. Add rest_framework and the app name to the installed apps.

    INSTALLED_APPS = (
        ...
        'rest_framework',
        'restapp'
    )
4. Run syncdb command

    python manage.py syncdb

4. Create a serializers.py which should look like the below way.

    serializers.py

    from django.contrib.auth.models import User

    from rest_framework import serializers


    class UserSerializer(serializers.ModelSerializer):
        class Meta:
            model = User
            fields = ('id', 'username', 'first_name', 'last_name', 'email')

5. Create a views.py for listing all users, create a new user and so on.

    from django.contrib.auth.models import User

    from restapp.serializers import UserSerializer
    from django.http import Http404
    from rest_framework.views import APIView
    from rest_framework.response import Response
    from rest_framework import status


    class UserList(APIView):
        """
        List all users, or create a new user.
        """
        def get(self, request, format=None):
            users = User.objects.all()
            serializer = UserSerializer(users, many=True)
            return Response(serializer.data)

        def post(self, request, format=None):
            serializer = UserSerializer(data=request.DATA)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data, status=status.HTTP_201_CREATED)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

        def delete(self, request, pk, format=None):
            user = self.get_object(pk)
            user.delete()
            return Response(status=status.HTTP_204_NO_CONTENT)


    class UserDetail(APIView):
        """
        Retrieve, update or delete a user instance.
        """
        def get_object(self, pk):
            try:
                return User.objects.get(pk=pk)
            except User.DoesNotExist:
                raise Http404

        def get(self, request, pk, format=None):
            user = self.get_object(pk)
            user = UserSerializer(user)
            return Response(user.data)

        def put(self, request, pk, format=None):
            user = self.get_object(pk)
            serializer = UserSerializer(user, data=request.DATA)
            if serializer.is_valid():
                serializer.save()
                return Response(serializer.data)
            return Response(serializer.errors, status=status.HTTP_400_BAD_REQUEST)

        def delete(self, request, pk, format=None):
            user = self.get_object(pk)
            user.delete()
            return Response(status=status.HTTP_204_NO_CONTENT)

6. Update the urls.py

    from django.conf.urls import patterns, include, url

    from restapp import views

    from django.contrib import admin
    admin.autodiscover()

    urlpatterns = patterns('',
        # Examples:
        # url(r'^$', 'rest_example.views.home', name='home'),
        # url(r'^blog/', include('blog.urls')),

        url(r'^admin/', include(admin.site.urls)),
        url(r'^user/', views.UserList.as_view()),
        url(r'^users/(?P<pk>[0-9]+)/$', views.UserDetail.as_view()),
    )
And Viola we are done creating our API. 

Let is test our API now.
    python manage.py runserver

Now go to your browser and try localhost:8000/users/1/ or use curl on your command line

    curl http://127.0.0.1:8000/users/1/

You can get the user details what you have filled while creating the super user. which would look like this.

    {"id": 1, "username": "restapp", "first_name": "", "last_name": "", "email": "rakesh@agiliq.com"}

Now we can try posting data:

    curl -X POST http://127.0.0.1:8000/user/ -d '{"username":"rakhi", "email":"rakhi@agiliq.com"}' -H "Content-Type: application/json"

And it creates a new user with the username "rakhi" and email "rakhi@agiliq.com"

{"id": 6, "username": "rakhi", "first_name": "", "last_name": "", "email": "rakhi@gmail.com"}


# Tutorial from Traversy Media [Python Django 7 Hour Course](https://www.youtube.com/watch?v=PtQiiknWUcI)
[View Live Demo of StudyBud](https://studybuddev.herokuapp.com/)

## Table of Contents
- [Get Started](#get-started)
- [Database](#database)
- [Admin Panel](#admin-panel)
- [Model Form](#model-form)
- [User Authentication and Authorization](#user-authentication-and-authorization)
- [Templates](#templates)

## Get Started
### Create Virtual environment using pipenv and install django in it
```bash
pipenv install django
```

### Virtual environment created at: C:\Users\Shivam\.virtualenvs\DjangoTest-ikN7YSUi
Find path using 
```bash
pipenv --venv
```

### Access the virtual environment
```bash
pipenv shell
```

### Check if virtual environment is active and django is available 
```bash
django-admin
```
This should list all the commands 

### Start a new project 
```bash
django-admin startproject studybud
```
This should create a new folder and have initial setup files in it.
Inside the folder there should be another folder with the same name.

### Run the new project
```bash
cd studybud
python manage.py runserver
```
Open on given port on localhost and check if everything runs well

### manage.py
File for running project

### wsgi.py
Web service gateway interface - actual server

### urls.py 
inside main project folder (name of project); contains all the routing configs. 

### settings.py
Core config of the project - templates, databases, URLs, static file configs etc.

### installed apps (listed in settings.py)
Apps that handle different parts of the overall application

### Copy paste virtual environment folder into project root
TODO: To do after testing.

### Create a new app for this application
```bash
python manage.py startapp base
```
+ Generates a folder inside the project root folder with initial setup files for that app.
+ Contains views.py - contains business logic (functions/logic) when hitting a specific URL or endpoint.
+ Contains models.py - data models
+ Add base.apps.BaseConfig to InstalledApps list in setting.py 
+ Add logic to views.py, and point to it using urls.py 

### Create template folder in project root
+ Add html template files to it
+ Add it to settings.py inside TEMPLATES config by adding BASE_DIR / 'templates' to 'DIRS' - so it knows to use this folder. 

### Templates specific for app - create templates folder inside app folder and create subfolder with the app's name
+ in views, reference it like this 'base/home.html'

### Template inheritance
+ Include sections of templates inside other templates 
+ eg. using {% include 'navbar.html' %} for navbar at top of all pages.
+ Extend templates into other templates eg. see main.html extend to home.html using {% extends 'main.html' %} in home.html and wrapping child content as per tags in main.html

### Pass data to template 
use {{ variable_name }} as used by django's template engine

### Create for loops or if-else statements in templates 
+ use tags like {% if user.isAuth %}Hello, {{ user.username }}. {% endif %}
+ Always close python if-else statements with endif, and for loops with endfor in templates.

### Add dynamic urls 
+ add variables to url patterns in urls.py path parameter like this: 'room/<str:pk>/'
+ this will render a specific room's template where value of pk variable will be available to the view along with the request.
+ access it in views function like this: def room(request, pk)

### Use tag name in href after naming the path using the name parameter
+ path('room/<str:pk>/', views.room, name="room")
+ href="{% url 'room' room.id %}"

### Migrate default database tables for Auth, Admin, ContentTypes, Sessions from django
```bash
python manage.py migrate
```

[Back to Top](#table-of-contents)
---

## Database
+ Python classes represent DB tables (inherits from django models)
+ Attributes of class represent columns in tables
+ Instance of the class represents a row in the table
+ Records have id generated automatically starting from 1 and increments
+ see Room model in base app

### Make Migrations
To be done after making new models so django registers them in its config
```bash
python manage.py makemigrations
```
A new config fine will be created in migrations folder inside the app folder in which the new model was created

### Apply the migrations after creating them
To add tables of new models in the database
```bash
python manage.py migrate
```

### Add queries in views (logic)
queryset = ModelName.objects
+ .all() - to get all records
+ .get() - get only one specific record based on some attribute
+ .filter() - to filter based on some attribute
+ .exclude()

### Create models with Relations to other models using primary keys and foreign keys
+ primary key is the id of a record in its own table
+ using this id to reference the record in another table will require it to be passed as a foriegn key
+ One to Many - single user can have multiple posts. 
+ Many to One - single category like a topic can be assigned to multiple rooms. The many side of the relationship should have the foreign key.
```python
class Manufacturer(models.Model):
    name = models.TextField()

class Car(models.Model):
    manufacturer = models.ForeignKey(Manufacturer, on_delete=models.CASCADE)
```
+ Many to Many - students can have multiple courses and courses can have multiple students. 
+ Django's ORM automatically creates an intermediate table when you use the ManyToManyField
```python
class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField('Course')  # Django creates a junction table automatically

class Course(models.Model):
    title = models.CharField(max_length=100)
```
+ If you need extra fields, define an explicit model and pass it to the ManyToManyField as a parameter as value for 'through'
```python
class Enrollment(models.Model):  # Explicit junction table
    student = models.ForeignKey('Student', on_delete=models.CASCADE)
    course = models.ForeignKey('Course', on_delete=models.CASCADE)
    enrollment_date = models.DateField()

class Student(models.Model):
    name = models.CharField(max_length=100)
    courses = models.ManyToManyField('Course', through='Enrollment')  # Custom junction table
```
+ [See docs for reference](https://docs.djangoproject.com/en/5.1/ref/models/fields/#django.db.models.ManyToManyField.related_name)

### Filter records
Filter reconds using search bar or using keywords/tags

### Add a search bar using html form and input
+ add action to the form to call given url where search results are to be displayed (home).
+ the input name parameter will add query parameters to the url path of the action parameter of the form 
```html
<form method="GET" action="{% url 'home' %}">
    <input type="text" name="q" placeholder="Search Rooms ..." />
</form>
```

### Add filter method on records when they are fetched in the view function
+ import Q from django.db.models for writing queries
+ topic__name__contains will search topic.name field in Rooms records
+ icontains is for case insensitive
+ And(&) and Or(|) can be used with queries 
```python
rooms = Room.objects.filter(
    Q(topic__name__icontains=q) | 
    Q(name__icontains=q) | 
    Q(description__icontains=q)
)
```

### Query on child objects of a parent model
+ A parent model is related to a child model by one to many relation. Eg. A room can have multiple messages.
```python
class Room(models.Model):
    ...

class Message(models.Model):
    room = models.ForeignKey(Room)
```
+ Run query on the child model by using the lowercase name for the child model suffixed by _set to get the reference of the set of child model and then add the query to that set.
+ TODO: Add relevant model attributes for room and message 

```python
messages = room.messege_set.all()
```

### Keep objects in model sorted in a certain order or sort them while querying as an [Aggregation operation](https://docs.djangoproject.com/en/5.1/topics/db/aggregation/#:~:text=order_by%28%29).
```python
class Meta:
    ordering = ['-updated', '-created'] # reverse ordering - so latest if first.
```
```python
messages = Message.objects.all().order_by('-created')
```

### Render datetime as time since that date time
```html
{{message.created|timesince}}
```

---
## Admin panel
+ Access admin panel on the /admin path
+ Create admin user 
```bash
python manage.py createsuperuser
```
+ Allows admin to see database and execute crud operations using the UI.

### Register models form apps in admin panel
+ Go to admin.py of the specific app 
+ import the model 
+ register it with the admin site
```python
from .models import Room
admin.site.register(Room)
```

[Back to Top](#table-of-contents)
---

## Model Form
A class based representation of a form

### Create a new model in a new file inside the app folder
+ Seeforms.py 
+ import ModelForm from django.forms
+ import the existing model for which this form is to be created
+ create a class named as the model name with Form suffix
+ define class Meta in the named Form class with attributes model and fields
```python
class RoomForm(ModelForm):
    class Meta:
        model = Room
        fields = '__all__'
```
[Docs for Form API](https://docs.djangoproject.com/en/5.1/ref/forms/api/#django.forms.Form)

### Create html for the form
+ add csrf_token for preventing cross-site scripting
+ render form from variable form
+ as_p template renders each input and label within a p tag.
```html
<form method="POST" action="">
    {% csrf_token %}
    {{form.as_p}}
    <input type="Submit" value="Submit" />
</form>
```
+ Add a link on the home page or other page to allow user to navigate to this form
```html
<a href="{% url 'create-room' %}">Create Room</a>
```

### Create function in views for creating that models new records
+ function should return render of the form html on get requests (default)
+ it should have in its context the form model (imported in views from forms.py where its class was defined)
```python
def createRoom(request):
    form = RoomForm()
    context = {'form': form}
    return render(request, 'base/room_form.html', context)
```
+ In the html of form, on submitting action is left blank and method is POST. 
+ Therefore it will trigger the same url on Submit but with a post request
+ add this condition to get submitted form data in same function
+ check if it is valid using the is_valid() method for FormModels
+ call the save() method to save the record from the form in the DB.
+ after saving, use the redirect method from django.shortcuts to redirect the user.
```python
def createRoom(request):
    form = RoomForm()
    if request.method == 'POST':
        form = RoomForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('home')
        
    context = {'form': form}
    return render(request, 'base/room_form.html', context)
```

### Add a path in urls for the createRoom function in views with unique name
+ the name will be used in html templates for href attribute of links
```python 
path('create-room/', views.createRoom, name="create-room")
```

### Similarly, add function in view for updating and deleting records, and paths in url
+ pass primary-key of the record in the request so the function can identify that record.
+ for updating, render the form html but with instance of the specific record.
+ Similar to creating function, accept post requests from the form submit button to update the records in the DB.
```python
def updateRoom(request, pk):
    room = Room.objects.get(id=pk)
    form = RoomForm(instance=room)
    
    if request.method == 'POST':
        form = RoomForm(request.POST, instance=room)
        if form.is_valid():
            form.save()
            return redirect('home')
    
    context = {'form': form}
    return render(request, 'base/room_form.html', context)
```

### Add buttons/links on home page to let users access update or delete records functions
+ pass room id as parameter in the url
+ use the name parameter passed in the url path
```html
<a href="{% url 'update-room' room.id %}">Edit</a>
<a href="{% url 'delete-room' room.id %}">Delete</a>
```

[Back to Top](#table-of-contents)
---

## User Authentication and Authorization
Django comes with inbuilt features for User accounts, groups, permissions and cookie-based user sessions. Find [the official docs here](https://docs.djangoproject.com/en/5.1/topics/auth/)

### User objects
User objects are the core of the authentication system. They typically represent the people interacting with your site and are used to enable things like restricting access, registering user profiles, associating content with creators etc.
The primary attributes of the default user are:
+ username
+ password
+ email
+ first_name
+ last_name
See the full [API docs](https://docs.djangoproject.com/en/5.1/ref/contrib/auth/#django.contrib.auth.models.User)

### Create functions in views for rendering register page, and performing register function
+ For registration, use the UserCreation form imported from django.contrib.auth.forms
```python
from django.contrib.auth.forms import UserCreationForm
```
+ render this form in html simply by passing it in the context and rendering as a form template inside a form
```html
<form method="POST" action="">
    {% csrf_token %}

    {{form.as_p}}

    <input type="submit" value="Register" />
</form>
```
+ The function for registration will be called on the same path as a POST request
```python
def registerPage(request):
    form = UserCreationForm()
    
    if request.method == 'POST':
        form = UserCreationForm(request.POST)
        if form.is_valid():
            user = form.save(commit=False)
            user.username = user.username.lower()
            user.save()
            login(request, user)
            return redirect('home')
        else:
            messages.error(request, 'An error occoured during registration.')
        
    return render(request, 'base/login_register.html', {'form': form})
```
+ Normally, calling form.save() saves the form's data directly to the database.
+ commit=False: This argument prevents Django from immediately saving the instance to the database.
+ This allows us to modify fields before saving and add additional logic or fields that aren't directly provided by the form.
+ Finally add the path for register view in the urls.py file
```python
path('register/', views.registerPage, name='register'),
```

### Create functions in views for rendering login page and performing login
+ The login function first checks if the user is not already authenticated using the is_authenticated attribute of the user object
+ Similar to the register function, it will render the login html if the request type is not POST method
```html
<div>
    <form method="POST" action="">
        {% csrf_token %}

        <label>Username:</label>
        <input type="text" name="username" placeholder="Enter Username" />
        
        <label>Password:</label>
        <input type="password" name="password" placeholder="Enter password" />


        <input type="submit" value="Login" />
    </form>

    <p>Haven't signed up yet?</p>
    <a href="{% url 'register' %}">Sign up</a>
</div>
```
+ If the request method is POST, it should run the login function
+ Get the username and passoword from the request
```python
username = request.POST.get('username')
password = request.POST.get('password')
```
+ Inside a try catch block check if the user exists or not
```python
try:
    user = User.objects.get(username=username)
except:
    messages.error(request, 'User does not exist')
```
+ import the authenticate, login, and logout methods
```python
from django.contrib.auth import authenticate, login, logout
```
+ If the user exists, verify them using the authenticate method.
```python
user = authenticate(request, username=username, password=password)
if user is not None:
    login(request, user)
    return redirect('home')
else:
    messages.error(request, 'Invalid Username or Password')
```

### Protected Routes/Views
+ To protect views i.e. prevent user access before logging in
```python
from django.contrib.auth.decorators import login_required
```
+ Add above views that need to be protected. Pass parameter login_url so they will be redirected to login.
```python
@login_required(login_url='login')
```

### Logout method
Add view, path in url and html link with href to logout url
```html
<a href="{% url 'logout' %}">Logout</a>
``` 
```python
def logoutUser(request):
    logout(request)
    return redirect('home')
```

### Authorization
+ Preventing users for unauthorized access can be done by rendering the html conditionally by checking if user has permission to perform some function.
+ And also by adding conditions in view functions that check if user is authenticated and/or authorized to perform some function.
+ If user does not have permissions, send a http response like in this example
```python
if request.user != room.host:
    return HttpResponse('You are not allowed to update this one!')
```

[Back to Top](#table-of-contents)
---

## Templates
A template contains the static parts of the desired HTML output as well as some special syntax describing how dynamic content will be inserted.
[See Docs for reference](https://docs.djangoproject.com/en/5.1/topics/templates/)

### Using templates with variables to get dynamic data.
+ When re-using templates in different html pages, ensure variable names in the html pages are matching with what is required in template.
+ For example, activity feed template is used in home and user profile page. 
+ In home it shows all activity, but in user profile it only show's that users activity. 
+ This works because all_messages passed to home has all users' messages, but in user profile it is filtered to contain only that user's messages.

## Static Files
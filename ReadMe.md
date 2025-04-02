# Tutorial from Traversy Media [Python Django 7 Hour Course](https://www.youtube.com/watch?v=PtQiiknWUcI)

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

### Add queries in views (logic)
queryset = ModelName.objects
+ .all() - to get all records
+ .get() - get only one specific record based on some attribute
+ .filter() - to filter based on some attribute
+ .exclude()

### Create models with Relations to other models using primary keys and foreign keys
+ primary key is the id of a record in its own table
+ using this id to reference the record in another table will require it to be passed as a foriegn key
+ One to Many - single user can have multiple posts
+ Many to One - single category like a topic can be assigned to multiple rooms.

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

## Filter records
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
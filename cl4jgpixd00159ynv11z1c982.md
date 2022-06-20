## Hey Django & Graphql

I have been flirting with Graphql for some time now and yes yes I have heard it all from how it reduces over and under fetching all the way to it is the best tool for building APIs. For the Rest fans out there this is not an article to convince you to use GraphQL, trust me and check it out with an open mind. GraphQL is an open-source data query and manipulation language for APIs, and a runtime for fulfilling queries with existing data. GraphQL was developed internally by Facebook in 2012 before being publicly released in 2015. Hey Mark! good one, this technology is amazing!!! . Enough with the theories let us integrate graphql with django and create magic without a magic wand Harry Porter will be amazed.

## Getting Started

Let's create a simple recipe project using django, that has a simple recipe model and add some data to query later on using Garaphql. Are you excited? let's start...... 


```
django-admin startproject core .
``` 

Add a virtual environment and .gitignore file to keep everything neat and tidy. To install a virtual environment.

```
python -m venv envrecipe
``` 

The second argument is the location to create the virtual environment. Generally, you can just create this in your project and call it env.

venv will create a virtual Python installation in the env folder. And after successfully creating the virtual environment you might want to activate it using the command below...

```
source envrecipe/bin/activate
``` 

Note: Installing and activating a virtual environment is a little different from other operating systems you might want to
[Check them out here](https://docs.python.org/3/library/venv.html)

## Creating a new app called Recipe

In this app, a schema will be created to hold all the necessary queries. Navigate to the root folder of the project and execute the command....

```
python manage.py startapp recipe
``` 

Tell django about the new app created in core/settings.py

```
INSTALLED_APPS = [
    # default apps
    'recipe',
]
``` 
All set with the django basic installation next step is to migrate and actually start the server with...

```
python manage.py makemigrations
``` 

```
python manage.py migrate
``` 

No errors ? Start the server.....

```
python manage.py runserver
``` 

![Django1.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1655225573998/qp99Jan3X.png align="left")


## Creating a simple model and adding data

In model.py create a new model and call it Recipe and add some fields.

```
from django.db import models

# Create your models here.

class Recipe(models.Model):
    dish = models.CharField(max_length=50)
    description = models.TextField()
    created_date = models.DateTimeField(auto_now_add=True)
    updated_date = models.DateField(auto_now=True)
``` 

Now we are getting to the good stuff. Inform django that a new model was created. And that can be done by running a couple of migrations.

Makemigrations command

```
python manage.py makemigrations
``` 

Migrate to create the database model using

```
python manage.py migrate
``` 

After adding a new model proceed and add a couple of recipe data that will be used in the query later in graphql. Django has two ways of adding data either by using the admin interface or with the django shell. I will be using the latter. Head over to your terminal and start the Django shell by typing the following:

```
python manage.py shell
``` 

In order to work with our django models in the shell first import the Recipe model from the recipe app

```
>>> from recipe.models import Recipe

>>> Recipe.objects.create(dish="Benachin", description="Benachin is a popular Gambian dish that is prepared in a pot. It consists of rice, vegetables, and meat cooked together.")

>>> exit()
``` 

You can add as many recipes as you wish and don't forget to exit the shell after you are done. The next step is to add graphql to the project, head over to the terminal and install some packages using....

```
pip install graphql-core
``` 

```
pip install graphene-django
``` 

These two packages are all you need to integrate graphql to your django project. If you run into any errors, try installing the [Graphql package ](https://pypi.org/project/graphql/).


## Working with Graphql and schemas

A GraphQL schema is a description of the data clients can request from a GraphQL API. It also defines the query function that the client can use to read and write data from the GraphQL server. In other words, you specify your client or application UI data requirements in your GraphQL schema. And as always django came through with batteries installed that aid in having your models and schemas in the same app. That will require adding graphene-django to the installed apps.


```
INSTALLED_APPS = [
    # default apps
    'recipe',
    'graphene_django',
]

# This tells django to look for a graphql schema in core / schema.py
GRAPHENE = {
    'SCHEMA' : 'core.schema.schema',
}
``` 

That seems about it let's write the first query now by creating a new file named schema.py in our recipe app. A schema describes the shape of your available data and the django-graphene package defines a hierarchy of types with fields that are populated from our model. The schema also specifies exactly which queries are available for clients to execute.

### Object types, Fields and a first Query

The most basic components of a schema are the object types, which just represent a kind of object you can fetch from your service, and what fields it has. The Graphiql UI helps mostly in documenting this with zero effort another win for Graphql. A GraphQL query is used to read or fetch values, the operation is a simple string that a GraphQL server can parse and respond to with data in a specific format. The popular response format that is usually used for mobile and web applications is JSON. We all love Django because it does a lot of the heavy lifting and django-graphene package is no different. This pacakge helps map your model and actually takes care of fields that are created automatically like ID's, datetime.now fields etc. Enough with the technicalities let's create the first query.

recipe/schema.py
```
import graphene
from graphene import ObjectType
from graphene_django import DjangoObjectType
from .models import Recipe

#Converting our recipe model to a graphql type
class RecipeType(DjangoObjectType):
    class Meta:
        model = Recipe
        # All fields in our model 
        fields = ('__all__')

class Query(ObjectType):
    #name of query that gets a list of all the recipe data and stores it as a list
    all_recipes = graphene.List(RecipeType)

     #A resolver populates the data in your schema.
    def resolve_all_recipes(self, info):
        return Recipe.objects.all()
``` 

After making a schema for the recipe model head over and create another graphql schema in the core project that maps the schema created in the recipe app. This is optional and the reason I use this approach is to have one schema file to respond to and all the other app schemas can be referenced back to just one schema file. I find it super convenient but feel free to implement your own logic.

core/schema.py

```
import graphene
from recipe import schema

#Maps the recipe model schema to the core schema that is referenced in settings.py
class Query(schema.Query, graphene.ObjectType):
    pass

schema = graphene.Schema(query=Query)
``` 

Finally, create a URL in core to access the graphql user interface. 

core/urls.py
```
from django.contrib import admin
from django.urls import path
from graphene_django.views import GraphQLView
from django.views.decorators.csrf import csrf_exempt

urlpatterns = [
    path('admin/', admin.site.urls),
    path('',csrf_exempt(GraphQLView.as_view(graphiql = True))),
]
``` 

Run the server and navigate to local host using the command

```
python manage.py runserver
``` 

### No errors ? Magic ? ......


![graphql-3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1655524670891/9bJ2MyE06.png align="left")

Looking closely graphql has inbuilt documentation, accurately documenting any defined query and the field types also specifying all required fields. Amazing just amaaaazing!

![query.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1655524786202/EY3AKgFMO.png align="left")

## Writing Queries in GraphQL

At its simplest, GraphQL is about asking for specific fields on objects. For this tutorial, a simple query was created to list all the recipes in our database, and to execute this particular query the syntax is as follows:

```
{
  allRecipes{
    id
    dish
    description
    createdDate
  }
}
``` 

And the response will look like this...

![query2.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1655526799874/tjNy0iEse.png align="left")

Let's say a client in the frontend wants to fetch the list of recipe names and descriptions only, well graphql fetches only those specific data with just a small tweak in the query...

```
{
  allRecipes{
    dish
    description
  }
}
```

![query3.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1655527055467/ZDrFST5ZV.png align="left")


## Conclusion

GraphQL is an amazing technology for building evolvable and dynamic APIs that can adapt to the needs of diverse clients and use cases. Of course, there are a few challenges as there always will be with relatively new technologies. Therefore, it’s important to understand that GraphQL is another tool in your toolkit and not something that is here to replace REST or make it obsolete. 

[Find the full project here](https://github.com/jattanjie21/HeyGraphql)





### Possible error solutions

[import error 'force_text' from 'django.utils.encoding'](https://stackoverflow.com/questions/70382084/import-error-force-text-from-django-utils-encoding)

[Graphql error handling](https://www.howtographql.com/graphql-python/6-error-handling/)

### Reference

https://dgraph.io/blog/post/designing-graphql-schemas/#:~:text=A%20GraphQL%20schema%20is%20a,requirements%20in%20your%20GraphQL%20schema.

https://graphql.org/learn/schema/

https://docs.graphene-python.org/projects/django/en/latest/tutorial-plain/
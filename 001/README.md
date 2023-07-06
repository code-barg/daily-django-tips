

![enter image description here](https://github.com/code-barg/daily-django-tips/blob/data/img/01-model-inheritance.jpeg?raw=true)

<h1 align="center">💡 Django Model Inheritance - Part 1</h1>

Hello :raised_hand_with_fingers_splayed:
How are you? Today, we are going to talk about inheritance in Django models and learn some key points about it. So, let's get started:

#### First of all, what is a model ❓

> A model is the single, definitive source of information about your data. It contains the essential fields and behaviors of the data you’re storing. Generally, each model maps to a single database table.

And now, Django's ORM has provided us with some interesting capabilities in terms of model inheritance, which we want to learn about today.

> ORM stands for **Object Relational Mapping**. Django allows us to add, delete, modify and query objects, using an API called ORM. Django provides a default admin interface that is required to perform operations like create, read, update and delete on the model directly.

##  Types of inheritance in Django models

> When we talk about inheritance in Django models, it essentially means that a class is going to inherit from one or more other classes that have all inherited from `django.db.models.Model`!
> Like a dog eat dog! The hardest thing in the world, explaining the concept with code.

### Abstract inheritance


Consider the following example:

```python
class TimeStampedModel(models.Model):
    created_at = models.DateTimeField(auto_add_now=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        abstract = True


class Post(TimeStampedModel):
    title = models.CharField(max_length=128)
    ...


class Product(TimeStampedModel):
    name = models.CharField(max_length=128)
    ...
```


If we have multiple classes like Product, we can utilize our custom abstract class, TimeStampedModel, without any difficulty.

So, what is the outcome of this operation? Before anything else, pay attention to the point in this code:
‍‍‍
```python
class ...:
    ...
	
    class Meta:
        abstract = True
...
```

To make Mixin models, which are actually abstract, we must follow this rules.

Well, let's define it now: abstract models are models that

  1. They don't exist in the database by themselves (we don't have a TimeStampedModel table)
  2. They are useful when you want to include some common information in a number of other models.
  3. When used as a base class for other models, its fields are added to the child class.

  - - -
Let's go for a practical and interesting example:

```python
from django.db import models  
  
  
class TimeStampedModel(models.Model):  
	"""  
	TimeStampedModel  
	  
	An abstract base class model that provides self-managed "created" and  
	"updated" fields.  
	"""  
	created = models.DateTimeField(auto_now_add=True)  
	updated = models.DateTimeField(auto_now=True)  
	  
	class Meta:  
		abstract = True  
		get_latest_by = 'created' # default to get latest by created  
		ordering = ['-created'] # default to order by created descending  
  
  
class LogicalDeleteModel(models.Model):  
	"""  
	LogicalDeleteModel  
	  
	An abstract base class model that provides an "is_deleted" field.  
	"""  
	is_deleted = models.BooleanField(default=False)  
	  
	class Meta:  
		abstract = True  
  
  
class LogicalActiveModel(models.Model):  
	"""  
	LogicalActiveModel  
	  
	An abstract base class model that provides an "is_active" field.  
	"""  
	is_active = models.BooleanField(default=True)  
	  
	class Meta:  
		abstract = True  
  
  
class Product(LogicalActiveModel, LogicalDeleteModel, TimeStampedModel):  
	"""  
	Product model  
	  
	A product that can be purchased.  
	"""  
	name = models.CharField(max_length=128)  
	description = models.TextField()  
	price = models.DecimalField(max_digits=10, decimal_places=2)  
	quantity = models.IntegerField()  
	  
	def __str__(self):  
		return self.name
```

In this example, we have implemented 4 models
All the first three models are actually BaseModel or abstract models, and the other models that we only used the Product model here are models that benefit from the capabilities of abstract models.

Pay attention to the `migrations` file of these models:

```python
from django.db import migrations, models  
  
class Migration(migrations.Migration):  
	initial = True  
	operations = [  
		migrations.CreateModel(  
			name='Product',  
			fields=[  
				('id', models.BigAutoField(auto_created=True, primary_key=True, serialize=False, verbose_name='ID')),  
				('created', models.DateTimeField(auto_now_add=True)),  
				('updated', models.DateTimeField(auto_now=True)),  
				('is_deleted', models.BooleanField(default=False)),  
				('is_active', models.BooleanField(default=True)),  
				('name', models.CharField(max_length=128)),  
				('description', models.TextField()),  
				('price', models.DecimalField(decimal_places=2, max_digits=10)),  
				('quantity', models.IntegerField()),  
			],  
			options={  
				'abstract': False,  
			},  
		),  
	]

```

you got it ? None of our abstract models exist in our database! But all the features of each of those models are present in the Product model, and in fact, we have used clean coding rules with high accessibility and order.

The remarkable thing is that all the methods and features of their classes are specific to themselves.

Do not be tired 🤠

- - -

### 🖇️ PS:
The level of this document is general and easy, so in the next parts we will try to improve the level bit by bit.

Of the two important abstract base classes shown here, for example, `LogicalActive` and `LogicalDelete` are two widely used and efficient abstract models that we will have a detailed session on shortly.

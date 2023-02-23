# Integration Testing Django Apps with Pytest

As software development progresses rapidly and consumers demand more functionalities within shorter development timeframes, developers may feel tempted to rush through testing and software release. However, rushing through testing can have serious consequences as it may result in unstable or unreliable software that is prone to security vulnerabilities. Additionally, rushing through testing can create technical debt, which can negatively impact the overall quality of the software and slow down development in the long run.

Integration testing is an important part of any software development process. It ensures that different parts of the software work together as expected and that the software as a whole behaves correctly under a variety of scenarios. In this blog post, Pytest is used to perform integration testing of Django apps and more specifically web application software. I know the saying "developers should write code and testers should do the testing" however it wouldn't hurt to add this skill to your amazing collection.

## **What is Integration Testing?**

Integration testing is a type of testing that checks whether different components of an application work together correctly. In web development, this often involves testing how the application's front end (HTML, CSS, and JavaScript) interacts with the back end (server-side code, databases, and APIs).

Integration testing is different from unit testing, which tests individual components in isolation. In contrast, integration testing tests how the components interact with each other. As a result, integration testing is often more complex and time-consuming than unit testing. However, a few drawbacks might include databases, environments, platforms, and third-party APIs but trust me it will save you a lot of debugging time.

### **Pytest for Integration Testing**

Pytest is a popular testing framework for Python applications. It has a simple and intuitive syntax, making it easy to write and read tests. Pytest also offers a wide range of features for testing, such as fixtures, parameterization, and test discovery.

To perform integration testing of Django apps, you can use pytest in conjunction with the Django testing framework. The Django testing framework provides a set of tools and utilities for testing Django apps, such as the ability to create test databases and test clients.

### **Setting up a Django Test Environment**

Before performing integration testing of a Django app with Pytest, you need to set up a test environment. To do this, create a separate settings file for testing, which specifies a test database and other test-related settings.

To create a test settings file, create a new file called `settings_test.py` in the same directory as the main `settings.py` file. In this file, specify the test database using the following code:

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / 'db_test.sqlite3',
    }
}
```

This code specifies an SQLite database for testing, with the database file located in the same directory as our project's base directory.

Also, specify the `TEST_RUNNER` setting in our `settings_test.py` file, which tells Django to use Pytest as the test runner:

```python
TEST_RUNNER = 'pytest_runner.run_tests'
```

Finally, add a `pytest.ini` file to the project's root directory, which tells Pytest how to discover and run tests:

```python
[pytest]
DJANGO_SETTINGS_MODULE = projectname.settings_test
python_files = tests.py test_*.py *_tests.py
```

This code specifies that Pytest should use the `settings_test.py` file as the Django settings module and that test files should be named with the prefix `test_`. This is helpful when isolating single files later on while testing its way cleaner and more flexible.

### **Writing Integration Tests with Pytest**

Now that the test environment is set up, you can start writing integration tests with pytest. In general, integration tests for Django apps involve simulating HTTP requests and responses using Django's test client, and then asserting that the response is correct.

Here's an example integration test for a simple Django app that displays a list of tasks:

```python
from django.urls import reverse
from django.test import Client

def test_task_list(client):
    url      = reverse('task-list')
    response = client.get(url)

    assert response.status_code == 200
    assert len(response.context['task']) == 10
```

In this test, the Django test client simulates a GET request to the `task-list` URL, which should display a list of tasks. We then assert that the response status code is 200 (indicating a successful request) and that the response context contains a list of 10 tasks.

Pytest also provides several features that make it easy to write and run integration tests. For example, you can use pytest fixtures to set up test data and resources. Here's an example of using a fixture to set up a test database:

```python
import pytest
from django.urls import reverse
from tasks.models import Task
from django.test import Client

@pytest.fixture
def client():
    return Client()

@pytest.fixture
def db_setup():
    # Set up test data and resources
    pass

def test_create_task(client):
    data     = {'title': 'Test Task'}
    response = client.post(reverse('task-create'), data)
    assert response.status_code == 302
    assert Task.objects.filter(title='Test Task').exists()

def test_complete_task(client):
    task = Task.objects.create(title='Test Task')
    data = {'completed': True}
    response = client.post(reverse('task-update', args=[task.id]), data)
    assert response.status_code == 302
    task.refresh_from_db()
    assert task.completed == True

def test_delete_task(client):
    task = Task.objects.create(title='Test Task')
    response = client.post(reverse('task-delete', args=[task.id]))
    assert response.status_code == 302
    assert not Task.objects.filter(title='Test Task').exists()
```

In this example, an integration test was created using pytest to write tests for creating, completing, and deleting tasks.

### Conclusion

Writing integration tests is an important part of the software development process, as it helps to ensure that our application is functioning correctly and meets the requirements. By using pytest, you can easily write and run tests, and it provides a lot of helpful features for testing Django applications.

I hope you found this tutorial helpful and that it encourages you to start writing integration tests for your Django projects. Remember, testing is an ongoing process, and we should strive to write tests that cover all aspects of our application. Happy testing!

### Useful Resources

[Testing in Django](https://docs.djangoproject.com/en/4.1/topics/testing/)

[pytest-django Documentation](https://pytest-django.readthedocs.io/en/latest/index.html)

[Pytest runner package](https://pypi.org/project/pytest-runner/)

[The importance of testing in web development: purpose and methods](https://bitbag.io/blog/the-importance-of-testing-in-web-development)

[Django testing using pytest](https://dev.to/xarala221/django-testing-using-pytest-1pak)

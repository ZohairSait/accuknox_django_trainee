# AccuKnox Django Trainee Assessment Project

This repository contains a Django project developed to explore and demonstrate the behavior of Django signals, along with a custom Python `Rectangle` class implementation.

The objective of this project is to provide clear, practical answers to the AccuKnox trainee assessment questions through hands-on experimentation and testing.

---

# Project Overview

This project consists of two main components:

## 1. Django Signal Demonstration (`demo_app`)

A minimal Django application created to examine how signals behave by default:

- Whether signals execute synchronously or asynchronously  
- Whether signals run in the same thread as the caller  
- Whether signals share the same database transaction as the triggering operation  

## 2. Custom Python `Rectangle` Class

A standalone Python file (`rectangle.py`) implementing a `Rectangle` class that supports iteration according to specified requirements.

---

# Setup Instructions

Follow these steps to run the project locally:

## 1. Install Django

```bash
pip install django
```

## 2. Apply Database Migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

## 3. Start the Development Server

```bash
python manage.py runserver
```

## 4. Test Signal Behavior

Visit:

```
http://127.0.0.1:8000/create/
```

## 5. Test the Rectangle Class

```bash
python rectangle.py
```

---

# Answers to AccuKnox Trainee Questions

---

# Topic: Django Signals

---

## Question 1: Are Django signals synchronous or asynchronous by default?

**Answer:**  
Django signals execute **synchronously by default**. When a signal is triggered, its handler executes immediately and blocks further execution until it completes.

---

### Demonstration

A `post_save` signal is connected to `MyModel`. The signal handler includes a 2-second delay to clearly demonstrate blocking behavior.

### View (`demo_app/views.py`)

```python
from django.http import HttpResponse
from .models import MyModel
import threading

def create_model(request):
    print(f"View running in thread: {threading.current_thread().name}")
    instance = MyModel.objects.create(name="Test")
    print("Model created")
    return HttpResponse("Model created")
```

### Signal (`demo_app/signals.py`)

```python
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import MyModel
import threading
import time

@receiver(post_save, sender=MyModel)
def my_signal_handler(sender, instance, **kwargs):
    print(f"Signal received in thread: {threading.current_thread().name}")
    time.sleep(2)
    print("Signal handler finished")
```

### Result

When accessing `/create/`, the browser response is delayed by 2 seconds.  
The terminal output confirms that the signal handler executes before the HTTP response is returned.

This demonstrates **synchronous execution**.

---

## Question 2: Do Django signals run in the same thread as the caller?

**Answer:**  
Yes. By default, Django signals run in the **same thread** as the code that triggers them.

---

### Observed Output

```
View running in thread: MainThread
Model created
Signal received in thread: MainThread
Signal handler finished
```

The identical thread name confirms that the signal handler runs in the same execution thread as the calling view.

---

## Question 3: Do Django signals run in the same database transaction as the caller?

**Answer:**  
Yes. By default, Django signals execute within the **same database transaction** as the triggering operation.

---

### Demonstration

The signal handler is modified to intentionally raise an exception:

```python
@receiver(post_save, sender=MyModel)
def my_signal_handler(sender, instance, **kwargs):
    raise Exception("Signal exception")
```

When `/create/` is accessed:

- The request fails with an exception
- The database record is not saved

---

### Verification in Django Shell

```bash
python manage.py shell
```

```python
from demo_app.models import MyModel
MyModel.objects.all()
```

Output:

```
<QuerySet []>
```

This confirms that the transaction is rolled back when the signal handler raises an exception.

---

# Topic: Custom Python Class

---

## Task: Implement a `Rectangle` Class

### Requirements

- Initialize with `length: int` and `width: int`
- Support iteration
- Iteration should yield:
  - `{'length': <value>}` first
  - `{'width': <value>}` second

---

## Implementation (`rectangle.py`)

```python
class Rectangle:
    def __init__(self, length: int, width: int):
        self.length = length
        self.width = width

    def __iter__(self):
        yield {'length': self.length}
        yield {'width': self.width}

if __name__ == "__main__":
    rect = Rectangle(5, 10)
    for item in rect:
        print(item)
```

---

## Output

```
{'length': 5}
{'width': 10}
```

The `__iter__` method uses `yield` to return values sequentially, enabling the object to be iterable while preserving the required output order.

---

# Conclusion

This project demonstrates:

- Clear understanding of Django signal behavior:
  - **Synchronous execution**
  - **Same-thread execution**
  - **Shared database transaction context**
- Practical experimentation to validate theoretical concepts
- Implementation of a custom iterable Python class using generators

The repository reflects structured experimentation, clean implementation, and accurate validation of expected behaviors as required for the AccuKnox trainee assessment.

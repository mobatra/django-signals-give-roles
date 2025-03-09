# **Django Signals: Deep Understanding & Role Assignment**

Django **signals** provide a way for different parts of an application to **communicate** without tightly coupling them. This document explains **what Django signals are, how they work, and how to use them to assign a default role to new users**.

---

## **🔍 1. What Are Django Signals?**
Django **signals** allow certain **senders** (like models) to **notify** other parts of the application when specific **events** occur. This enables **automated responses** to changes in the database without modifying existing business logic.

**Example Use Cases:**
- Assign **default roles** to users upon registration.
- Create a **UserProfile** automatically when a user registers.
- Send **welcome emails** to new users.

---

## **📌 2. How Django Signals Work**
### **🚀 Three Main Components**
| **Step** | **Description** |
|---------|---------------|
| **Signal is sent** | Django triggers an event (e.g., user created, model updated). |
| **Signal is received** | A **receiver function** listens for the event. |
| **Action is executed** | The receiver function runs (e.g., assigning a group, sending an email). |

### **📌 Types of Built-in Django Signals**
| **Signal** | **Triggers When** | **Use Case** |
|-----------|-----------------|-------------|
| `pre_save` | Before a model instance is saved | Modify data before saving |
| `post_save` | After a model instance is saved | Assign roles, send emails |
| `pre_delete` | Before a model instance is deleted | Prevent accidental deletion |
| `post_delete` | After a model instance is deleted | Cleanup related data |

---

## **📌 3. Using Django Signals to Assign Default User Roles**

### **🔹 Step 1: Modify the `User` Model**
```python
from django.contrib.auth.models import AbstractUser, Group
from django.db import models
from django.db.models.signals import post_save
from django.dispatch import receiver

class User(AbstractUser):
    email = models.EmailField(unique=True)

    def __str__(self):
        return self.username

# ✅ Automatically assign new users to the "User" group
@receiver(post_save, sender=User)
def assign_default_group(sender, instance, created, **kwargs):
    if created:  # Only run for new users
        user_group, _ = Group.objects.get_or_create(name="User")  # Default group
        instance.groups.add(user_group)  # Assign user to group
```

### **🔹 Step 2: Connect the Signal in `apps.py`**
Modify the `users/apps.py` file:
```python
from django.apps import AppConfig

class UsersConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'users'

    def ready(self):
        import users.signals  # ✅ Ensure signals are connected at startup
```

---

## **📌 4. Understanding the `post_save` Signal**
```python
@receiver(post_save, sender=User)
def assign_default_group(sender, instance, created, **kwargs):
```
- `@receiver(post_save, sender=User)`: Triggers **after** a `User` is saved.
- `instance`: The actual `User` object that was just created.
- `created`: `True` if the user **is new**, `False` if they were updated.

```python
if created:
    user_group, _ = Group.objects.get_or_create(name="User")
    instance.groups.add(user_group)
```
- ✅ **Ensures only new users get the "User" group.**
- ✅ **`get_or_create()`** → Finds or creates the group.
- ✅ **`groups.add(user_group)`** → Assigns the group to the new user.

---

## **📌 5. Testing the Signal**
### **1️⃣ Create a User & Verify Group Assignment**
#### **Test in Django Shell:**
```bash
python manage.py shell
```
```python
from django.contrib.auth.models import User
from django.contrib.auth.models import Group

# Create a new user
user = User.objects.create(username="batrawi", email="batrawi@gmail.com", password="securepass")

# Check assigned groups
print(user.groups.all())
```
✅ **Expected Output:**
```
<QuerySet [<Group: User>]>
```
🎉 The user is automatically part of the "User" group!

---

## **📌 6. Why Use Django Signals?**
✅ **Decouples logic** → Keeps code modular and reusable.  
✅ **Automates workflows** → No need to manually assign roles.  
✅ **Enhances security** → Ensures default roles are always assigned.  
✅ **Reduces code duplication** → Logic is centralized and reusable.  

---

## **📌 7. Summary Table**
| **Concept** | **Explanation** |
|------------|----------------|
| **Django Signals** | Allow different parts of the app to communicate without tight coupling. |
| **`post_save`** | Runs **after** an object is saved. |
| **Receiver Function** | The function that runs when the signal is triggered. |
| **`sender=User`** | Ensures the signal only listens for changes in the `User` model. |
| **`instance`** | Refers to the actual object being saved (the new user). |
| **`created=True`** | Ensures the action runs **only when a new object is created**. |



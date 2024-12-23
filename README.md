# recipe-app-api
Tests for models.
"""
from django.test import TestCase
from django.contrib.auth import get_user_model


class ModelTests(TestCase):
    """Test models."""

    def test_create_user_with_email_successful(self):
        """Test creating a user with an email is successful."""
        email = 'test@example.com'
        password = 'testpass123'
        user = get_user_model().objects.create_user(
            email=email,
            password=password,
        )

        self.assertEqual(user.email, email)
        self.assertTrue(user.check_password(password))
ChatGPT said:
ChatGPT
The provided code tests the create_user functionality for a custom user model in Django. Below is an explanation of the code flow and its purpose.

Code Breakdown
Imports:

TestCase: A Django-specific class for writing unit tests.
get_user_model: A Django utility function to dynamically retrieve the user model (useful when a custom user model is defined).
ModelTests Class:

Inherits from TestCase, allowing the use of Django's testing framework.
test_create_user_with_email_successful Method:

Purpose: Tests whether a user can be successfully created with an email and password.
Test Steps:
Setup: Define test data for the email and password.
Action: Call create_user on the custom user model to create a user object.
Assertions:
Ensure the email provided matches the email of the created user.
Verify the password by using the check_password method, which hashes and compares the input password with the stored hash.
How It Works
User Creation:

get_user_model() retrieves the current user model (User by default, or a custom user model if specified in settings.AUTH_USER_MODEL).
objects.create_user is the method used to create a user in the database. It's typically defined in the custom user model manager.
Assertions:

self.assertEqual(user.email, email):
Confirms that the user's email matches the expected email.
self.assertTrue(user.check_password(password)):
Verifies that the provided password was properly hashed and stored, and that it matches when checked.
Purpose of the Test
This test ensures that:

A user can be created successfully with an email and password.
The email is stored correctly in the database.
Passwords are hashed securely and verified correctly.


"""
Database models.
"""
from django.db import models
from django.contrib.auth.models import (
    AbstractBaseUser,
    BaseUserManager,
    PermissionsMixin,
)


class UserManager(BaseUserManager):
    """Manager for users."""

    def create_user(self, email, password=None, **extra_fields):
        """Create, save and return a new user."""
        user = self.model(email=email, **extra_fields)
        user.set_password(password)
        user.save(using=self._db)

        return user


class User(AbstractBaseUser, PermissionsMixin):
    """User in the system."""
    email = models.EmailField(max_length=255, unique=True)
    name = models.CharField(max_length=255)
    is_active = models.BooleanField(default=True)
    is_staff = models.BooleanField(default=False)

    objects = UserManager()

    USERNAME_FIELD = 'email'
ChatGPT said:
ChatGPT
The provided code defines a custom user model in Django with a corresponding manager for creating and managing user instances. Here's a detailed breakdown of the code and its components:

Code Breakdown
1. UserManager Class
This class inherits from BaseUserManager and provides methods for creating user instances.

create_user Method:
Used to create and save a regular user.
Parameters:
email: The user's email address.
password: The user's password.
**extra_fields: Additional fields that can be passed while creating the user.
Steps:
Instantiates a new user object using self.model.
Calls set_password to hash the provided password securely.
Saves the user object in the database.
Returns:
The created user object.
2. User Class
This is the custom user model that inherits from:

AbstractBaseUser:
Provides core authentication functionality, such as password handling.
PermissionsMixin:
Adds fields and methods for user permissions, such as groups and superuser status.
Fields:
email: A unique email field (used as the primary identifier instead of a username).
name: A string field for the user's name.
is_active: A Boolean field indicating whether the user is active (used for soft deletion).
is_staff: A Boolean field indicating whether the user is a staff member (used to grant access to the admin site).
Meta Information:
objects: Links the custom UserManager to the model, allowing you to use its methods (e.g., create_user).
USERNAME_FIELD: Defines which field is used as the unique identifier for authentication (in this case, email).
How This Works in Django
Custom User Model:

Replaces Django's default User model.
Configured in settings.py with:
python
Copy code
AUTH_USER_MODEL = 'app_name.User'
Creating a User:

Example:
python
Copy code
from app_name.models import User

user = User.objects.create_user(
    email='test@example.com',
    password='testpass123',
    name='Test User'
)
Custom Authentication:

By using AbstractBaseUser and PermissionsMixin, Django integrates this custom model into its authentication system.
Key Features
Email as Username:

Replaces the default username field with email as the primary identifier.
Ensures that email addresses are unique.
Custom Manager:

Centralizes logic for creating and managing users, allowing for customization (e.g., normalizing email addresses).
Password Handling:

set_password hashes the password before saving it, ensuring secure storage.
Admin Integration:

To manage the custom user model in the Django admin, you can define a custom admin class:
python
Copy code
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin as BaseUserAdmin
from app_name.models import User

@admin.register(User)
class UserAdmin(BaseUserAdmin):
    ordering = ['id']
    list_display = ['email', 'name']
    fieldsets = (
        (None, {'fields': ('email', 'password')}),
        ('Personal Info', {'fields': ('name',)}),
        ('Permissions', {'fields': ('is_active', 'is_staff', 'is_superuser')}),
        ('Important dates', {'fields': ('last_login',)}),
    )
    add_fieldsets = (
        (None, {
            'classes': ('wide',),
            'fields': ('email', 'password1', 'password2', 'name'),
        }),
    )
Extending the Model
Additional Fields:

Add new fields for your application's requirements:
python
Copy code
phone_number = models.CharField(max_length=15, null=True, blank=True)
create_superuser Method:

Add a method in the UserManager to create superusers:
python
Copy code
def create_superuser(self, email, password=None, **extra_fields):
    """Create and save a new superuser."""
    extra_fields.setdefault('is_staff', True)
    extra_fields.setdefault('is_superuser', True)

    if not extra_fields.get('is_staff'):
        raise ValueError('Superuser must have is_staff=True.')
    if not extra_fields.get('is_superuser'):
        raise ValueError('Superuser must have is_superuser=True.')

    return self.create_user(email, password, **extra_fields)
Example Workflow
Creating a Regular User:

python
Copy code
user = User.objects.create_user(
    email='user@example.com',
    password='password123',
    name='Regular User'
)
Creating a Superuser:

python
Copy code
superuser = User.objects.create_superuser(
    email='admin@example.com',
    password='adminpass123',
    name='Admin User'
)
Authentication:

Use Django's authentication system to log in the user with their email and password:
python
Copy code
from django.contrib.auth import authenticate

user = authenticate(email='user@example.com', password='password123')
if user:
    print("Authentication successful!")
else:
    print("Invalid credentials.")
This custom user model provides flexibility and better control, especially when email is preferred over usernames for authentication. Let me know if you'd like more details or examples



----------------------------

normalize email addresses

def test_new_user_email_normalized(self):
        """Test email is normalized for new users."""
        sample_emails = [
            ['test1@EXAMPLE.com', 'test1@example.com'],
            ['Test2@Example.com', 'Test2@example.com'],
            ['TEST3@EXAMPLE.com', 'TEST3@example.com'],
            ['test4@example.COM', 'test4@example.com'],
        ]
        for email, expected in sample_emails:
            user = get_user_model().objects.create_user(email, 'sample123')
            self.assertEqual(user.email, expected)
ChatGPT said:
ChatGPT
Test Case Explanation:
The test_new_user_email_normalized method is designed to verify that the email addresses are normalized when a new user is created in Django.

Key Concepts:
Normalization in this context refers to converting an email address to a consistent format. Specifically, it involves converting the domain part of the email to lowercase, as email addresses are case-insensitive in practice (e.g., EXAMPLE.com is the same as example.com).

The create_user method in the UserManager is expected to normalize the email before saving it in the database, ensuring consistent formatting.

Step-by-Step Breakdown of the Test:
Define Test Data: The sample_emails list contains pairs of:

Input email (with varied cases).
Expected email (normalized to lowercase domain).
Example:

python
Copy code
['test1@EXAMPLE.com', 'test1@example.com']
The input email is test1@EXAMPLE.com, but the expected normalized email is test1@example.com.
Loop Over Test Data:

The test iterates over the sample_emails list using a for loop, which checks each pair of input and expected email.
python
Copy code
for email, expected in sample_emails:
    user = get_user_model().objects.create_user(email, 'sample123')
    self.assertEqual(user.email, expected)
For each email:

Create a user: It calls create_user from the custom UserManager, passing in the email and a password ('sample123').
Assert that the email is normalized: After the user is created, the test checks whether the email stored in the user object matches the expected normalized email.
What Happens in create_user: In your custom UserManager, the create_user method should normalize the email before saving it to the database. If the create_user method has normalization logic implemented like this:

python
Copy code
def create_user(self, email, password=None, **extra_fields):
    """Create, save and return a new user."""
    if email:
        email = self.normalize_email(email)
    user = self.model(email=email, **extra_fields)
    user.set_password(password)
    user.save(using=self._db)
    return user
The normalize_email method should convert the email to lowercase for consistency.

Expected Outcome: For each email in sample_emails, the test checks if the email is normalized (converted to lowercase) and matches the expected result.

Purpose of the Test:
This test ensures that emails are normalized before they are stored in the database, which:

Prevents multiple entries of the same email with different casing.
Ensures consistency and avoids potential authentication issues.


--------------------------

def test_new_user_without_email_raises_error(self):
        """Test that creating a user without an email raises a ValueError."""
        with self.assertRaises(ValueError):
            get_user_model().objects.create_user('', 'test123')
ChatGPT said:
ChatGPT
Test Case Explanation:
The test_new_user_without_email_raises_error method is designed to test that creating a user without providing an email raises a ValueError.

Step-by-Step Breakdown of the Test:
Test Description: The docstring ("""Test that creating a user without an email raises a ValueError.""") clearly states the expected behavior: attempting to create a user with an empty email should result in a ValueError.

Test Code:

python
Copy code
with self.assertRaises(ValueError):
    get_user_model().objects.create_user('', 'test123')
with self.assertRaises(ValueError):: This is a context manager that expects the enclosed code to raise a ValueError. If a ValueError is raised, the test passes. If not, the test will fail.
get_user_model().objects.create_user('', 'test123'): This line attempts to create a user with an empty email ('') and a password ('test123').
What Happens in create_user:

In the UserManager's create_user method, the email is checked before creating the user. Here's a typical implementation:
python
Copy code
def create_user(self, email, password=None, **extra_fields):
    """Create, save and return a new user."""
    if not email:
        raise ValueError('The Email field must be set')  # Raises error if email is empty
    email = self.normalize_email(email)
    user = self.model(email=email, **extra_fields)
    user.set_password(password)
    user.save(using=self._db)
    return user
If the email is an empty string (''), a ValueError is raised with the message: 'The Email field must be set'.
Expected Outcome:

The test should pass if a ValueError is raised when trying to create a user without an email.
If no error is raised, or if an error of a different type is raised, the test will fail.
Purpose of the Test:
This test ensures that your application correctly handles the case where a user tries to create an account without providing an email. Since the email is a crucial identifier for the user, the test guarantees:

An empty email is not allowed.
The appropriate error is raised in such cases.
What Happens if the Test Fails?
If this test fails:

Check your create_user method to ensure that it properly checks for an empty email and raises the appropriate ValueError.
Example Test Output:
Success: If the ValueError is raised as expected, the test will pass.
Failure: If no error is raised or an error other than ValueError is raised, the test will fail.


---------------------------------


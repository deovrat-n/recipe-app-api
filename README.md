# recipe-app-api
version: "3.9"

services:
  app:
    build:
      context: .
      args:
        - DEV=true
    ports:
      - "8000:8000"
    volumes:
      - ./app:/app
    command: >
      sh -c "python manage.py runserver 0.0.0.0:8000"
    environment:
      - DB_HOST=db
      - DB_NAME=devdb
      - DB_USER=devuser
      - DB_PASS=changeme
    depends_on:
      - db

  db:
    image: postgres:13-alpine
    volumes:
      - dev-db-data:/var/lib/postgresql/data
    environment:
      - POSTGRES_DB=devdb
      - POSTGRES_USER=devuser
      - POSTGRES_PASSWORD=changeme

volumes:
  dev-db-data:
This is a docker-compose.yml file that defines two services: an application (app) and a PostgreSQL database (db), configured for development purposes.

Breakdown:
Version:

Specifies the Docker Compose file format (3.9).
Services:

app:
build: Specifies the context and build arguments. It builds the application from the current directory (.) and sets an argument DEV=true.
ports: Maps port 8000 of the container to port 8000 on the host, so the app is accessible via localhost:8000.
volumes: Mounts the local ./app directory to /app inside the container for live code reloading.
command: Runs the Django development server (python manage.py runserver 0.0.0.0:8000) within the container.
environment: Sets environment variables for the database connection (host, name, user, password).
depends_on: Ensures the db service is started before the app service.
db:
image: Uses the official postgres:13-alpine Docker image for PostgreSQL.
volumes: Persists the database data in a named volume (dev-db-data) to retain data across container restarts.
environment: Sets environment variables for the PostgreSQL database (database name, user, password).
Volumes:

dev-db-data: A named volume to store PostgreSQL data persistently.
How it works:
When you run docker-compose up, the app and database containers are created.
The app connects to the database using the environment variables for DB_HOST, DB_NAME, DB_USER, and DB_PASS.
The database service initializes a PostgreSQL instance with the provided credentials.

In Docker, volumes are used to persist data outside the container’s filesystem, ensuring data is retained even when containers are stopped or recreated.

In your docker-compose.yml, the volume dev-db-data is used to store PostgreSQL database data persistently. It's mapped to /var/lib/postgresql/data inside the container, where PostgreSQL stores its data. This setup ensures that the database data is retained across container restarts, providing data persistence, isolation, and better performance compared to bind mounts. Volumes are managed by Docker and can be inspected, backed up, and reused across container lifecycles.



psycopg2 is a popular PostgreSQL adapter for Python. It allows Python applications to connect to and interact with PostgreSQL databases using SQL queries.

Key Features:
Database Connection: It facilitates connecting to a PostgreSQL database from a Python application.
Query Execution: Allows you to execute SQL queries (e.g., SELECT, INSERT, UPDATE, DELETE) and retrieve results.
Cursor Object: Uses a cursor to interact with the database, execute queries, and fetch results.
Transaction Support: Supports database transactions, including committing or rolling back changes.




DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql',
        'HOST': os.environ.get('DB_HOST'),
        'NAME': os.environ.get('DB_NAME'),
        'USER': os.environ.get('DB_USER'),
        'PASSWORD': os.environ.get('DB_PASS'),
    }
}
For a local development setup, your environment variables (e.g., in a .env file or Docker) might look like this:


DB_HOST=localhost
DB_NAME=mydatabase
DB_USER=myuser
DB_PASS=mypassword
In a Docker environment, these variables could be passed through the docker-compose.yml file as shown earlier.

Conclusion:
This configuration ensures that your Django application can connect to a PostgreSQL database using dynamic, environment-specific credentials, making it easier to deploy the application in different environments (development, staging, production) without hardcoding sensitive information.



Code Flow of wait_for_db Command
The Command Code
Here’s a likely implementation of wait_for_db based on the test cases:

python

import time
from django.core.management.base import BaseCommand
from psycopg2 import OperationalError as Psycopg2OpError
from django.db.utils import OperationalError


class Command(BaseCommand):
    """Django command to pause execution until database is available."""

    def handle(self, *args, **options):
        self.stdout.write('Waiting for database...')
        db_up = False

        while not db_up:
            try:
                # The 'check' method verifies the database connection.
                self.check(databases=['default'])
                db_up = True
            except (Psycopg2OpError, OperationalError):
                # If an error occurs, print a message and wait for 1 second.
                self.stdout.write('Database unavailable, waiting 1 second...')
                time.sleep(1)

        self.stdout.write(self.style.SUCCESS('Database available!'))
Flow of Execution
Command Invocation:

The command is run using python manage.py wait_for_db.
The handle method is the entry point for the command's logic.
Initial Output:

It prints "Waiting for database..." to the console.
Database Connection Check:

The self.check(databases=['default']) method checks if the default database is available.
If the database is available, the method completes successfully, and the db_up flag is set to True, exiting the loop.
If the database is unavailable, it raises one of the following exceptions:
Psycopg2OpError (specific to PostgreSQL).
OperationalError (a general Django error for database issues).
Retry Logic:

When an exception is raised, the command prints "Database unavailable, waiting 1 second..." and pauses for 1 second using time.sleep(1).
The loop continues until the database becomes available.
Success Message:

Once the database connection is successful, it prints "Database available!" in green (styled output).
How the check Method Works
What is self.check?

self.check is a method inherited from the BaseCommand class in Django.
It validates the state of the Django project, including database connections.
By passing databases=['default'], it specifically checks the connection to the default database.
Internally:

It attempts to connect to the database.
If the connection fails, it raises OperationalError or a database-specific error like Psycopg2OpError.
How the Tests Work with the Command
1. Mocking check
The @patch('core.management.commands.wait_for_db.Command.check') decorator mocks the check method in the wait_for_db command. Instead of calling the real method, it replaces it with a mock object.

This allows the test to simulate the database being ready or unavailable without relying on an actual database.
2. test_wait_for_db_ready
python
Copy code
@patch('core.management.commands.wait_for_db.Command.check')
def test_wait_for_db_ready(self, patched_check):
    """Test waiting for database if database ready."""
    patched_check.return_value = True  # Simulate database being ready.

    call_command('wait_for_db')  # Run the command.

    # Verify 'check' was called once with the correct arguments.
    patched_check.assert_called_once_with(databases=['default'])
Simulated Scenario:
The database is ready on the first attempt (patched_check.return_value = True).
Flow:
The call_command('wait_for_db') runs the command.
Since check is mocked to always return True, the loop in wait_for_db exits immediately.
The test verifies that check was called exactly once with databases=['default'].
3. test_wait_for_db_delay
python
Copy code
@patch('time.sleep')
@patch('core.management.commands.wait_for_db.Command.check')
def test_wait_for_db_delay(self, patched_check, patched_sleep):
    """Test waiting for database when getting OperationalError."""
    patched_check.side_effect = [Psycopg2OpError] * 2 + \
                                [OperationalError] * 3 + [True]  # Simulate failures then success.

    call_command('wait_for_db')  # Run the command.

    # Assert 'check' was called 6 times (2 + 3 + 1).
    self.assertEqual(patched_check.call_count, 6)
    # Verify the last call to 'check' was with 'databases=['default']'.
    patched_check.assert_called_with(databases=['default'])
Simulated Scenario:
The database is unavailable for 5 attempts:
Psycopg2OpError is raised twice.
OperationalError is raised three times.
On the 6th attempt, check returns True, simulating a successful connection.
Flow:
The call_command('wait_for_db') runs the command.
On each failed attempt, the wait_for_db command retries after calling time.sleep(1) (mocked in the test to avoid actual delay).
After 5 failed attempts, the command succeeds, and the loop exits.
Assertions:
patched_check.call_count == 6: Confirms that check was called 6 times.
patched_check.assert_called_with(databases=['default']): Confirms that the check method was called with the correct arguments.
End-to-End Workflow
Command:

Continuously tries to connect to the database using self.check(databases=['default']).
Retries on exceptions (Psycopg2OpError, OperationalError) until the database is ready.
Tests:

Simulate database states (ready or unavailable) using the mocked check method.
Validate the command's retry logic and ensure it handles errors correctly.


--------------------------------



Database Migration in Django
In Django, database migrations are a way to propagate changes made to your models (e.g., adding, deleting, or modifying fields) to the database schema. Django uses the migrations framework to handle this process automatically and ensure your database schema stays in sync with your application's models.

How Migrations Work
Model Changes:

When you make changes to your models (e.g., add a field, modify a field, delete a field), Django detects these changes.
Migration Files:

Django creates a migration file that contains instructions to apply the changes to the database schema.
Apply Migrations:

Once a migration file is created, you apply it to the database using Django commands.
Key Commands
1. Make Migrations
Command: python manage.py makemigrations
Purpose:
Detects changes in your models and generates a migration file in the migrations folder of your app.
Example:
bash
Copy code
python manage.py makemigrations
Output:
bash
Copy code
Migrations for 'myapp':
  myapp/migrations/0001_initial.py
    - Create model MyModel
2. Apply Migrations
Command: python manage.py migrate
Purpose:
Applies the migration files to the database, updating its schema.
Example:
bash
Copy code
python manage.py migrate
Output:
Copy code
Applying myapp.0001_initial... OK
3. Check Migration Status
Command: python manage.py showmigrations
Purpose:
Lists all available migrations and their current status (applied or unapplied).
Example:
bash
Copy code
python manage.py showmigrations
Output:
css
Copy code
admin
 [X] 0001_initial
 [ ] 0002_auto_20240220_1234
4. Rollback a Migration
Command: python manage.py migrate <app_name> <migration_name>
Purpose:
Rolls back to a specific migration state.
Example:
bash
Copy code
python manage.py migrate myapp 0001
Example Workflow
1. Create a Model
python
Copy code
# models.py
from django.db import models

class MyModel(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()
2. Generate Migration File
bash
Copy code
python manage.py makemigrations
Output:

bash
Copy code
Migrations for 'myapp':
  myapp/migrations/0001_initial.py
    - Create model MyModel
3. Apply the Migration
bash
Copy code
python manage.py migrate
Output:

Copy code
Applying myapp.0001_initial... OK
4. Modify the Model
python
Copy code
# models.py
class MyModel(models.Model):
    name = models.CharField(max_length=100)
    age = models.IntegerField()
    email = models.EmailField(null=True)
5. Generate and Apply New Migration
bash
Copy code
python manage.py makemigrations
python manage.py migrate
Output:

bash
Copy code
Migrations for 'myapp':
  myapp/migrations/0002_auto_20240222_1234.py
    - Add field email to MyModel

Applying myapp.0002_auto_20240222_1234... OK
Understanding Migration Files
Migration files are Python scripts that describe the changes in the database schema. For example:

python
Copy code
# myapp/migrations/0001_initial.py
from django.db import migrations, models

class Migration(migrations.Migration):

    initial = True

    dependencies = []

    operations = [
        migrations.CreateModel(
            name='MyModel',
            fields=[
                ('id', models.AutoField(auto_created=True, primary_key=True)),
                ('name', models.CharField(max_length=100)),
                ('age', models.IntegerField()),
            ],
        ),
    ]
Best Practices
Commit Migrations to Version Control:

Always include migration files in your repository to ensure everyone has the same schema.
Apply Migrations in Deployment:

Use the migrate command as part of your deployment process to update the database schema.
Avoid Manual Edits:

Never manually edit migration files unless absolutely necessary.
Use null=True for Optional Fields:

When adding a new field to an existing table, always set null=True or provide a default to avoid breaking the migration.
Common Issues and Fixes
Missing Migration File:

Issue: Forgetting to run makemigrations.
Fix: Run python manage.py makemigrations.
Migration Dependency Errors:

Issue: Circular or missing dependencies in migration files.
Fix: Check the dependencies attribute in migration files and resolve conflicts.
Database Errors (e.g., Duplicate Column):

Issue: Re-running migrations that have already been applied.
Fix: Roll back the migration or fake the migration using python manage.py migrate --fake.
Out-of-Sync Migrations:

Issue: The database schema doesn't match the migration files.
Fix: Use python manage.py showmigrations and resolve discrepancies by reapplying or faking migrations.
Key Concepts
Initial Migration: The first migration for an app, usually includes creating tables.
Dependencies: Specify which migrations must be applied before this one.
Operations: Actions performed in the migration, such as creating models, adding fields, or modifying data.
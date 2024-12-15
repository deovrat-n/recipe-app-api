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
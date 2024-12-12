# recipe-app-api
Recipie API project


Docker Hub

Docker Hub is a cloud-based container registry that allows users to store, share, and access Docker images. It offers public and private repositories, official and community-contributed images, automated builds, webhooks, and image versioning. Users can push, pull, and search for images via the Docker CLI or web interface, making it a valuable tool for development, testing, and deploying containerized applications.


GitHub Actions

GitHub Actions is a CI/CD platform that automates workflows directly in GitHub. It triggers tasks (e.g., testing, building, deploying) based on events like push or pull_request. Workflows are defined in YAML files and consist of jobs and steps, which can use reusable actions from the GitHub Marketplace. Key features include parallel jobs, matrix builds, and deep GitHub integration. It’s commonly used for automated testing, deployment, and code quality checks.

Linting

Linting is the automated process of analyzing code to detect errors, enforce coding standards, and promote best practices. Tools like ESLint (JavaScript), Pylint (Python), and Stylelint (CSS) provide early feedback, improve code quality, and ensure consistency in projects. Linters can be customized for specific guidelines and are essential for maintaining readable, error-free code.

This Dockerfile creates a secure, minimal container for running a Django application with Python 3.9 and Alpine Linux. Key actions include:

Installing dependencies in a virtual environment.
Running the application with a non-root user.
Exposing port 8000 for communication with the container.
The result is a small, efficient, and secure container tailored for a Django project.

Docker-compose 


Key Features:
Builds Image: Uses the local Dockerfile for building.
Port Mapping: Maps host port 8000 to container port 8000.
Live Sync: Syncs the ./app directory with /app in the container.
Runs Server: Starts the Django server at 0.0.0.0:8000.




How They Work Together
Dockerfile: Creates the image for a single service (e.g., Django backend or a database).
Docker Compose: Uses that image (or multiple images) to manage containers, networks, and volumes.
For example:

Dockerfile defines how to build a Django app image.
docker-compose.yml launches the app and links it with a database.


What is Flake8?
Flake8 is a Python tool for:

PEP 8 compliance: Enforces Python's style guide.
Linting: Detects issues like unused imports or undefined variables.
Complexity checks: Warns about overly complex code.

To Execute - docker-compose run --rm app sh -c "flake8"

Explanation:
docker-compose run:

Runs a one-time command in the container specified in the docker-compose.yml file.
--rm:

Removes the container after the command finishes, keeping your system clean.
app:

Refers to the service named app in the docker-compose.yml.
sh -c "flake8":

Executes the flake8 command inside a shell (sh) in the container.
docker-compose run --rm app sh -c "django-admin startproject app ."
Breakdown:
docker-compose run: Runs a one-time command in a container specified in the docker-compose.yml file.

--rm: Removes the container after the command finishes to avoid leaving unnecessary stopped containers.

app: The name of the service defined in your docker-compose.yml file where the command will run (likely configured to use a Django-compatible image).

sh -c "django-admin startproject app .":

Runs a shell (sh) to execute the given command.
django-admin startproject app . creates a new Django project named app in the current directory (.).
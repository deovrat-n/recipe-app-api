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

---
name: Checks

on: [push]

jobs:
  test-lint:
    name: Test and Lint
    runs-on: ubuntu-20.04
    steps:
      - name: Login to Docker Hub
        uses: docker/login-action@v1
        with:
          username: ${{ secrets.DOCKERHUB_USER }}
          password: ${{ secrets.DOCKERHUB_TOKEN }}
      - name: Checkout
        uses: actions/checkout@v2
      - name: Test
        run: docker-compose run --rm app sh -c "python manage.py test"
      - name: Lint
        run: docker-compose run --rm app sh -c "flake8"

This GitHub Actions workflow does the following:

Triggered by: Any code push.
Environment: Runs on Ubuntu 20.04.
Steps:
Logs into Docker Hub using credentials.
Checks out the repository code.
Runs tests using python manage.py test inside a Docker container.
Checks Python code style using flake8.
It ensures code quality (linting) and correctness (testing) with every push.
Hi
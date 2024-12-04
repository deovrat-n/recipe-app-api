# recipe-app-api
Recipie API project


Docker Hub

Docker Hub is a cloud-based container registry that allows users to store, share, and access Docker images. It offers public and private repositories, official and community-contributed images, automated builds, webhooks, and image versioning. Users can push, pull, and search for images via the Docker CLI or web interface, making it a valuable tool for development, testing, and deploying containerized applications.


GitHub Actions

GitHub Actions is a CI/CD platform that automates workflows directly in GitHub. It triggers tasks (e.g., testing, building, deploying) based on events like push or pull_request. Workflows are defined in YAML files and consist of jobs and steps, which can use reusable actions from the GitHub Marketplace. Key features include parallel jobs, matrix builds, and deep GitHub integration. It’s commonly used for automated testing, deployment, and code quality checks.

Linting

Linting is the automated process of analyzing code to detect errors, enforce coding standards, and promote best practices. Tools like ESLint (JavaScript), Pylint (Python), and Stylelint (CSS) provide early feedback, improve code quality, and ensure consistency in projects. Linters can be customized for specific guidelines and are essential for maintaining readable, error-free code.

Dockerfile

Using Alpine Linux as the base image for Docker containers is a popular choice due to several key advantages, especially in terms of size and security. Here’s why you might choose Alpine over other base images:

1. Smaller Image Size
Alpine is a minimal, lightweight Linux distribution, typically around 5MB in size. In contrast, many other base images (e.g., python:3.x or ubuntu) are much larger (often hundreds of MB).
A smaller image size leads to:
Faster build times (less data to download and process).
Reduced storage requirements.
Faster deployments due to smaller images.
2. Improved Security
Alpine is known for its security features:
It uses musl libc and busybox for core utilities, which are more lightweight and often more secure than the alternatives.
Alpine has a strong focus on security with minimal bloat, reducing the potential attack surface of your container.
The Alpine image is also maintained and updated regularly to patch security vulnerabilities.
3. Better Control
With Alpine, you have more control over what’s included in your image. Since Alpine starts minimal, you can install only the libraries and dependencies you need, leading to a cleaner, more efficient container.
4. Faster Start-Up Times
Due to the smaller size and fewer components, containers built on Alpine generally start faster than those built on larger base images.

When Not to Use Alpine
Compatibility Issues: Some Python packages or binaries may not work properly on Alpine due to differences between Alpine's musl libc and other distributions' glibc.
Debugging Complexity: The minimal environment may make debugging harder if something goes wrong, as many utilities (e.g., debugging tools) are not installed by default.

This Dockerfile creates a lightweight Docker image for a Django application using Python 3.9 on Alpine 3.13. Key steps include:

Base Image: Starts with python:3.9-alpine3.13, a small, efficient image based on Alpine Linux.
Environment Setup: Sets PYTHONUNBUFFERED=1 for unbuffered Python output.
File Copying: Copies requirements.txt and the Django app's source code into the container.
Virtual Environment: Creates a Python virtual environment in /py, installs dependencies from requirements.txt, and removes temporary files to reduce the image size.
Security: Adds a non-root user (django-user) to run the application securely.
Path Configuration: Adds the virtual environment's bin directory to the PATH for using Python and pip from the virtual environment.
Expose Port: Exposes port 8000 for the Django app.



This Dockerfile describes the process of building a Docker image for a Django application, using Python 3.9 on Alpine 3.13. Here's a breakdown of what each part does:


Base Image:
FROM python:3.9-alpine3.13
Starts with Python 3.9 on Alpine 3.13 as the base image. Alpine is a minimal, lightweight Linux distribution that helps reduce the image size.
Label:
LABEL maintainer="deovratapidemo.com"
Adds a maintainer label, which can be used for metadata about the image. In this case, it specifies the maintainer's website.
Python Environment Configuration:
ENV PYTHONUNBUFFERED 1
PYTHONUNBUFFERED=1 ensures that Python output is immediately written to the console (unbuffered), which is useful for logging in Docker containers.
Copying Files:


COPY ./requirements.txt /tmp/requirements.txt
COPY ./app /app
COPY ./requirements.txt: Copies the requirements.txt file from the local machine into the container at /tmp/requirements.txt.
COPY ./app: Copies the entire application (./app) into the /app directory of the container.
Working Directory:
WORKDIR /app
Sets /app as the working directory in the container. All subsequent commands will be executed from this directory.
Expose Port:

EXPOSE 8000
Exposes port 8000, which is the default port for Django development servers. It allows communication with the container on this port.
Installing Dependencies:


RUN python -m venv /py && \
    /py/bin/pip install --upgrade pip && \
    /py/bin/pip install -r /tmp/requirements.txt && \
    rm -rf /tmp && \
    adduser \
        --disabled-password \
        --no-create-home \
        django-user
python -m venv /py: Creates a Python virtual environment in the /py directory.
/py/bin/pip install --upgrade pip: Upgrades pip inside the virtual environment.
/py/bin/pip install -r /tmp/requirements.txt: Installs the dependencies listed in the requirements.txt file.
rm -rf /tmp: Removes the temporary requirements.txt file to reduce image size.
adduser: Adds a non-root user named django-user to the container for better security. This user is disabled and has no home directory, improving the container's security posture.
Updating PATH:


ENV PATH="/py/bin:$PATH"
Adds the virtual environment’s bin directory to the system's PATH so the Python and pip commands from the virtual environment are used, rather than the global Python installation.
Switching User:


USER django-user
Changes the user to django-user, ensuring the Django application runs under a non-root user for security.




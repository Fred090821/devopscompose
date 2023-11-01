# Use a lightweight base image
FROM python:3.8-alpine

LABEL authors="Fred Assi"
LABEL maintainer="assifred2005@gmail.com"

# Set the working directory
WORKDIR /pythontest

# Copy application files
COPY . /pythontest

# Install curl and system dependencies
RUN apk update && \
    apk upgrade && \
    apk --no-cache add build-base libffi-dev openssl-dev

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Command to run the application
CMD ["pytest", "docker_tests.py"]
# Use a lightweight base image
FROM python:3.8-alpine

LABEL authors="Fred Assi"
LABEL maintainer="assifred2005@gmail.com"

# Set the working directory
WORKDIR /app/python

# Copy application files
COPY . /app/python


# Install curl and system dependencies
RUN apk --no-cache add curl && \
    apk update && \
    apk upgrade && \
    apk --no-cache add build-base libffi-dev openssl-dev

# Create a non-root user
RUN adduser -D myuser

# Set permissions for application files
RUN chmod 644 requirements.txt
RUN chmod 644 rest_app.py

# Install Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Expose the port your application will run on
EXPOSE 5003

# Create a volume for logs
VOLUME /app/logs

# Command to run the application
CMD ["python3", "rest_app.py"]
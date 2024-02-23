# Docker-Django-Linux-Tutorial
A short tutorial about deploying Django website using Docker.

## 1. Preparing Django
1) Copy or create a Django project.
2) In ALLOWED_HOSTS, add your IP or domain. You can also add "*" (not safe for a production server!).
```
ALLOWED_HOSTS = ["*"]
```
4) Add a requirements.txt file.
Minimum requirements:
```
Django==4.2.1
gunicorn==20.1.0
```

## 2. Working with Docker
### 1) At the same level as the Django project folder, create a "Dockerfile".
``` 
# Use the official Python image
FROM python:3.8
# Set the environment variables for Python
ENV PYTHONUNBUFFERED 1
# Set the working directory in the container
WORKDIR /app
# Copy the dependency file and install them
COPY projdjango/requirements.txt /app/
RUN pip install --no-cache-dir -r requirements.txt
# Copy the app source code into the container
COPY projdjango /app/
# Run Gunicorn
CMD ["gunicorn", "projdjango.wsgi:application", "--bind", "0.0.0.0:8000"]
```
Don't forget to replace projdjango with the name of your project!
### 2) Create a Docker image
```
docker build -t djangoimage1 .
```
### 3) Launch the container
```
docker run -d --name djangocontainer1 -p 8001:8000 djangoimage1
```

## 3. Configuring Nginx:
1) Create a new file in /etc/nginx/sites-available/newfile
```
server {
    listen 80;
    server_name your_server_domain_or_IP;

    location / {
        proxy_pass http://127.0.0.1:8001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
    location /static/ {
        # full path from "/" dir to static folder, for example: /root/CRST/Craft-Site/static/; 
        alias /full/path/from/slash/root/dir/; 
        expires 30d;
        add_header Cache-Control "public, max-age=2592000";
    }

    # If you have static files:
    location /static/ {
        alias /path/to/your/static/folder/on/host/;  # Specify the actual path to the static files
    }
}

```
### 2) Restart nginx to update the configurations
```
sudo systemctl restart nginx
```
### 3) Set read permissions for static files for each directory in the path
```
chmod +r directory/.../static
...
chmod +r directory
```
## 4) Remember to adjust file paths and other specifics as needed for your actual setup!
## 5) SSL 
https://www.8host.com/blog/poluchenie-ssl-sertifikatov-s-pomoshhyu-certbot/z

## Useful Commands:
* sudo systemctl reload nginx
* chmod +x /dir
* ls -l /path/to/file/file.js
* docker logs djangoapp
* sudo tail -f /var/log/nginx/error.log

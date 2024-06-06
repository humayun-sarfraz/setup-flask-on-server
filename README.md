# setup-flask-on-server



#Configure Gunicorn and Ngnix
# Flask Application Deployment Guide

This guide provides step-by-step instructions to run your Flask application continuously and manage multiple applications on the same server using Gunicorn and Nginx. This setup will also allow you to remove the port number from the URL.

## Verifying and Terminating Processes

### Verify the Port is Free
```sh
sudo lsof -i :8000

Terminate the Process (if you want to use the same port)

sh

sudo kill <PID>

Run Gunicorn on a New Port

sh

gunicorn --bind 127.0.0.1:<another_port> wsgi:app

Installation and Configuration
1. Install Required Software

First, install Gunicorn and Nginx if they are not already installed:

sh

sudo apt update
sudo apt install python3-pip nginx
pip3 install gunicorn

2. Configure Gunicorn

Navigate to your application directory and test running your Flask app with Gunicorn:

sh

cd /var/www/your_application
gunicorn --bind 127.0.0.1:8000 wsgi:app

Here, wsgi:app assumes that your Flask application instance is named app and is located in a file named wsgi.py.
3. Create a Systemd Service for Gunicorn

Create a new service file for your Flask application:

sh

sudo nano /etc/systemd/system/your_application.service

Add the following content to the service file:

ini

[Unit]
Description=Gunicorn instance to serve your_application
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/your_application
ExecStart=/usr/local/bin/gunicorn --workers 3 --bind unix:/var/www/your_application/your_application.sock -m 007 wsgi:app

[Install]
WantedBy=multi-user.target

Replace /var/www/your_application with the path to your application directory and adjust the User and Group if necessary.
4. Start and Enable the Gunicorn Service

Enable and start the service:

sh

sudo systemctl start your_application
sudo systemctl enable your_application

5. Configure Nginx

Create a new Nginx configuration file for your application:

sh

sudo nano /etc/nginx/sites-available/your_application

Add the following content:

nginx

server {
    listen 80;
    server_name 146.190.130.129;

    location / {
        include proxy_params;
        proxy_pass http://unix:/var/www/your_application/your_application.sock;
    }
}

Create a symbolic link to enable this configuration:

sh

sudo ln -s /etc/nginx/sites-available/your_application /etc/nginx/sites-enabled

Test the Nginx configuration and restart the service:

sh

sudo nginx -t
sudo systemctl restart nginx

6. Allow Multiple Applications

Repeat the above steps for each Flask application you want to host, making sure each application has its own unique Gunicorn socket and Nginx configuration.
7. Secure Your Server (Optional)

Consider securing your server with HTTPS using Let's Encrypt:

sh

sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx

Follow the prompts to set up HTTPS.

By following these steps, you will have your Flask application running continuously with Gunicorn and served by Nginx. You can access it via your IP address without a port number, and you can add additional applications as needed.

csharp


Save this content in a file named `README.md`. This will serve as a guide for deploying Flask applications using Gunicorn and Nginx.

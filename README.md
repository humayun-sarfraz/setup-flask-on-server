# setup-flask-on-server

Got it. Here's the adjusted guide based on your specifications:
Step 1 — Installing the Components from the Ubuntu Repositories

Update the package index and install the necessary packages:

bash

sudo apt update
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools

Step 2 — Creating a Python Virtual Environment

Install python3-venv:

bash

sudo apt install python3-venv

Create a parent directory for your Flask project and navigate into it:

bash

sudo mkdir -p /var/www/trackio/api
cd /var/www/trackio/api

Create a virtual environment:

bash

sudo python3 -m venv trackio-api

Activate the virtual environment:

bash

source trackio-api/bin/activate

Step 3 — Setting Up a Flask Application

Install wheel, Flask, and Gunicorn:

bash

pip install wheel
pip install gunicorn flask

Create a simple Flask application:

bash

nano /var/www/trackio/api/app.py

Add the following code:

python

from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5003)

Allow access to port 5003 (adjust firewall settings if necessary):

bash

sudo ufw allow 5003

Test the Flask application:

bash

python app.py

Visit http://146.190.130.129:5003 to see the Flask app.
Step 4 — Configuring Gunicorn

Create a WSGI entry point:

bash

nano /var/www/trackio/api/wsgi.py

Add the following code:

python

from app import app

if __name__ == "__main__":
    app.run()

Test Gunicorn:

bash

cd /var/www/trackio/api
gunicorn --bind 0.0.0.0:5003 wsgi:app

Visit http://146.190.130.129:5003 to see the Flask app.
Step 5 — Creating a systemd Service

Create a systemd service unit file:

bash

sudo nano /etc/systemd/system/trackio-api.service

Add the following content:

ini

[Unit]
Description=Gunicorn instance to serve trackio-api
After=network.target

[Service]
User=root
Group=root
WorkingDirectory=/var/www/trackio/api
Environment="PATH=/var/www/trackio/api/trackio-api/bin"
ExecStart=/var/www/trackio/api/trackio-api/bin/gunicorn --workers 3 --bind 0.0.0.0:5003 wsgi:app

[Install]
WantedBy=multi-user.target

Start and enable the service:

bash

sudo systemctl start trackio-api
sudo systemctl enable trackio-api

Check the status:

bash

sudo systemctl status trackio-api

Step 6 — Configuring Nginx

Create a new server block configuration file:

bash

sudo nano /etc/nginx/sites-available/trackio-api

Add the following content:

nginx

server {
    listen 80;
    server_name 146.190.130.129;

    location / {
        proxy_pass http://127.0.0.1:5003;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}

Enable the configuration by linking the file to sites-enabled:

bash

sudo ln -s /etc/nginx/sites-available/trackio-api /etc/nginx/sites-enabled

Test the Nginx configuration:

bash

sudo nginx -t

Restart Nginx:

bash

sudo systemctl restart nginx

Step 7 — Adjusting Firewall

Remove the rule for port 5003 and allow full access to the Nginx server:

bash

sudo ufw delete allow 5003
sudo ufw allow 'Nginx Full'

Step 8 — Securing the Application

Install Certbot and obtain an SSL certificate:

bash

sudo apt install python3-certbot-nginx
sudo certbot --nginx -d 146.190.130.129

Select the option to redirect HTTP to HTTPS during the Certbot process.

Check the status:

bash

sudo ufw delete allow 'Nginx HTTP'

Verify the setup

Navigate to https://146.190.130.129 to see your Flask application secured with HTTPS.

This guide will help you set up your Flask application named "trackio-api" on port 5003, managed by Gunicorn, proxied by Nginx, and secured with SSL.



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

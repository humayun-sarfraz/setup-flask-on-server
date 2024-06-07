Flask Application Deployment on Ubuntu
Installing the Components from the Ubuntu Repositories
sh
Copy code
sudo apt update
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools
Creating a Python Virtual Environment
sh
Copy code
sudo apt install python3-venv

mkdir /var/www/<your-directory-name>
cd /var/www/<your-directory-name>

python3 -m venv <your-env-name>

source <your-env-name>/bin/activate
Setting Up a Flask Application
sh
Copy code
(your-env-name) pip install wheel

# Install Flask and Gunicorn
pip install gunicorn flask

# Create Sample Flask Application
(your-env-name) nano /var/www/<your-directory-name>/<your-file-name.py> # e.g., app.py
Sample Flask Application Code:

python
Copy code
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port='5001')
Testing the Flask Application
Allow access to the port if UFW firewall is enabled:
sh
Copy code
(your-env-name) sudo ufw allow 5001 # Change port if needed
Run the Flask application:
sh
Copy code
(your-env-name) python3 <your-file-name.py> # e.g., app.py
Output:

mathematica
Copy code
* Serving Flask app "myproject" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:5001/ (Press CTRL+C to quit)
Visit:

arduino
Copy code
http://your_server_ip:5001
Creating the WSGI Entry Point
sh
Copy code
(your-env-name) nano /var/www/<your-directory-name>/wsgi.py
WSGI File Code:

python
Copy code
from app import app

if __name__ == "__main__":
    app.run()
Configuring Gunicorn
sh
Copy code
(your-env-name) cd /var/www/<your-directory-name>/

(your-env-name) gunicorn --bind 0.0.0.0:5001 wsgi:app # Ensure correct port number
Output:

less
Copy code
[2020-05-20 14:13:00 +0000] [46419] [INFO] Starting gunicorn 20.0.4
[2020-05-20 14:13:00 +0000] [46419] [INFO] Listening at: http://0.0.0.0:5001 (46419)
[2020-05-20 14:13:00 +0000] [46419] [INFO] Using worker: sync
[2020-05-20 14:13:00 +0000] [46421] [INFO] Booting worker with pid: 46421
Visit:

arduino
Copy code
http://your_server_ip:5001
Deactivate Virtual Environment:

sh
Copy code
(your-env-name) deactivate
System File for Service
sh
Copy code
sudo nano /etc/systemd/system/<your_service_name>.service
Service File Code:

ini
Copy code
[Unit]
Description=Gunicorn instance to serve Flask Application
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/<your-directory-name>
Environment="PATH=/var/www/<your-directory-name>/<your-env-name>/bin"
ExecStart=/var/www/<your-directory-name>/<your-env-name>/bin/gunicorn --workers 3 --bind 127.0.0.1:5001 wsgi:app

[Install]
WantedBy=multi-user.target
sh
Copy code
sudo systemctl start <your_service_name>
sudo systemctl enable <your_service_name>
sudo systemctl status <your_service_name>
Configuring Nginx to Proxy Requests
sh
Copy code
sudo nano /etc/nginx/sites-available/<Your-project-name>
Nginx Configuration File:

nginx
Copy code
server {
    listen 80;
    server_name Your.IP.Goes.Here;

    location / {
        proxy_pass http://127.0.0.1:5001;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
    }
}
sh
Copy code
sudo ln -s /etc/nginx/sites-available/<Your-project-name> /etc/nginx/sites-enabled

# Test for syntax errors
sudo nginx -t

sudo systemctl restart nginx

sudo ufw delete allow 5001
sudo ufw allow 'Nginx Full'
Note: You may receive an HTTP 502 gateway error if Nginx cannot access Gunicorn’s socket file. Ensure the directory permissions are set correctly.

sh
Copy code
sudo chmod 755 /var/www
Checking Logs:

sh
Copy code
sudo less /var/log/nginx/error.log
sudo less /var/log/nginx/access.log
sudo journalctl -u nginx
sudo journalctl -u myproject
Securing the Application
sh
Copy code
sudo apt install python3-certbot-nginx
sudo certbot --nginx -d <your_domain> -d www.<your_domain>

# Update UFW rules
sudo ufw delete allow 'Nginx HTTP'
Replace placeholders like <your-directory-name>, <your-env-name>, <your-file-name.py>, <your_service_name>, Your.IP.Goes.Here, and <your_domain> with your actual values. This guide helps you set up a Flask application on an Ubuntu server, configure it with Gunicorn and Nginx, and secure it with Certbot.

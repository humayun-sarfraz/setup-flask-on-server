# setup-flask-on-server

########################################################
#Installing the Components from the Ubuntu Repositories#
########################################################
sudo apt update
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools

########################################################
######### Creating a Python Virtual Environment ########
########################################################
sudo apt install python3-venv

mkdir /var/www/<your-directory-name>
cd /var/www/<your-directory-name>

python3 -m venv <your-env-name>

source <your-env-name>/bin/activate


########################################################
############ Setting Up a Flask Application ############
########################################################

(your-env-name) pip install wheel

#Next, install Flask and Gunicorn:
pip install gunicorn flask

#create Sample File
(your-env-name) nano /var/www/<your-directory-name>/<your-file-name.py> # app.py

-------------------File Code Start-----------------
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port='5001')

-------------------File Code Ends-----------------

#you should have a UFW firewall enabled. To test the application, you need to allow access to port.
(your-env-name) sudo ufw allow 5000 #your port number if you have define different in .py file use that

#then

(your-env-name) python3 <your-file-name.py> #app.py

-------------------Your Output Like This Starts-----------------
Output
* Serving Flask app "myproject" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:5001/ (Press CTRL+C to quit)
-------------------Your Output Like This Ends-----------------

#Visit your server’s IP address followed by port number 
http://your_server_ip:5001


####### Now Creating the WSGI Entry Point ##########
(your-env-name) nano /var/www/<your-directory-name>/wsgi.py

------------File Code Starts----------
from app import app

if __name__ == "__main__":
    app.run()
------------File Code Ends----------

####################################################
##################  Configuring Gunicorn############
####################################################

(your-env-name) cd /var/www/<your-directory-name>/

(your-env-name) gunicorn --bind 0.0.0.0:5001 wsgi:app #make sure you are using correct port number.

-------------Output look like Starts---------------
Output
[2020-05-20 14:13:00 +0000] [46419] [INFO] Starting gunicorn 20.0.4
[2020-05-20 14:13:00 +0000] [46419] [INFO] Listening at: http://0.0.0.0:5001 (46419)
[2020-05-20 14:13:00 +0000] [46419] [INFO] Using worker: sync
[2020-05-20 14:13:00 +0000] [46421] [INFO] Booting worker with pid: 46421
-------------Output look like Ends---------------

#Visit your server’s IP address with :5001 appended to the end in your web browser again

http://your_server_ip:5001

#You should see your application’s output in browser

#now exit from 

(your-env-name) deactivate

####################################################
############# System File For Service ##############
####################################################

sudo nano /etc/systemd/system/<your_service_name>.service

--------------File Code Starts-------------
[Unit]
Description=Gunicorn instance to serve Trackio API
After=network.target
[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/<your-directory-name>
Environment="PATH=/var/www/<your-directory-name>/<your-env-name>/bin"
ExecStart=/var/www/<your-directory-name>/<your-env-name>/bin/gunicorn --workers 3 --bind 127.0.0.1:5001 wsgi:app

[Install]
WantedBy=multi-user.target
-------------File Code Ends-----------

#make sure to setup the  user and group correctly.
User=www-data #your user
Group=www-data #your group

#With that, your systemd service file is complete. Save and close it now.
sudo systemctl start <your_service_name>
sudo systemctl enable <your_service_name>

sudo systemctl status <your_service_name>

####################################################
##################### Nginx File ###################
####################################################
#Configuring Nginx to Proxy Requests

sudo nano /etc/nginx/available-sites/<Your-project-name>

----------------File Code Starts----
server {
    listen 80;

    server_name Your.IP.Goes.Here;

    location / {
        proxy_pass http://127.0.0.1:5011;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_redirect off;
    }

 }
----------------File Code Ends----


sudo ln -s /etc/nginx/sites-available/<Your-project-name> /etc/nginx/sites-enabled

#With the file in that directory, you can test for syntax errors:

sudo nginx -t

sudo systemctl restart nginx

sudo ufw delete allow 5001
sudo ufw allow 'Nginx Full'


#Note: You will receive an HTTP 502 gateway error if Nginx cannot access gunicorn’s socket file. Usually this is because the user’s home directory does not allow other users to access files inside it.

sudo chmod 755 /var/www


If you encounter any errors, trying checking the following:

    sudo less /var/log/nginx/error.log: checks the Nginx error logs.
    sudo less /var/log/nginx/access.log: checks the Nginx access logs.
    sudo journalctl -u nginx: checks the Nginx process logs.
    sudo journalctl -u myproject: checks your Flask app’s Gunicorn logs.


########################## Securing the Application ########################
sudo apt install python3-certbot-nginx

sudo certbot --nginx -d <IP>_domain name -d www.<IP>_domain name



sudo ufw delete allow 'Nginx HTTP'

<h1>Flask Application Deployment on Ubuntu</h1>

<p>Please read info.txt file</p>

<h2>Installing the Components from the Ubuntu Repositories</h2>
<pre>sudo apt update</pre>
<pre>sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools</pre>

<h2>Creating a Python Virtual Environment</h2>
<pre>
sudo apt install python3-venv

mkdir /var/www/your-directory-name
cd /var/www/your-directory-name

python3 -m venv your-env-name
source your-env-name/bin/activate</pre>

<h2>Setting Up a Flask Application</h2>
<pre>(your-env-name) pip install wheel</pre>

<p>Next, install Flask and Gunicorn:</p>
<pre>pip install gunicorn flask</pre>

<p>Create Sample File i.e app.py</p>
<pre>(your-env-name) nano /var/www/<your-directory-name>/<your-file-name.py></pre>

<pre>from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "< h1 style='color:blue'>Hello There!< /h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port='5001')</pre>
<p>You should have a UFW firewall enabled. To test the application, you need to allow access to port.</p>
<p>Your port number if you have define different in .py file use that</p>
<pre>(your-env-name) sudo ufw allow 5001</pre>
<p>Rurn your applocation. #app.py</p>
<pre>(your-env-name) python3 <your-file-name.py></pre>
<p>Output will look like this</p>
<pre>Output
* Serving Flask app "myproject" (lazy loading)
 * Environment: production
   WARNING: Do not use the development server in a production environment.
   Use a production WSGI server instead.
 * Debug mode: off
 * Running on http://0.0.0.0:5001/ (Press CTRL+C to quit)</pre>

 <p>Visit your server’s IP address followed by port number</p>
 <pre>http://your_server_ip:5001</pre>
<h2>Now Creating the WSGI Entry Point</h2>
<pre>(your-env-name) nano /var/www/<your-directory-name>/wsgi.py</pre>
<pre>from app import app

if __name__ == "__main__":
    app.run()
------------File Code Ends----------</pre>

<h2>Configuring Gunicorn</h2>
<pre>(your-env-name) cd /var/www/<your-directory-name>/</pre>
<p>Make sure you are using correct port number.</p>
<pre>(your-env-name) gunicorn --bind 0.0.0.0:5001 wsgi:app</pre>
<p>Output look like.</p>
<pre>
Output
[2020-05-20 14:13:00 +0000] [46419] [INFO] Starting gunicorn 20.0.4
[2020-05-20 14:13:00 +0000] [46419] [INFO] Listening at: http://0.0.0.0:5001 (46419)
[2020-05-20 14:13:00 +0000] [46419] [INFO] Using worker: sync
[2020-05-20 14:13:00 +0000] [46421] [INFO] Booting worker with pid: 46421
</pre>
<p>Visit your server’s IP address with :5001 appended to the end in your web browser again</p>
<pre>http://your_server_ip:5001</pre>
<p>You should see your application’s output in browser</p>
<p>Now exit from</p>
<pre>(your-env-name) deactivate</pre>
<h2>System File For Service</h2>
<pre>sudo nano /etc/systemd/system/<your_service_name>.service</pre>
<pre>[Unit]
Description=Gunicorn instance to serve Trackio API
After=network.target
[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/<your-directory-name>
Environment="PATH=/var/www/<your-directory-name>/<your-env-name>/bin"
ExecStart=/var/www/<your-directory-name>/<your-env-name>/bin/gunicorn --workers 3 --bind 127.0.0.1:5001 wsgi:app

[Install]
WantedBy=multi-user.target</pre>
<p>#make sure to setup the  user and group correctly.
User=www-data #your user
Group=www-data #your group</p>
<p>With that, your systemd service file is complete. Save and close it now.</p>
<pre>sudo systemctl start <your_service_name>
sudo systemctl enable <your_service_name>

sudo systemctl status <your_service_name></pre>
<h2>Nginx File</h2>
<p>Configuring Nginx to Proxy Requests</p>
<pre>sudo nano /etc/nginx/available-sites/<Your-project-name></pre>
<pre>server {
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

 }</pre>

<pre>sudo ln -s /etc/nginx/sites-available/<Your-project-name> /etc/nginx/sites-enabled</pre>
<p>With the file in that directory, you can test for syntax errors</p>
<pre>sudo nginx -t

sudo systemctl restart nginx

sudo ufw delete allow 5001
sudo ufw allow 'Nginx Full'</pre>
in process...



<br />

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

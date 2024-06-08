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

Next, install Flask and Gunicorn:
<pre>pip install gunicorn flask</pre>

Create Sample File i.e app.py
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

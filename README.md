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

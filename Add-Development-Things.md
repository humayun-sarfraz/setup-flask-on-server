To ensure Flask does not cache templates and changes are always reflected immediately, you can configure Flask to run in development mode. Additionally, you can explicitly disable caching in the Jinja2 template engine used by Flask. Here’s how you can do it:

1. Configure Flask for Development
Set Flask to run in development mode which inherently disables caching. You can do this by setting the FLASK_ENV environment variable to development and enabling debug mode in your application configuration.

2. Explicitly Disable Jinja2 Caching
Modify the Flask app configuration to explicitly disable Jinja2 template caching.

Example Flask Configuration
Here’s how you can set this up in your Flask application:

<h3>1. Update Flask App Configuration</h3>
Modify your Flask application to include these configurations:

<pre>
from flask import Flask, render_template

app = Flask(__name__)

# Set environment to development and enable debug mode
app.config['ENV'] = 'development'
app.config['DEBUG'] = True

# Disable Jinja2 template caching
app.config['TEMPLATES_AUTO_RELOAD'] = True

@app.route('/')
def home():
    return render_template('home.html')

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=5011)    
</pre>

<h3>2. Set Environment Variables</h3>
Set the environment variables in your terminal before starting the Flask application. This ensures the application runs in development mode:

<pre>
export FLASK_ENV=development
export FLASK_DEBUG=1   
</pre>

Alternatively, you can include these environment variables in your service file if you are using systemd to manage the Flask application.

<h3>3. Update Service File</h3>
If you are using a systemd service file to manage your Flask application, ensure it includes these environment variables:

<pre>
[Unit]
Description=Gunicorn instance to serve Flask Application
After=network.target

[Service]
User=www-data
Group=www-data
WorkingDirectory=/var/www/trackio/api
Environment="PATH=/var/www/trackio/api/env/bin"
Environment="FLASK_ENV=development"
Environment="FLASK_DEBUG=1"
ExecStart=/var/www/trackio/api/env/bin/gunicorn --workers 3 --bind 127.0.0.1:5011 wsgi:app

[Install]
WantedBy=multi-user.target    
</pre>

<h3>4. Reload Systemd and Restart Services</h3>
After updating the service file, reload the systemd daemon and restart the service:

<pre>
sudo systemctl daemon-reload
sudo systemctl restart trackio_api.service
sudo systemctl restart nginx    
</pre>

<h3>Summary of Steps</h3>
Update Flask Configuration: Ensure Flask is set to development mode and template caching is disabled.
Set Environment Variables: Set FLASK_ENV and FLASK_DEBUG to enable development mode.
Update and Reload Systemd Service: Ensure the service file includes these environment variables, reload systemd, and restart the services.
By following these steps, Flask will not cache templates, and any changes to your HTML files should be reflected immediately.

<hr />

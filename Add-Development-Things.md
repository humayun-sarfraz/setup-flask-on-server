<h1>Add Development Things</h1>


Example Flask Configuration
Here’s how you can set this up in your Flask application:

1. Update Flask App Configuration
Modify your Flask application to include these configurations:

python
Copy code
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
2. Set Environment Variables
Set the environment variables in your terminal before starting the Flask application. This ensures the application runs in development mode:

sh
Copy code
export FLASK_ENV=development
export FLASK_DEBUG=1
Alternatively, you can include these environment variables in your service file if you are using systemd to manage the Flask application.

3. Update Service File
If you are using a systemd service file to manage your Flask application, ensure it includes these environment variables:

ini
Copy code
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
4. Reload Systemd and Restart Services
After updating the service file, reload the systemd daemon and restart the service:

sh
Copy code
sudo systemctl daemon-reload
sudo systemctl restart trackio_api.service
sudo systemctl restart nginx
Summary of Steps
Update Flask Configuration: Ensure Flask is set to development mode and template caching is disabled.
Set Environment Variables: Set FLASK_ENV and FLASK_DEBUG to enable development mode.
Update and Reload Systemd Service: Ensure the service file includes these environment variables, reload systemd, and restart the services.
By following these steps, Flask will not cache templates, and any changes to your HTML files should be reflected immediately

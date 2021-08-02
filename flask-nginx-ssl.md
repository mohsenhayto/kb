# Serve Flask Applications with uWSGI and Nginx on Ubuntu 20.04

## Step 1 — Installing the Components
```
sudo apt update
sudo apt install python3-pip python3-dev build-essential libssl-dev libffi-dev python3-setuptools
```
## Step 2 — Creating a Python Virtual Environment
```
sudo apt install python3-venv
mkdir ~/myproject && cd ~/myproject
python3 -m venv myprojectenv
source myprojectenv/bin/activate
```
## Step 3 — Setting Up a Flask Application
```
pip install wheel
pip install uwsgi flask
nano ~/myproject/myproject.py
```
* Add sample code
```
from flask import Flask
app = Flask(__name__)

@app.route("/")
def hello():
    return "<h1 style='color:blue'>Hello There!</h1>"

if __name__ == "__main__":
    app.run(host='0.0.0.0')
```
* Run Sample code
```
sudo ufw allow 5000
python myproject.py
```
* Open http://your_server_ip:5000
## Step 4 — Configuring uWSGI
```
nano ~/myproject/wsgi.py
uwsgi --socket 0.0.0.0:5000 --protocol=http -w wsgi:app
```
* Open http://your_server_ip:5000
```
deactivate
nano ~/myproject/myproject.ini
```
* Add this config to myproject.ini
```
[uwsgi]
module = wsgi:app

master = true
processes = 5

socket = myproject.sock
chmod-socket = 660
vacuum = true

die-on-term = true
```
## Step 5 — Creating a systemd Unit File
```
sudo nano /etc/systemd/system/myproject.service
```
* Add this config to myproject.service
```
[Unit]
Description=uWSGI instance to serve myproject
After=network.target

[Service]
User=la
Group=www-data
WorkingDirectory=/home/<user>/myproject
Environment="PATH=/home/<user>/myproject/myprojectenv/bin"
ExecStart=/home/<user>/myproject/myprojectenv/bin/uwsgi --ini myproject.ini

[Install]
WantedBy=multi-user.target
```
* Start Service
```
sudo systemctl start myproject
sudo systemctl enable myproject
sudo systemctl status myproject
```

## Step 6 — Configuring Nginx to Proxy Requests
```
sudo nano /etc/nginx/sites-available/myproject
```
* Add this Config to myproject
```
server {
    listen 80;
    server_name your_domain www.your_domain;

    location / {
        include uwsgi_params;
        uwsgi_pass unix:/home/<user>/myproject/myproject.sock;
    }
}
```
* Enable site
```
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
sudo unlink /etc/nginx/sites-enabled/default
sudo nginx -t
sudo systemctl restart nginx
sudo ufw delete allow 5000
sudo ufw allow 'Nginx Full'
```
* Open `http://your_domain`

## Step 7 — Securing the Application
```
sudo apt install certbot python3-certbot-nginx
sudo certbot --nginx -d your_domain -d www.your_domain
sudo ufw delete allow 'Nginx HTTP'
```
* Open `https://your_domain`

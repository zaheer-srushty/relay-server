# Push Notification Relay Server for Frappe Apps
This repo was created to enable push notifications for Frappe Apps such as Raven.

## Getting Started
To be able to run this application, you will need to do the following:

1. Clone this project and create a virtual environment `python -m venv env`
2. Install all requirements `pip install -r requirements.txt`
4. Create a Firebase Project & get Service Account credentials [Link](https://sharma-vikashkr.medium.com/firebase-how-to-setup-a-firebase-service-account-836a70bb6646)
5. Run this command on the shell
  ``` bash
  export GOOGLE_APPLICATION_CREDENTIALS="/home/frappe/relay-server/{service-account-file_name}.json"

  ```
6. Follow **Register you app** under Step 1 given in the [Firebase documentation](https://firebase.google.com/docs/web/setup#register-app) and obtain the `FIREBASE_CONFIG` JSON object. Save it to `my_secrets.py`.
7.  Follow this StackOverflow [Link](https://stackoverflow.com/a/54996207) to generate a VAPID key. Save it to `my_secrets.py`
8.  Generate `API_KEY` & `API_SECRET` and add the `API_SECRET` value to the `my_secrets.py`.
9.  Finally, your `my_secrets.py` should like this
``` python
API_SECRET = 'tIUAGguQH-xajbkcjsd-lsd'
VAPID_PUBLIC_KEY = "Bxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
FIREBASE_CONFIG = {
  "apiKey": "AIzaSyC3UVxbCkUv3l4PpyWkQZGEuwOds76sdUgk0",
  "authDomain": "xxxxxxxx-frappe.firebaseapp.com",
  "projectId": "xxxxxxxxx-frappe",
  "storageBucket": "xxxxxxxxx-frappe.appspot.com",
  "messagingSenderId": "815115xxx703",
  "appId": "1:815115xxx703:web:e89fcdadfcf8df09e4852",
  "measurementId": "G-XXXXXXXXXG"
}
```
10. Change the `USER_DEVICE_MAP` key values to `{project_name}_{site_name}` where `project_name` could be `raven` or `hrms` & `site_name` as your site name while setting up the Frappe bench.
11. Run the application
12. Add the `API_SECRET` & `API_KEY` in ERPNext Push Notification settings and then enable the Push Notification Relay option.

## Running the Application
``` bash
gunicorn app:app -c gunicorn.conf.py
```
## Run a Systemd Service
- Create a Systemd service file at `/etc/systemd/system/push-relay.service` and copy the following and replace the paths/filenames accordingly :
``` bash
# /etc/systemd/system/push-relay.service
[Unit]
Description=Gunicorn instance to server Frappe Push Notification Relay Server
After=network.target

[Service]
User=frappe
Group=www-data
WorkingDirectory=/home/frappe/relay-server
Environment="PATH=/home/frappe/relay-server/env/bin"
Environment="GOOGLE_APPLICATION_CREDENTIALS=/home/frappe/relay-server/{service-account-file_name}.json"
ExecStart=/home/frappe/relay-server/env/bin/gunicorn app:app -c /home/frappe/relay-server/gunicorn.conf.py

[Install]
WantedBy=multi-user.target
```
- Run the following commands
  ``` bash
  sudo systemctl daemon-reload
  sudo systemctl enable push-relay
  sudo systemctl start push-relay
  ```




## fix CORS issue [ localhost ] :

  - Run the following commands
    ``` bash
     pip install flask-cors
    ```

     
  - app.py                                          [add line 8 & 10]
    ``` bash
      7    from my_secrets import API_SECRET, FIREBASE_CONFIG, VAPID_PUBLIC_KEY, BADGE_ICON
    + 8    from flask_cors import CORS
    
      9    app = Flask(__name__)
    + 10   CORS(app, resources={r"/*": {"origins": "*"}})  # This will allow all origins
    
      11   firebase_app = firebase_admin.initialize_app()
      12   basic_auth = HTTPBasicAuth()
    ```
  
  - firebase_admin/_messaging_encoder.py            [ comment line : 507, 508 ]
      ``` bash
        506    link = result.get('link')
     - 507    # if link is not None and not link.startswith('https://'):
     - 508        # raise ValueError('WebpushFCMOptio ns.link must be a HTTPS URL.') 
        509    return result
      ```
  
  - Restart bench and gunicorn server

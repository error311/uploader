# File Uploader

A simple file uploader application that allows authenticated users to upload, list, and delete files. 
The application uses PHP and a Python authentication service running on Apache2.

## Prerequisites

- Apache2, configured, up and running
- PHP 8.1 or higher
- Python 3
- Required PHP extensions: `php-json`, `php-curl`

Hint:
```
sudo apt update
sudo apt install apache2
sudo apt install php libapache2-mod-php
sudo apt install python3
```

## Installation

For simplicity, I'll use my current Ubuntu instance user name, you should replace by yours.

  ### Clone 
   ```
   git clone https://github.com/yourusername/uploader.git
   cd uploader
   ```
   ### or download this repository
   ```
   wget https://github.com/sensboston/uploader/archive/refs/heads/master.zip
   unzip master.zip -d uploader
   mv uploader/uploader-master/* uploader/
   rm -r uploader/uploader-master
   rm master.zip
   ```
  ### Install Python prerequisites

  ```
  pip install flask pam
  ```

  ### Create Python authentication service
  (note: **port 7000** is used; if you need to change port number, make needful changes in the **app.py** and php scripts - search for '7000')  
  
  ```
  sudo nano /etc/systemd/system/flaskapp.service
  ```

  Add the following content to this file (but replace **User**, **WorkingDirectory** and **ExecStart**):

  ```
  [Unit]
  Description=Flask Application
  After=network.target

  [Service]
  User=ubuntu
  WorkingDirectory=/home/ubuntu/uploader
  ExecStart=/usr/bin/python3 /home/ubuntu/uploader/app.py
  Restart=always

  [Install]
  WantedBy=multi-user.target
  ```

  ### Enable and start the service:
  ```
  sudo systemctl daemon-reload
  sudo systemctl enable flaskapp
  sudo systemctl start flaskapp
  sudo systemctl status flaskapp.service
  ```

  ### Configure PHP

  Ensure the following PHP settings are in your **/etc/php/8.1/apache2/php.ini**:
  ```
  log_errors = On
  error_log = /var/log/php_errors.log
  ```
  
  Also check for max upload file/post size limits in **/etc/php/8.1/apache2/php.ini** (adjust to your needs, like 10G):
  ```
  upload_max_filesize = 10M
  post_max_size = 10M
  ```

  ### Create the upload directory and set the necessary permissions:

  ```
  sudo mkdir -p /var/www/html/upload
  sudo chown -R www-data:www-data /var/www/html/upload
  sudo chmod -R 755 /var/www/html/upload
  ```

  ### Create a new  user for uploading files
  (please note, I don't recommend you to use your actual ssh-enabled user account):

  ```
  sudo useradd -m -d /var/www/html/upload -s /bin/bash uploader
  sudo passwd uploader
  sudo chown -R uploader:www-data /var/www/html/upload
  sudo chmod -R 755 /var/www/html/upload
  ```

  ### Create application directory at webroot (or configure app/site):
  (note: with my Apache configuration, I just need to create a subdirectory)
  ```
  sudo mkdir -p /var/www/html/uploader
  ```

  ### Edit file config.php and adjust variables
  ```
  sudo nano /home/ubuntu/uploader/config.php
  ```

  ### Copy all files to the folder created above:
  ```
  cd /home/ubuntu/uploader/
  sudo cp -r * /var/www/html/uploader
  ```

  ### Restart Apache to apply changes:

  ```
  sudo systemctl restart apache2
  ```

## Usage
Open your web browser and navigate to https://yourserveraddress/uploader

Enter your username and password to authenticate.

Choose a file to upload and click the "Upload" button.

The uploaded files will be listed on the page, and you can delete them using the "Delete" button.

![screenshot](https://github.com/sensboston/uploader/assets/1036158/5428672d-7dcc-4d7a-a96f-dfe578618c75)

## Issues / TODO

There are two unresoved (yet) issues with the app: 
 - First, I suppose to use Python app, running as service, for the user authentication in the PHP scripts because I can't make the PAM PHP extension works 😒 Google's searches returned tonns of useless and non-working advises and suggestions; even ChatGPT can't resolve that issue. It looks like this module is deprecated but I can't find any working replacement.
By the way, if I'll get PAM module working in PHP, Python app and auth.php will be replaced by very simple call:
    ``` if (pam_auth($username, $password))``` 

    BTW, I found a [third-party implementation](https://github.com/amishmm/php-pam) but haven't tried yet.

 - Secondly, a very strange error occurs: if I use the authenticate function from the **auth.php** module in file_list.php (adding ```require_once 'auth.php';```), then the file list is not appearing on web page. Although the user is authenticated and **file_list.php** sends a list of files in JSON format. If I define this function (using copy&paste) in the **file_list.php** module itself, then everything works great.
Perhaps this has something to do with PHP session - I'm not a big expert in PHP and web programming. If anyone can help solve this problem, I would be very grateful - I hate strange errors! 

# gunicorn
Deploying Django Applicatin on to production server using Nginx and Gunicorn

In this post, we will see how to use nginx with gunicorn to serve django applications in production. 

steps to be followed:

step1: Installing Python and nginx
       $ sudo apt update
       $ sudo apt install python3-pip python3-dev nginx

step2: Creating Virtual Environment
       $ sudo pip3 install virtualenv                 //Installing Virtual Env package
       $ mkdir ~/projectdir                           //create a project directory to host our Django application
       $ cd ~/projectdir                              
       $ virtualenv env                               //create a virtual environment inside that directory
       $ source env/bin/activate                      //activate the virtual environment

step3: Installing Django and Gunicorn
       $ sudo apt update
       $ pip install django gunicorn

step4: Setting up an Django project / import the created one to the directory
       $ django-admin startproject <project_name> 

step5: Add your IP address or domain to the ALLOWED_HOSTS variable in settings.py

step6: Allow the port over firewall
       $ sudo ufw allow 8000

step7: Validate Django Development server 
       $ ~/projectdir/manage.py runserver 0.0.0.0:8000
      
step8: Configuring gunicorn
       $ gunicorn --bind 0.0.0.0:8000 textutils.wsgi         //checking gunicorn's ability to serve application
       $ deactivate                                          //Deactivate the virtualenvironment
       $ sudo vim /etc/systemd/system/gunicorn.socket        //create a system socket file for gunicorn 
         add below lines into it
          [Unit]
          Description=gunicorn socket

          [Socket]
          ListenStream=/run/gunicorn.sock

          [Install]
          WantedBy=sockets.target
          
       $ sudo vim /etc/systemd/system/gunicorn.service              //create a service file for gunicorn
         add below lines to it
          
          [Unit]
          Description=gunicorn daemon
          Requires=gunicorn.socket
          After=network.target

          [Service]
          User=vm2
          Group=www-data
          WorkingDirectory=/home/vm2/project/mysite/Django
          ExecStart=/home/vm2/project/myenv/bin/gunicorn \
                    --access-logfile - \
                    --workers 3 \
                    --bind unix:/run/gunicorn.sock \
                    mysite.wsgi:application
          Restart=always
          RestartSec=3

          [Install]
          WantedBy=multi-user.target
          
      NOTE: 1. Replace User with your system user name
            2. Replace mysite.wsgi with your Django project name
      
step9: start and enable the gunicorn socket
       $ sudo systemctl start gunicorn.socket
       $ sudo systemctl enable gunicorn.socket

step10: Configuring Nginx as a reverse proxy
        Create a configuration file for Nginx
        $sudo nano /etc/nginx/sites-available/mysite
        Paste the below contents inside the file created
           
           server {
              listen 80;
              server_name <ip_addr / domain name >;

              location = /favicon.ico { access_log off; log_not_found off; }
              location /static/ {
                  root /home/path to your/projectdir;
              }

              location / {
                  include proxy_params;
                  proxy_pass http://unix:/run/gunicorn.sock;
              }
            }
 
step11: Activate the configuration 
         $ sudo ln -s /etc/nginx/sites-available/mysite /etc/nginx/sites-enabled/

step12: Delete the default config files 
        $ cd /etc/nginx/sites-enabled
        $ sudo rm default

step13: Restart nginx service
        $ sudo systemctl restart nginx

step14: Validate
        http://<ip_addr>
        Your Django website should now work fine!
         
         
         
         

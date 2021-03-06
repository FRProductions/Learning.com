setup of Django app on nginx / uwsgi
------------------------------------

tutorial / guide: http://uwsgi.readthedocs.io/en/latest/tutorials/Django_and_nginx.html

target Python version:
Python 2.7.6

pip list should include:
Django (1.9)
django-filter (0.13.0)
djangorestframework (3.3.3)
Markdown (2.6.6)
uWSGI (2.0.13)

Setup new DNS entry on GoDaddy.com to map learningdotcom.fraboom.com to server IP address [BD's note: use the load balancer's IP address]

Copy and configure 3 files to deployment directory, from /var/git/learningdotcom:
- learningdotcom-REST-nginx.conf
- learningdotcom-REST-uwsgi.ini
- uwsgi_params

Create nginx symlink in /etc/nginx/sites-available

    ln -s /var/git/learningdotcom/learningdotcom-REST-nginx.conf learningdotcom.fraboom.com

Create nginx symlink in /etc/nginx/sites-enabled

    ln -s /etc/nginx/sites-available/learningdotcom.fraboom.com learningdotcom.fraboom.com

Make sure Django directory has proper ownership / permissions:
- all files have group = www-data
- directory is group writable so file socket can be created

Add uwsgi vassal symlink in /etc/uwsgi/vassals/

    ln -s /var/git/learningdotcom/learningdotcom-REST-uwsgi.ini learningdotcom-REST-uwsgi.ini

IMPORTANT: make sure upstream name (i.e. "django_learningdotcom") matches the "uwsgi_pass" setting

Useful commands
---------------

    nginx
      ps -Al | grep nginx         # view running nginx processes
      sudo service nginx restart  # WARNING: have Fraboom SSL password handy for quickly restarting the server!
                                  # NOTE: this password is called "fraboom ssl private key" in Dashlane
      sudo fuser -k 80/tcp        # kill stubborn nginx!

      view log file:
          sudo tail -f /var/log/nginx/error.log
          sudo tail -f /var/log/nginx/learningdotcom.fraboom.com.error.log

    UWSGI
      ps -Al | grep uwsgi         # view running uwsgi processes
      sudo service uwsgi stop
      sudo service uwsgi start

      view log file:
          sudo tail /var/log/uwsgi.log

      view uwsgi upstart config:
          /etc/init/uwsgi.conf

Update Database
---------------

    # go to learningdotcom directory
    /var/git/learningdotcom

    # get latest
    sudo git fetch
    sudo git checkout <commit-hash>

    # apply any db migrations
    cd REST
    sudo python manage.py migrate

    # restart uwsgi
    sudo service uwsgi stop
    sudo service uwsgi start

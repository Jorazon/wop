# Metropolia Educloud virtual computer (CentOS server)

## Working from outside metropolia network

1. Configure [Metropolia VPN](https://wiki.metropolia.fi/display/itservices/Install+and+Use+VPN+Utility+Program+Installation+on+Your+Own+Computer)
2. Other options (e.g. during zoom session to reduce vpn bandwidth)
   1.  to test your app/webpages, create a [ssh tunnel and configure one of your browsers through it](https://tietohallinto.metropolia.fi/display/itservices/SSH+Tunnelling).
       (use your metropolia username and password)
       ```console
       $ ssh -Nf -D 8888 <your-metropolia-username>@shell.metropolia.fi
       ```
   1.  For the terminal access to your server, use double ssh, first from your local machine to shell.metropolia.fi (with your metropolia username/password):
        ```console
        $ ssh <your-metropolia-username>@shell.metropolia.fi
        ```
       once connected to metropolia shell, you can connect to your server:
       ```console
       $ ssh <your-server-username>@<your-server-IP>
       ```

# LAMP (well, P is optional and is replaced with NodeJS)

## Configure Linux server

1. Order your virtual server
   1.  visit [https://educloud.metropolia.fi/](https://educloud.metropolia.fi/)
   1.  login with your metropolia credentials
   1.  navigate to Services/Catalogs
   1.  choose Centos 7 x64 - Basic Install with IP address setup done
   1.  Lease time 4 months, size doesn't matter much (medium or large will have more cpu/memory)
   1.  ([Helpdesk page](https://tietohallinto.metropolia.fi/display/itservices/Educational+educloud+virtual+services))
2. Wait for 10-15 minutes for the server to be created
   1.  You will receive an email with the IP address of the server and the root password (root is default administrator on any linux/unix operating system)
3. Open a terminal (CLI). On windows computer, install [bash shell](https://docs.microsoft.com/en-us/windows/wsl/install-win10) or [Git Bash](https://gitforwindows.org/). Mac/Linux already have a terminal to start with (e.g. on Mac under Applications/Utilities, on gnome/mate linux desktop with ``ALT + CTRL + T`` shortcut)
4. From terminal, use ssh protocol to connect to your virtual server
   1.  If you are not in metropolia network, use VPN or double ssh (check [first section](#working-from-outside-metropolia-network))
   1.  Run the following command:
        ```console
        $ ssh root@IPaddress
        ```
   1.  substitute IPaddress with the address of your server (you got in the mail), root is the username (keep it as it is). After pressing enter, it will ask for password (copy/paste from the mail). Note, blind typing password (the cursor will not move nor show stars nor nothing while you type); it a security measure: if anyone look over your shoulders, they cannot guess the length of your password.
1.  After login: first security measures, change root password and create yourself a user account: avoid
to do everything as root! If someone crack your root account, s/he can destroy the full server.
If s/he “only” crack your user account, s/he will be sandboxed (and can do less damages).
   1.  Change root password:
        ```console
        # passwd
        ```
   1.  Create user account:
        ```console
        # useradd wantedUsername
        # passwd wantedUsername
        ```
   1.  Give that user sudo (super user do) privileges:
        ```console
        # visudo
        ```
   1.  Navigate to the line that shows:
       ```apacheconf
        ## Allow root to run any commands anywhere
        root ALL=(ALL) ALL
        ```
        To move in the text, use arrow keys or ``hjkl`` keys (e.g. to move 92 lines down, press ``92j``).
   1.  Add a new line and enter in insert mode by typing ``o``-key (notice the INSERT on the last line).
        Type
       ```apacheconf
        wantedUsername ALL=(ALL) ALL
        ```
   1.  To escape from insert mode, type ``esc``-key. Save and quit the editor by typing ``:wq``
   1.  Give your user the permission to ssh to the server by editing ssh deamon configuration:
        ```console
        # vi /etc/ssh/sshd_config
        ```
   1.  Navigate to the end of the file (e.g. with ``G``-key (capital g)) and add the following line:
       ```apacheconf
        AllowUsers wantedUsername
       ```
   1.  Escape, save and quit.
   1.  Restart the ssh deamon (service)
        ```console
        $ sudo systemctl restart sshd
        ```
   1.  Test that you can login with the new account:
        ```console
        # logout
        ```
        (alternative to logout is to use ``CTRL+D`` shortcut).
        ```console
        $ ssh wantedUsername@IPaddress
        ```
   1.  If and only if the login is successful with your new account, disable the root login
        ```console
        $ sudo vi /etc/ssh/sshd_config
        ```
   1.  Search the line ``#PermitRootLogin`` (to search with vi, type ``/``-key, then your search pattern and enter). Once found, remove the #-mark (e.g. use ``x``-key to delete the character under the cursor).
        And replace yes with ``no``.\
        Escape, save and quit.
   1. Restart the ssh deamon again:
        ```console
        $ sudo systemctl restart sshd
        ```
2.  The CentOS package manager (to install/update applications and operating system)
    is YUM. For example, to maintain the operating system with the latest bug and security fixes, run the following command about every weeks:
    ```console
    $ sudo yum update
    ```

## Install and configure Apache web server

1.  Install apache web server:
    ```console
    $ sudo yum install httpd
    ```
    1.  start it:
        ```console
        $ sudo systemctl start httpd
        ```
    2.  make sure that the web server will always start (e.g. if server reboot):
        ```console
        $ sudo systemctl enable httpd
        ```
4. Open the firewall to allow web traffic through http (port 80) and https (port 443):
        ```console
        $ sudo firewall-cmd --permanent --zone=public --add-service=http --add-service=https```
        And reload the firewall
        ```console
        $ sudo firewall-cmd --reload
        ```
5. In your browser, open the URL: ``http://<ip-address>/`` (substitute with your IP address). You should see your Apache web server welcome page.

1. (Optional) The root of your web server is on following path: ``/var/www/html``; but it require root privileges to add/edit/delete files. to create a public_html-folder for your user:
   1.  Follow these steps '[Enable Apache Userdirs](https://www.unixmen.com/linux-basics-enable-apache-userdir-centos-7rhel-7/)'.
        Substitue the "unixmenuser" with your username "wantedUsername".
   1.  visit: ``http://<ip-address>/~<wantedUsername>/``

## Install and configure MariaDB database server

1. Install MariaDB server:
   ```console
   $ sudo yum install mariadb-server
   ```
   1. Start and enable it:
      ```console
      $ sudo systemctl start mariadb
      $ sudo systemctl enable mariadb
      ```
   1. Secure it:
      ```console
      $ mysql_secure_installation
      ```
      Note: database root user is not operating system root user! Avoid same password!
   1. Connect to your database:
      ```console
      $ mysql -u root -p
      ```
      and create a database and a user with privileges on it:
        ```sql
        > CREATE DATABASE catdb;
        > CREATE USER 'dbuser' IDENTIFIED BY 'test123';
        > GRANT USAGE ON *.* TO 'dbuser'@localhost IDENTIFIED BY 'test123';
        > GRANT ALL ON catdb.* TO 'dbuser'@'localhost';
        > FLUSH PRIVILEGES;
        > exit
        ```
        (in case you would need outside access (e.g. during project, separate database server from app server), replace ``localhost`` with ``'%'`` in the two GRANT queries).
   1. Download the ``tables.txt`` SQL script:
   ```console
   $ curl -O https://raw.githubusercontent.com/ilkkamtk/wop-starters/week2-1/tables.txt
   ```
   (or upload with any FTP tool the ``tables.txt`` to your home folder ``/home/wantedUsername``\
       (or use ``scp`` from your local machine, navigate with terminal to the folder and run ``$ scp tables.txt <wantedUsername>@<ip-address>:~``))
   1. Import the tables and insert the data:
      ```console
      $ mysql -u dbuser -p catdb < tables.txt
      ```
   1. Eventually check:
      ```console
      $ mysql -u dbuser -p catdb
      ```
      ```sql
      > SHOW TABLES;
      > SELECT * FROM wop_cat;
      > exit
      ```
1. (Optional) if you would like to install phpMyAdmin to administrate your databases, tables and data with a graphical user interface, check e.g. [here](https://www.digitalocean.com/community/tutorials/how-to-install-and-secure-phpmyadmin-with-apache-on-a-centos-7-server)

## Install and configure NodeJS application server

1.  Install node for Centos: [https://github.com/nodesource/distributions/blob/master/README.md#rpm](https://github.com/nodesource/distributions/blob/master/README.md#rpm)
    use the ``# No root privileges`` version, so something like:
    ```console
    $ curl -fsSL https://rpm.nodesource.com/setup_14.x | sudo bash -
    $ sudo yum install -y nodejs
    ```
1. Configure Apache httpd server as a reverse proxy to node server:
   1.  create/edit an apache configuration file:
        ```console
        $ sudo vi /etc/httpd/conf.d/node.conf
        ```
   1.  add the following content:
       ```apacheconf
       <VirtualHost *:80>
         ProxyPreserveHost On
         ProxyPass /app/ http://127.0.0.1:3000/
         ProxyPassReverse /app/ http//:127.0.0.1:3000/
       </VirtualHost>
       ```
1.   save and restart apache server
        ```console
        $ sudo systemctl restart httpd
        ```
1.  give permission to apache server to visit URL
        ```console
        $ sudo setsebool -P httpd_can_network_connect 1
        ```
2. Install and run your node application:
   1.  make sure you are in your home folder:
        ```console
        $ cd
        ```
   1.  clone your app (choose HTTPS)
        ```console
        $ git clone https://gitlab.metropolia.fi/<your-repo>
        ```
   1.  go to the cloned repo
        ```console
        $ cd <your-repo>
        ```
   1.  eventually, check that you are in the right branch (checkout if
        not)
   1.  install dependencies
        ```console
        $ npm i
        ```
   1. create/edit ``.env`` file with your db credentials (you set in [MariaDB](#install-and-configure-mariadb-database-server), step iii)
       ```apacheconf
        DB_HOST=127.0.0.1
        DB_USER=<your-db-user>
        DB_PASS=<your-db-user_password>
        DB_NAME=<your-db-name>
        ```
   1.  run your application
        ```console
        $ node app.js
        ```
   1.  test, open a browser and visit ``http://<ip-address>/app/``
   1.  if you want to let your app running forever, run it as a
        background task (note the ampstamp & at the end)
        ```console
        $ node app.js &
        ```
   1.  and if you want to stop your background app:
        ```console
        $ pkill node
        ```

## Extra and resources
- [Unix/linux command](https://centoshelp.org/resources/commands/linux-system-commands/)
- [vim commands](https://vim.rtorr.com/) (works with ``vi`` and ``visudo``)
- [autocomplete](https://www.cyberciti.biz/faq/fedora-redhat-scientific-linuxenable-bash-completion/) (make typing command faster using tab key)
- [sudo autocomplete](http://www.webupd8.org/2010/03/how-to-autocomplete-commands-preceded.html)  (so tab key works when you type ``sudo ...``)
- [installing extra packages](https://wiki.centos.org/AdditionalResources/Repositories)

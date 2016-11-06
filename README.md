# Multisite vagrant box based on scotch/box 
Based on scotch/box. Creates hosts and folders for defined domain names in Vagrantfile

DOMAINS=("symfony.local" "test.local")

##### Creates these folders

+ symfony.local / web
+ test.local / web

##### Creates these hosts
- http://somfony.local
- https://somfony.local
- http://m.symfony.local
- https://m.symfony.local
+ http://test.local
+ https://test.local
+ http://m.test.local
+ https://m.test.local

## PHPSTRORM XDEBUG CONFIG

shift-ctrl-a  and type 'Edit Config' and select 'Edit Configurations...'

Click the green + sign to add a new 'PHP Web Application' configuration and set:
Name: servername
Server: click the '...' and add a new server

To add a new Server - click the green + and set:
Name: servername
Host: 192.168.33.10
Port: 80
Debugger: Xdebug

In the File mapping list, click the first entry under Project files, and set
the 'Absolute path on server' to be /var/www


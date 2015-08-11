Liferay | Load balancer | Apache
=========

An Ansible role to install and configure Apache HTTPD as AJP load balancer to group of application servers.
This role was created to make it easy to set up load balancer for Liferay but can be used for any other servers supporting AJP protocol.   

Requirements & Limitations
------------

This role was tested only on Ansible 1.9.2 but may also work with older versions!
 
This is a very basic role and while it is capable of provisioning fully functional Apache HTTPD load balancer, 
it has some limitations (comparing to what one can do configuring Apache manually)!

 - Tested only on Debian Linux
 - While the list of modules to installed is configurable, it only supports AJP for the balancer.  
 - The configuration is provided as virtual host (by default bound to *:80)  
 - By default it will disable the default virtual host "000-default"
 - The list of servers to proxy requests to must be defined as Ansible group in the inventory  
 

Role Variables
--------------

```yaml
#
# The host and port to bind the virtual host configuration to
#
balancer_host: "*"
balancer_port: 80

#
# Virtual host name. By default empty so it matches any virtual host
#
#balancer_server_name: 

#
# List of visrtual host aliases
#
#balancer_server_aliases: []
 

#
# Ansible hosts group containing the application servers to proxy to.
# In this group additional per host properties can be provided:
# - ajp_port - the port on which AJP connection should be made (default is 8009) 
# - loadfactor - the load balancing factor (default is 1) 
# Example:
# 192.168.121.12 ajp_port=8010 loadfactor=2
#
balancer_app_server_group: liferay

#
# Load balancing related apache modules to be loaded.
#
balancer_apache_modules: 
- proxy
- proxy_ajp
- proxy_http
- proxy_balancer
- lbmethod_bybusyness
- lbmethod_byrequests
- lbmethod_bytraffic
- lbmethod_heartbeat

#
# Load balancing method.
#
balancer_method: byrequests
#balancer_method: bytraffic
#balancer_method: bybusyness
#balancer_method: heartbeat

#
# The name of the balancer virtual host configuration file
#
balancer_config_file_name: balancer

#
# The name of the default virtual host that comes with apache
#
balancer_apache_default_vhost: 000-default
```

Dependencies
------------

None

Example Playbook
----------------

The simplest form:

    - hosts: servers
      roles:
      - role: milendyankov.liferay-balancer-apache

will 
 
 * make sure Apache 2 is installed
 * enable the following Apache modules
   - proxy
   - proxy_ajp
   - proxy_http
   - proxy_balancer
   - lbmethod_bybusyness
   - lbmethod_byrequests
   - lbmethod_bytraffic
   - lbmethod_heartbeat      
 * create a virtual host configuration in `/etc/apache2/sites-enabled/balancer.conf` which
   - is bound to `*:80` 
   - does not have `ServerName` and `ServerAlias` set (will match any host)
   - balances requests to all servers in `liferay` inventory group 
   - has `lbmethod` set to `byrequests`
 * enable `balancer.conf` and disable `000-default.conf`
 
Here is an example of somewhat more advanced configuration 
 
    - hosts: servers
      roles:
      - role: milendyankov.liferay-balancer-apache
        balancer_config_file_name: tomcat_balancer    # save vhost config in `tomcat_balancer.conf` file
        balancer_app_server_group: tomcat_servers     # proxy requests to servers in `tomcat_servers` inventory group
        balancer_method: bytraffic                    # set balancer's `lbmethod` to `bytraffic`
        balancer_host: 192.168.121.121                # bound this virtual host to specific IP address
        balancer_server_name: mydomain.com            # match only requests for `mydomain.com`
        balancer_server_aliases:
        - www.mydomain.com
        - mail.mydomain.com
        - shop.mydomain.com

License
-------

BSD


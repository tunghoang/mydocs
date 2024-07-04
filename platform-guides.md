# Overview
This platform is built for the following courses:  
- INT3306: Web Application Development
- INT3305: Multimedia Communications

# Available Services
The platform provide the following features/services:
- Linux environment
- Built-in frameworks: NodeJS (v16), Python (v3.9.7)
- Databases: MySQL, PostgreSQL
- Utilities: Git, PHPMyAdmin, vim, tmux
- Service Proxy for exposing your services

# How to access
## Accessing The Linux Environment 
Visit [https://int3306.freeddns.org](https://int3306.freeddns.org). 
Use your `$YOUR_ACCOUNT` to login. If you do not have one, contact your professor/instructor to request an account.
This platform use *First Login Authentication*. Password that you use for the first time you login will be remembered for your next logins
## Accessing MySQL Database
After login to the Linux Environment, you can access *MySQL* from the Linux Environment. MySQL database hostname is defined in environment variables `$MYSQL_SERVICE_HOST` and `$MYSQL_SERVICE_PORT`. Your credential for accessing MySQL is `$YOUR_ACCOUNT`. The default password is `qwertyuiop`.

Affter accessing your database, you are recommended to change your password. Use the following statement at *mysql* prompt:
```
SET PASSWORD = PASSWORD("SOME_PASSWORD")
```
***
Notes: You can only create database whose name start with `$YOUR_ACCOUNT_`. For example, if `YOUR_ACCOUNT="group1"` then a valid database name for you is `group1_somedb`
***
## Accessing PHPMyAdmin
You can also use PHPMyAdmin at [https://pma.int3306.freeddns.org](https://pma.int3306.freeddns.org). 

# Expose your service
You can expose a port so that your web application can be accessible by a client from the internet. Apply the following command at shell prompt of the Linux Environment.

```
/etc/jupyter/bin/expose <port>
```
Once the above command is run, your service can be accessible at [https://$YOUR_ACCOUNT.int3306.freeddns.org](https://$YOUR_ACCOUNT.int3306.freeddns.org)
***
NOTES: One account can expose only one port.
***

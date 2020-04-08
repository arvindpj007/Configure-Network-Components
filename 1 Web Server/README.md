# Web Server
These tasks requires 3 VM hosts: `lab1` (10.0.2.8), `lab2` (10.0.2.9) and `lab3` (10.0.2.10). 

> [Lynx](https://lynx.browser.org/ "Lynx Reference") is a text browser for the World Wide Web which is used for these tasks. Lynx can be install using the command `sudo apt-get install lynx`

## Apache2

- This task requires lab2 to install Apache2 server and serve the Apache default webpage using lynx. Further the using the SSH port forwarding the Apache default webpage should be served in a browser in the host system. The required steps are enumerated below:

>[Apache](https://help.ubuntu.com/lts/serverguide/httpd.html "Apache Web Server Reference") is the most commonly used Web server on Linux systems and are used to serve Web pages requested by client computers. 

1. Installing Apache2 in lab2: 

        sudo apt-get update && sudo apt-get upgrade -y
        sudo apt-get install apache2

2. Serving a default web page using Apache2:

        lynx localhost

    ![Default Webpage](./images/1_default_webpage.png)

    The default web page is served using the html file `/var/www/html/index.html`. The default page configuration is specified in the `000-default.conf` file present in the location `/etc/apache2/sites-available`.

3. SSH port forwarding to load web page local web browser: 

    This can be done using the `ssh` command, the mapping of the ports can be done using the `L` flag as shown below:

        ssh -L localhost:1234:10.0.2.9:80 -p 10011 lab2@localhost
    
    Port number `80` (the HTTP port) from the VM lab 2 is forwarded to port number `1234` of the host system. This web page can now loaded in browser with the URL `http://localhost:1234/` as shown below: 

    ![Portforward Webpage](./images/2_portforward_webpage.png)

## Serving Webpage Using Node.js

- This task requires to install nodejs on lab3 from package manager and create a HTTP application helloworld.js listening on port 8080 that serves a web page
with a text "Hello world!". The steps required are enumerated below:

>[Node.js](https://nodejs.org/en/about/ "Node.js reference") is an asynchronous event-driven JavaScript runtime, designed to build scalable network applications.

1. On `lab3`, nodejs is installed as follows:

        sudo apt-get update && sudo apt-get upgrade -y
        sudo apt-get install nodejs

2. To serve a web page with text "Hello World!", a HTTP application `helloworld.js` is created that is listening on the `port 8080`.

    The javascript code `helloworld.js` is as follows:

    ```javascript
    //helloworld.js

    var http = require('http');

    http.createServer(function (req, res) {
        res.writeHead(200, {'Content-Type': 'text/plain'});
        res.end('Hello World!');
    }).listen(8080);
    ```   

3. To run the server, the following command is executed:

        node helloworld.js 

4. The web page served by the server can be viewed using a browser. Here lynx is used to view the web page that server is serving at port 8080. 

        lynx lab3:8080

    The output would be:
    
    ![Nodejs helloworld](./images/3_nodejs_helloworld.png)

- The contents of the JavaScript file helloworld.js is explained below:

1. 
    ```javascript 
    var http = require("http");
    ```

    The require directive is to load the http module and store the returned HTTP instance into an http variable
2. 
   ```javascript 
    http.createServer()
    ``` 
    The http instance calls createServer() method to create a server instance.

3. 
    ```javascript 
    server.listen(8080)
    ``` 
    To bind it to port 8080, the listen method is called associated with the server instance.

4. ```javascript 
    res
    ```

    res is the response object that can responds back to the browser dependinng on the values set using its associated parameters.

-  Node.js is event driven:

    Event-Driven Programming makes use of the following concepts:
    - An Event Handler is a callback function that will be called when an event is triggered.
    - A Main Loop listens for event triggers and calls the associated event handler for that event.


## Configuring SSL for Apache2

- For this task a 2048-bit key is generated for the Apache2 server and a matching certificate. Apache server is configured to utilize this certificate for HTTPS traffic. The required steps are enumerated below:

1. Generate 2048-bit key and the corresponding certificate

        sudo openssl req -new -newkey rsa:2048 -x509 -sha256 -days 365 -nodes -out MyCertificate.crt -keyout MyKey.key
2. Placing the certificate `MyCertificate.crt` and key `MyKey.key` at the following location `/etc/ssl/certs/` and `/etc/ssl/private/` respectively.
3. Here SSL is being configured for the default web page served by the Apache2 server. For this the following information is appended to the file `/etc/apache2/sites-available/000-default.conf`:

    ``` xml
    <VirtualHost *:443>

        SSLEngine On
        SSLCertificateFile /etc/ssl/certs/MyCertificate.crt
        SSLCertificateKeyFile /etc/ssl/private/MyKey.key

        ServerAdmin webmaster@localhost
        DocumentRoot /var/www/html

        ErrorLog ${APACHE_LOG_DIR}/error.log
        CustomLog ${APACHE_LOG_DIR}/access.log combined
    </VirtualHost>
    ```
4. The following command is executed to ensure that the Apache SSL module is enabled:

        a2enmod ssl
5. Now the server is restarted with the command:

        service apache2 restart
6. To verify if SSL is configured, SSH port-forwarding can be used. Note that here port 443 (HTTPS port) is forwarded from the VM lab2 to the host machine's port 1234 with SSH:

        ssh -L localhost:1234:10.0.2.9:443 -p 10011 lab2@localhost

    The browser from the host system can be used to access the web page from the SSL-enabled server by providing an HTTPS URL - https://localhost:1234/.

    Since the browser tries to verify the certificate created above and the as certificate cannot be verified, it will throw a warning that the site cannot be trusted:

    ![Cannot Trust Certificate](./images/4_cannot_verify.png)

    After proceeding by accepting the risk in the advanced options, the default web page will be served:

    ![Served Untrusted Site](./images/5_loads_webpage.png)

- Certificate and its neccessity in the context of web servers:

    If the signature is valid, and the software examining the certificate trusts the issuer, then it can use that key to communicate securely with the certificate's subject.
    In TLS, a certificate's subject is typically a computer or other device, though TLS certificates may identify organizations or individuals in addition to their core role in identifying devices

    In TLS (an updated replacement for SSL), a server is required to present a certificate as part of the initial connection setup. A client connecting to that server will perform the certification path validation algorithm:
    1. The subject of the certificate matches the hostname (ie. domain name) to which the client is trying to connect.
    2. The certificate is signed by a trusted certificate authority.

- PKI:

    A public key infrastructure (PKI) is a set of roles, policies, and procedures needed to create, manage, distribute, use, store & revoke digital certificates and manage public-key encryption. It is required for activities where simple passwords are an inadequate authentication method and more rigorous proof is required to confirm the identity of the parties involved in the communication and to validate the information being transferred. (eg: Internet Banking)

## Enforcing HTTPS

- For this task a subdirectory called `secure_session` is created to the public_html directory of the user `lab2`. The userdir module is used to serve public_html from the user's home directory and HTTPS is enforced using `mod_rewrite` and `.htaccess`. The steps required for this task are enumerated below:

1. Enable Apache2 to use the userdir module with the following command:

        sudo a2enmod userdir
2. Now to access the `public_html` directory:

        mkdir ~/public_html
        chmod -R 755 ~/public_html

    Apache server is restarted for the changes to take affect.

        sudo service apache2 restart
3. To activate mod_rewrite module:

        sudo a2enmod rewrite
    Apache server is restarted for changes to take affect.

        sudo service apache2 restart

>The [mod_rewrite](https://httpd.apache.org/docs/2.4/mod/mod_rewrite.html "mod_rewrite module reference") module uses a rule-based rewriting engine, based on a PCRE regular-expression parser, to rewrite requested URLs on the fly.

4. Now the subdirectory `secure_session` is created in the `public_html` directory:

        cd ~/public_html
        mkdir secure_session
        cd secure_session

5. The URL rewrites can be setup using `.htaccess` file in the directory; `.htaccess` file is a way to configure the details of the website without altering the server config files. The file is placed in `secure_session` directory since the rewrite rule only applies to that directory.

        sudo vim .htaccess

    The contents of the file are the Rewrite rules, this rule is specifically made for this task:

        RewriteEngine On
	    RewriteCond %{HTTPS} off
	    RewriteRule .* https://lab2%{REQUEST_URI} [R=301,L]
    
    Apache server is restarted for changes to take affect.

        sudo service apache2 restart

6. This configuration can be verified by using lynx:

        lynx http://localhost/~lab2/secure_session
    
    It can be seen that the query will change from http://localhost/~lab2/secure_session to https://localhost/~lab2/secure_session during execution.

- HSTS:

    HTTP Strict Transport Security (HSTS) is a web security policy mechanism that helps to protect websites against protocol downgrade attacks and cookie hijacking. It allows web servers to declare that web browsers (or other complying user agents) should interact with it using only secure HTTPS connections and never via the insecure HTTP protocol.

- .htaccess:

    .htaccess is a configuration file for use on web servers running the Apache Web Server software. When a .htaccess file is placed in a directory which is in turn 'loaded via the Apache Web Server', then the .htaccess file is detected and executed by the Apache Web Server software. 

    When to use .htaccess?
        
    .htaccess files should be used in a case where the content providers need to make configuration changes to the server on a per-directory basis, but do not have root access on the server system. In the event that the server administrator is not willing to make frequent configuration changes, it might be desirable to permit individual users to make these changes in .htaccess files for themselves. This is particularly true, for example, in cases where ISPs are hosting multiple user sites on a single machine, and want their users to be able to alter their configuration.

    What should be considered:

    1. The first of these is performance. When AllowOverride is set to allow the use of .htaccess files, httpd will look in every directory for .htaccess files. Thus, permitting .htaccess files causes a performance hit, whether or not you actually even use them. Also, the .htaccess file is loaded every time a document is requested.

    2. Secondly, for each file access out of that directory, there are 4 additional file-system accesses, even if none of those files are present. (Note that this would only be the case if .htaccess files were enabled for /, which is not usually the case.)


## Nginx as a Reverse Proxy

- For this task, nginx is installed on `lab1` and it is configured to act as a gateway to both Apache2 at lab2 and Node.js at lab3. Specifically HTTP requests to http://lab1/apache are directed to Apache2 server listening on lab2:80 and requests to http://lab1/node to Node.js server on lab3:8080. The required steps are enumerated below:

1. In lab1, nginx is installed:

        sudo apt-get update
        sudo apt-get upgrade
        sudo apt-get install nginx

2. Reverse proxy configuration is specified in the file `nodeapp.conf` that is placed in `/etc/nginx/conf.d/`. The contents of the configuration file is:

    ```
    server {
    listen 80;
    listen [::]:80;

    server_name localhost;

    location /apache {
        rewrite ^/apache(.*) /$1 break;
        proxy_pass https://10.0.2.4:443;
    }

    location /node {
        rewrite ^/node(.*) /$1 break;
        proxy_pass http://10.0.2.15:8080;
    }

    }
    ```

3. Once the servers are verified to be running, the reverse proxy can be tested using curl or lynx:

        lynx http://localhost/apache

    ![Reverse Proxy - Apache](./images/6_lab1_apache.png)

        lynx http://localhost/node

    ![Reverse Proxy - Nodejs](./images/7_lab1_node.png)

- NGINX:

    NGINX is a powerful web server and uses a non-threaded, event-driven architecture. It can also do other important things, such as load balancing, HTTP caching, or be used as a reverse proxy. The main purpose of the master process is to read and evaluate configuration, and maintain worker processes. Worker processes do actual processing of requests. nginx employs event-based model and OS-dependent mechanisms to efficiently distribute requests among worker processes.
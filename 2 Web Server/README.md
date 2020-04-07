# Web Server
This task requires 3 VM hosts: `lab1` (10.0.2.8), `lab2` (10.0.2.9) and `lab3` (10.0.2.10). 

> [Lynx](https://lynx.browser.org/ "Lynx Reference") is a text browser for the World Wide Web which is used for these tasks. Lynx can be install using the command `sudo apt-get install lynx`
## Apache2

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

>Node.js 

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

    To run the server, the following command is executed:

        node helloworld.js 

    The web page served by the server can be viewed using a browser. 

        lynx lab3:8080

    The output would be:
    
    ![Nodejs helloworld](./images/3_nodejs_helloworld.png)

3. 

4. 

<!-- ## Configuring SSL for Apache2

For this task a 2048-bit key is generated for the Apache2 server and a matching certificate. Apache server is configured to utilize this certificate for HTTPS traffic.

- Generate 2048-bit key and the corresponding certificate

        openssl req –new –newkey rsa:2048 –nodes –keyout server.key –out server.csr
- Placing the certificate and key at the following respective location `/etc/ssl/certs/` and `/etc/ssl/private/` as `apache-selfsigned.crt` and `apache-selfsigned.key` respectively. -->


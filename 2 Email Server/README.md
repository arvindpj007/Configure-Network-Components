# Email Server
## 1. Preparations
    
This task requires 2 hosts (`lab1` and `lab2`). All daemons that are listening on the default SMTP port are stopped. 
    
IPv4 addresses and aliases of lab1 and lab2 are added on both computers for easeness. This can be done by changing the `/etc/hosts` file on both the hosts. 

In lab2 `/etc/hosts` file add the entry:

    10.0.2.8 lab1

In lab1 `/etc/hosts` file add the entry:

    10.0.2.9 lab2

## 2. Installing software

On `lab1` install the following packages - `postfix`, `procmail`, `spamassassin`.

> [Postfix](http://www.postfix.org/start.html "Reference for Postfix") - It is a mail server that attempts to be fast, easy to administer, and secure. Postfix has several hundred configuration parameters that are controlled via the `main.cf` file. 
    
Postfix email server can be installed with the following commands:

    sudo apt update
    sudo apt install postfix

> [procmail](https://wiki.ubuntu.com/Procmail "Reference for procmail") - Procmail is a versatile e-mail processor that can be used to sort your incoming mail. You configure it by creating a `~/.procmailrc` file.

procmail can be installed with the following commands:

    sudo apt-get update
    sudo apt-get install procmail

> [spamassassin](https://wiki.ubuntu.com/Procmail "Reference for spamassassin") - Apache SpamAssassin is an intelligent software application for filtering unsolicited emails from telemarketers and hackers.  The utility runs on top of a Mail Transfer Agent (MTA) like Postfix to classify and block unwanted emails.

 spamassassin can be installed with the following commands:

    sudo apt-get update
    sudo apt-get install spamassassin


On `lab2` install the package - `exim4`

> [exim4](https://wiki.ubuntu.com/Procmail "Reference for Exim4") - Exim4 is another Message Transfer Agent (MTA) for use on Unix systems connected to the Internet.

 exim4 can be installed with the following commands:

    sudo apt-get update
    sudo apt-get install exim4

## 3. Configuring postfix and exim4
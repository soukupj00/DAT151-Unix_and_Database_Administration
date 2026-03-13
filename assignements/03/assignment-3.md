# DAT151: Assignment 3 Report

**Group:** 2

**Group Members:** Soukup Jan, Fabienne Feilke

**Date:** March ..., 2026

---

## Task 1: SELinux

*Objective: Work with SELinux contexts, users, domains, and policies.* 

### 1a: SELinux Status


**Task:** Check whether SELinux is currently enabled and make sure it is in targeted and enforcing mode.

* **Commands Used:**
* `sestatus`


* **Terminal Output / Printout:**



![Screenshot](https://github.com/user-attachments/assets/9df951b1-b9b5-48db-925c-7ce8ddc87d74)



* **Explanation:**
Under SELinux status we see that SELinux is already enabled and under Current mode we check that we are in enforcing Mode.The loaded policy name is also set to targeted. In action SELinux consults its Targeted Policy database. It asks: "Is a process with label A allowed to perform action X on a file with label B. The enforcing mode ensures that unallowed operations will be blocked.

### 1b: Working with SELinux Users

* **1b.1 Mapping Linux users to SELinux users:**

**Task:** List the mapping between Linux users and SELinux confined users on your computer.

* **Commands used:**
* `sudo semanage login -l`
* **Output:**

![Screenshot](https://github.com/user-attachments/assets/85b29502-3529-4f2d-aa75-9c928434365f)

* **Explanation:**
* Any new user you create will automatically be mapped to the SELinux user listed under default (unconfined_u). SELinux doesn't interrupt daily tasks but still protects specific network services.



* **1b.2 Available SELinux Users:**
  
**Task:** List all available SELinux users on your computer using `seinfo`.


* **Commands used:**
* `seinfo -u`
  
* **Output:**

![Screenshot](https://github.com/user-attachments/assets/94a280af-fa1a-4269-afb1-cac4e9e40353)


* **1b.3 Restricting `su` and `sudo`:**
  
**Task:** Create a new Linux user and use SELinux to prevent this user from using the `su` and `sudo` tools.

* **Commands & Output:**

![Screenshot](https://github.com/user-attachments/assets/a25a2764-e97b-473b-b109-20d8cba48480)
![Screenshot](https://github.com/user-attachments/assets/1f621a33-5269-4dd0-b234-90b1594f078f)



* **Explanation of Method:**
* We change the restricted User from a unconfined_u to a user_u
* _t (Type Enforcement)s the primary way permissions are handled
* The tools su and sudo are setuid binaries -> when a user runs them, the process attempts to change its Effective User ID to root (UID 0) and SELinux blocks this for user_u


### 1c. 

Domain Transitions (systemd to Apache) 

**Task:** Verify that an Apache web server started by systemd has read access to the content of `/var/www/html`. Note: For each step below, you must use the `sesearch` command and document the applicable policy rules.


  
**1c.1 Is systemd allowed to start the Apache web server?** 


* SELinux type of systemd: init_t

* SELinux type of Apache executable: httpd_exec_t

* Is systemd allowed to run it?: Yes

**Commands & Policy Output (`sesearch`):** 

![Screenshot](https://github.com/user-attachments/assets/5a55911c-acd3-43de-81ed-97c64b77a937)

* This rule explicitly allows the init_t domain to perform the execute operation on files labeled httpd_exec_t
  
**1c.2 Can the Apache web server run in domain `httpd_t`?** 

**Commands & Policy Output (`sesearch`):** 


![Screenshot](https://github.com/user-attachments/assets/ff7e1fa3-cb68-43c0-9419-4c17a7964f41)

* When a process in init_t executes a file of type httpd_exec_t, the resulting process should run in the httpd_t domain
  
**1c.3 Is systemd allowed a transition to `httpd_t`?** 


**Commands & Policy Output (`sesearch`):** 

![Screenshot](https://github.com/user-attachments/assets/72bbe979-4d9c-43d6-8bb5-2f2a87036ce9)

* This rule grants init_t the permission to actually transition the process context into httpd_t
* 
**1c.4 Has domain `httpd_t` access to open and read files in `/var/www/html`?** 


**Commands & Policy Output (`sesearch`):** 

![Screenshot](https://github.com/user-attachments/assets/b60b3907-7194-40e9-b562-ef8dc27425ab)

* This rule allows the Apache domain (httpd_t) to open and read files labeled as system web content (httpd_sys_content_t)

### 1d. 

SELinux Boolean for `public_html` 

**Task:** Use a SELinux boolean to allow Apache to read web content in a `public_html` directory in the home directory of users.


* **Commands & Output:**

![Screenshot](https://github.com/user-attachments/assets/8a26d8b3-b91f-4388-8682-e63d3bcb8d0f)



### 1e. 

Custom Web Directory `/www` 

**Task:** Create a directory `/www` and configure SELinux to allow Apache to read web content in this directory.


* **Commands & Output:**

![Screenshot](https://github.com/user-attachments/assets/2665a31b-e69e-4d8d-b64f-17ed0d1ee0c9)


---

## Task 2: Printing 

**Task:** Install CUPS and all necessary dependencies, start the cups service, set it to run on startup, and add the HVL printing system.


* **Commands & Output (Installation & Setup):**
  

![Screenshot](https://github.com/user-attachments/assets/55fc6f79-1ba6-4c8d-b713-13f089ad32c0)

* cd '/home/ffeilke/Downloads/myPrintInstaller_LINUX'
* './installprinter.sh'


![Screenshot](https://github.com/user-attachments/assets/c06c0704-48f5-4934-b2d4-f90ebfb151b6)


**2.1 Print job from LibreOffice:**

* cd /home/ffeilke/Downloads/LibreOffice_26.2.1_Linux_x86-64_rpm/LibreOffice_26.2.1.2_Linux_x86-64_rpm/RPMS
* sudo dnf install *.rpm -y

![Screenshot](https://github.com/user-attachments/assets/ccf95867-37ed-4c69-a3c5-3e2314cdbbc3)


**2.2 Print job from command line (`lpr`):** 


* **Commands & Output:**

![Screenshot](https://github.com/user-attachments/assets/2f6873a0-2dda-45ae-b15c-abfdc9d28113")



---

## Task 3: Open Ports and Processes 

**Task:** Find all ports open for tcp and udp connections on your computer, and list the processes and services that use these ports.

* sudo ss -tulpn
* -t: Display TCP sockets

* -u: Display UDP sockets

* -l: Show only listening sockets (ports waiting for a connection)

* -p: Show the process name and PID using the socket

* -n: Show numerical port numbers


![Screenshot](https://github.com/user-attachments/assets/3eaa38dd-7b2e-430b-a7b5-0cd570d5af88)


* **Explanation of Findings:**
  
* CUPS - Port 631 (TCP)
* Avahi - Port 5353(UDP)
* WSDD - Ports 3702, 39389, 34194 (UDP)
* SSH - Port 22 (TCP)
*  Cockpit - Port 9090 (TCP)
*  MariaDB - Port 3306 (TCP)
*  Coronyd -Port 323 (UDP)
  
*  Netid shows the type of protocoll
*  State shows the status of that port
*  Recv-Q shows the amount of data (in bytes) received but not yet processed by the program
*  Send-Q shows the amount of data sent that hasn't been acknowledged by the other side yet
*  Local Address:Port defines the internal IP address and unique port number on the local machine where a specific service is hosted and listening for traffic
*  Peer Address:Port dentifies the remote IP address and port number of the external device that is either currently communicating with the local service or is permitted to connect to it




## Task 4: Two-Factor Authentication (2FA)

> **Important:** Do not modify any of the files below `/etc/ssh/`; create new files with the modifications.

* **Task:** Set up 2FA for SSH login on your computer, using an SSH key and time-based one-time passwords. Set this up on all lab computers of the group.

![Screenshot From 2026-03-02 13-49-31.png](images/Task4/Screenshot%20From%202026-03-02%2013-49-31.png)

First we installed the google authenticator and qrencode packages

![Screenshot From 2026-03-02 13-49-05.png](images/Task4/Screenshot%20From%202026-03-02%2013-49-05.png)

![Screenshot From 2026-03-02 13-48-54.png](images/Task4/Screenshot%20From%202026-03-02%2013-48-54.png)

Command to generate the secret and `google-authenticator` showing the generated secret and QR code for enrollment.

![Screenshot From 2026-03-02 14-02-23.png](images/Task4/Screenshot%20From%202026-03-02%2014-02-23.png)

![Screenshot From 2026-03-02 14-01-28.png](images/Task4/Screenshot%20From%202026-03-02%2014-01-28.png)

- Then, we created a new file `/etc/ssh/sshd_config.d/99-2fa.conf` to setup ssh config properly for the google authenticator
    - KbdInteractiveAuthentication - allows the server to prompt for the time-based one-time password (TOTP) during the login process
    - PasswordAuthentication - disable simple password login
    - AuthenticationMethods - specify what methods are used to login

![Screenshot From 2026-03-02 13-54-50.png](images/Task4/Screenshot%20From%202026-03-02%2013-54-50.png)

![Screenshot From 2026-03-02 13-53-18.png](images/Task4/Screenshot%20From%202026-03-02%2013-53-18.png)

Then we configured the /etc/pam.d/sshd config file to use the google authenticator

![Screenshot From 2026-03-09 12-09-15.png](images/Task4/Screenshot%20From%202026-03-09%2012-09-15.png)

![Screenshot From 2026-03-09 12-09-06.png](images/Task4/Screenshot%20From%202026-03-09%2012-09-06.png)

We created new key pair fot the local users, added them to `.ssh` to be available. We were then able to login to the other user account with the google authentication - using the code in the Authenticator app and password as a secure login.

---

## Task 5: Secure your computer 

**Task:** Use Fail2Ban to secure your computer. Set PAM requirements for password qualities, password history, and account locking if there are many consecutive authentication failures.


> 
> **Requirement:** Do not modify directly the PAM files; configure the PAM system using `authselect`.
>

* **Commands & Output:**

We installed and enabled Fail2Ban, and used `authselect` to enable PAM features such as account lockout and password history. Below are all Task 5 screenshots with short captions. Each image is followed by one blank line and then its caption.

Screenshots:

![Screenshot From 2026-03-09 12-13-20.png](images/Task5/Screenshot%20From%202026-03-09%2012-13-20.png)

Fail2Ban installed

![Screenshot From 2026-03-09 12-13-42.png](images/Task5/Screenshot%20From%202026-03-09%2012-13-42.png)

Running fail2ban service

![Screenshot From 2026-03-09 12-18-29.png](images/Task5/Screenshot%20From%202026-03-09%2012-18-29.png)

`authselect` profile selection and feature enabling (faillock, passwd-history).

![Screenshot From 2026-03-09 12-21-42.png](images/Task5/Screenshot%20From%202026-03-09%2012-21-42.png)

Edited faillock options to be more user friendly with wrong password inputs.

![Screenshot From 2026-03-09 12-23-11.png](images/Task5/Screenshot%20From%202026-03-09%2012-23-11.png)

Password history file edited.

![Screenshot From 2026-03-09 12-24-19.png](images/Task5/Screenshot%20From%202026-03-09%2012-24-19.png)

Password quality file edited.

![Screenshot From 2026-03-09 12-25-37.png](images/Task5/Screenshot%20From%202026-03-09%2012-25-37.png)

Test of password change showing enforcement of password policies. Tried a short 6 char password and then long password but withou enough classes, both got rejected.

---

## Task 6: SSH 

**Setup:** Create an SSH key pair as a normal user on your own computer and start an OpenSSH server on your lab computer.

6.1 Direct Login Attempt

**Task:** Try to log in to your lab computer from your own computer.

We attempted a direct SSH connection to the lab host but received a network-level failure (connection timed out). This indicates the lab host is not directly reachable from the client network (no SSH port open or firewall/gateway blocks direct access). The screenshots below show the failed connection and the local key setup.

![Screenshot From 2026-03-09 12-29-33.png](images/Task6/Screenshot%20From%202026-03-09%2012-29-33.png)

Direct connection attempt showing network-level failure - connection timeout.

![Screenshot From 2026-03-09 12-31-10.png](images/Task6/Screenshot%20From%202026-03-09%2012-31-10.png)

SSH key generation and `ssh-copy-id` confirmation - this was later changed to use the provided `https://epleauth.azurewebsites.net` method, that already has public and secret keys generated and we just copied them.

6.2 Jump Host Login

**Task:** From your own computer, log into your lab computer with a jump through `eple.hvl.no`.

Using the gateway `eple.hvl.no` as a jump host allowed us to access because the gateway has network reachability to the lab host. We used `ssh -J` to route the connection through the gateway; the SSH key authentication and agent forwarding were used to authenticate. Screenshots below show a successful jump-host login and an interactive shell on the lab host.


![Screenshot From 2026-03-09 12-29-45.png](images/Task6/Screenshot%20From%202026-03-09%2012-29-45.png)

Tried connecting first, got rejected because of the key.

![Screenshot From 2026-03-09 12-46-57.png](images/Task6/Screenshot%20From%202026-03-09%2012-46-57.png)

![Screenshot From 2026-03-09 12-45-24.png](images/Task6/Screenshot%20From%202026-03-09%2012-45-24.png)

Tried viewing the key, adding it to ssh as possible solutioons, none worked. We also added the public key to the lab computer `authorized_keys` file. This didn't work, so we had to use the `eple.hvl.no` generated key.

![Screenshot From 2026-03-09 12-47-22.png](images/Task6/Screenshot%20From%202026-03-09%2012-47-22.png)

Added the keys to private computer and lab computer as well. After that the connection was established.

6.3 MariaDB SSH Tunnel

**Task**: Install the MariaDB client tool on your own computer, set up an SSH tunnel through `eple.hvl.no` to the MariaDB server on your lab computer, and connect to it.

This task was more challenging then it should have been. We got stuck on the key authorization and discussed with the lecturer the solution for about an hour. In the end it was a combination of corrupted public key on the personal laptop and incorrect ssh calls to the lab computer. Following are screenshots showing the progress, issues we encountered, trying to find the solution and finally working MariaDB connection from personal computer. In the end we created a local port forward from `localhost:13389` to the remote MariaDB port on the lab host via the jump host and then connected the local MariaDB client to the forwarded port. The tunnel successfully forwarded traffic and the client connected to the remote database through the SSH tunnel.

![Screenshot From 2026-03-09 12-50-03.png](images/Task6/Screenshot%20From%202026-03-09%2012-50-03.png)

Installing the MariaDB client tool on personal computer.

![Screenshot From 2026-03-09 13-49-40.png](images/Task6/Screenshot%20From%202026-03-09%2013-49-40.png)

Part of debug command we and lecturer used for finding out the solution to the connection.

![Screenshot From 2026-03-09 13-49-19.png](images/Task6/Screenshot%20From%202026-03-09%2013-49-19.png)

Finally working MariaDB client session over the tunnel after a lot of struggle and debugging. 

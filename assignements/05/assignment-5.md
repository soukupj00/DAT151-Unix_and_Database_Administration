# DAT151: Assignment 5 Report

**Group:** 5

**Group Members:** Soukup Jan, Fabienne Feilke

**Date:** March 2026

---

## Task 1: Logging

### a) Configure systemd journal
*Configure systemd journal at the lab to save the logs persistent in the file system.*

The screenshots for this step show the journald configuration workflow: copying the default file to `/etc/systemd/journald.conf`, creating a backup copy in `/root/origs/`, and editing the config file.

![journald config copy and edit](images/task1/Screenshot%20From%202026-03-16%2012-14-52.png)
![journald config open in editor](images/task1/Screenshot%20From%202026-03-16%2012-15-11.png)
![backup directory creation](images/task1/Screenshot%20From%202026-03-16%2012-14-31.png)

The configuration was applied by copying the default file, editing it, and restarting the service. After that, `journalctl --flush` was used so the current journal was written into the persistent directory, and `ls -la /var/log/journal/` confirmed that the journal directory existed.

```bash
sudo cp /usr/lib/systemd/journald.conf /etc/systemd/journald.conf
sudo nano /etc/systemd/journald.conf
sudo systemctl restart systemd-journald
sudo journalctl --flush
sudo ls -la /var/log/journal/
```

![journal directory and tmpfiles setup](images/task1/Screenshot%20From%202026-03-16%2012-18-28.png)
![journal flush and persistent path check](images/task1/Screenshot%20From%202026-03-16%2012-21-18.png)

This means the system will keep journal entries across reboots instead of storing them only in memory.


### b) Protect journal logs
*Protect the journal logs from unnoticed alteration by enabling the Seal feature and create a sealing key. Verify the logs and prove that the logs are sealed. The verification output will show a time interval that is not sealed.*

The screenshot below shows the sealing setup and verification step. After restarting `systemd-journald`, `journalctl --setup-keys` was run to generate the sealing key, and `journalctl --verify` was used to check the journal files.

![journal sealing and verification](images/task1/Screenshot%20From%202026-03-16%2012-17-14.png)

```bash
sudo systemctl restart systemd-journald
sudo journalctl --setup-keys
sudo journalctl --verify
```

The output shows a warning that the build was compiled without forward-secure sealing support, but the verification still reports `PASS` for the journal files that were checked. That is the verification output used to demonstrate the integrity of the stored journal.


### c) Syslog local5 facility
*The syslog facility names “local0” to “local7” are used for local custom messages defined by an administrator. Create your own configuration that saves only the critical level and information level messages from the local5 facility to a file local5 in the /var/log directory. Test your configuration using the logger command with specific messages. Look under the /var/log directory and verify that the messages you added are saved at the correct location.*

The screenshots show creating `/etc/rsyslog.d/local5.conf` and editing the file with a custom rule. The rule writes only `local5.info` and `local5.crit` messages to `/var/log/local5`.

![rsyslog local5 setup commands](images/task1/Screenshot%20From%202026-03-16%2012-41-25.png)
![rsyslog local5 rule](images/task1/Screenshot%20From%202026-03-16%2012-41-44.png)

```bash
sudo nano /etc/rsyslog.d/local5.conf
sudo systemctl restart rsyslog
logger -p local5.info "Task 1c: This is an INFO message."
logger -p local5.crit "Task 1c: This is a CRITICAL message."
logger -p local5.err "Task 1c: This is an ERROR message and should be ignored."
sudo cat /var/log/local5
```

![local5 logger test results](images/task1/Screenshot%20From%202026-03-16%2013-37-12.png)

The configuration line `local5.=info;local5.=crit    /var/log/local5` means that only messages from facility `local5` with severity `info` or `crit` are stored in that file. The `err` message does not match the rule and is therefore not written there. The screenshot of `/var/log/local5` shows the `INFO` and `CRITICAL` messages in the correct location.


---

## Task 2: LDAP

*In this task you will set up an LDAP server and client(s). Use one of the computers of the group as the LDAP server, and the others as clients.*

### Installation and Configuration
*Document your configurations clearly, include your LDIF files in your report, and explain how user records were added to your server.*

We installed OpenLDAP server/client packages.

![OpenLDAP installation](images/task2/Screenshot%20From%202026-03-23%2012-25-28.png)

Then we enabled and verified `slapd`.

![slapd enable and status](images/task2/Screenshot%20From%202026-03-23%2012-25-38.png)

```bash
sudo dnf install openldap-clients openldap-servers
sudo systemctl enable --now slapd
sudo systemctl status slapd
```

`openldap-clients` provides the LDAP client tools such as `ldapadd`, `ldapmodify`, and `ldapsearch`, while `openldap-servers` installs the server daemon `slapd`. Enabling the service makes the LDAP server start automatically on boot.


### DN Suffix and Manager Password
*Configure the database with the new DNs (base DN and administrator DN) and set up the manager password and access.*

**Base DN LDIF (`basedn.ldif`):**
```ldif
dn: olcDatabase={2}mdb,cn=config
changetype: modify
replace: olcSuffix
olcSuffix: dc=h68,dc=dat151
replace: olcRootDN
olcRootDN: cn=Manager,dc=h68,dc=dat151
```

**Manager Config LDIF (`manager.ldif`):**
```ldif
dn: olcDatabase={2}mdb,cn=config
changetype: modify
add: olcRootPW
olcRootPW: {SSHA}<hash generated with slappasswd>
add: olcAccess
olcAccess: to * by dn="cn=Manager,dc=h68,dc=dat151" write by self write by * read
```

First we create `basedn.ldif`, run `ldapmodify`, and inspect the initial error.

![basedn initial modify error](images/task2/Screenshot%20From%202026-03-23%2012-27-24.png)

![basedn initial draft](images/task2/Screenshot%20From%202026-03-23%2012-27-39.png)

Then we generate manager password hash and draft `manager.ldif`.

![manager password hash generation](images/task2/Screenshot%20From%202026-03-23%2012-28-08.png)

![manager ldif draft](images/task2/Screenshot%20From%202026-03-23%2012-28-36.png)

Then we fix LDIF separators/format and apply updated suffix/root DN.

![base DN and manager configuration](images/task2/Screenshot%20From%202026-03-23%2012-29-03.png)

![basedn corrected content](images/task2/Screenshot%20From%202026-03-23%2012-30-55.png)

![basedn apply success](images/task2/Screenshot%20From%202026-03-23%2012-30-46.png)

We apply manager permissions and password, then replace root password using `fixpw.ldif`.

![manager ldif with placeholders](images/task2/Screenshot%20From%202026-03-23%2012-31-18.png)

![manager ldif with real hash](images/task2/Screenshot%20From%202026-03-23%2012-31-53.png)

![manager apply success](images/task2/Screenshot%20From%202026-03-23%2012-32-12.png)

![new manager hash for rootPW replace](images/task2/Screenshot%20From%202026-03-23%2012-37-11.png)

![fixpw ldif content](images/task2/Screenshot%20From%202026-03-23%2012-40-43.png)

![fixpw apply](images/task2/Screenshot%20From%202026-03-23%2012-40-34.png)

The changes were applied with `ldapmodify -Y EXTERNAL -H ldapi:/// -f basedn.ldif` and `ldapmodify -Y EXTERNAL -H ldapi:/// -f manager.ldif`. This updates the live OpenLDAP configuration database under `cn=config` without editing the backend files by hand.


### Schemas
*Install necessary schemas (COSINE, NIS).*

We run COSINE/NIS schema add commands (output shows duplicate attributes, i.e., already loaded).

![schema add output](images/task2/Screenshot%20From%202026-03-23%2012-33-03.png)

Then we verify schema entries with `ldapsearch`.

![schema verification output](images/task2/Screenshot%20From%202026-03-23%2012-35-08.png)

After that we add `inetorgperson` schema required for user object classes.

![inetorgperson schema add](images/task2/Screenshot%20From%202026-03-23%2012-52-55.png)

```bash
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/cosine.ldif
sudo ldapadd -Y EXTERNAL -H ldapi:/// -f /etc/openldap/schema/nis.ldif
sudo ldapsearch -LLL -Y EXTERNAL -H ldapi:/// -b "cn={2}nis,cn=schema,cn=config"
```

The `ldapsearch` output shows that the schema entries are present in the configuration database, including `core`, `cosine`, and `nis`.


### Create and Fill User Records Database
*Create the database for user records and fill it with users and groups.*

**Database DN LDIF (`top.ldif`):**
```ldif
dn: dc=h68,dc=dat151
dc: h68
objectClass: top
objectClass: domain
```

**User/Group Creation:**
The database was filled in two steps. First, the organizational units for groups and people were created, and then the actual user entry was added under `ou=People`.

**Organizational Units (`ou.ldif`):**
```ldif
dn: ou=Group,dc=h68,dc=dat151
ou: Group
objectClass: organizationalUnit

dn: ou=People,dc=h68,dc=dat151
ou: People
objectClass: organizationalUnit
```

**User Entry (`justuser.ldif`):**
```ldif
dn: uid=testuser,ou=People,dc=h68,dc=dat151
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: testuser
sn: User
givenName: Test
cn: Test User
displayName: Test User
uidNumber: 2000
gidNumber: 2000
userPassword: {SSHA}<hash generated with slappasswd>
gecos: Test User
loginShell: /bin/bash
homeDirectory: /home/testuser
```

**Group Entry:**
```ldif
dn: cn=testgroup,ou=Group,dc=h68,dc=dat151
objectClass: posixGroup
cn: testgroup
gidNumber: 2000
```

We prepare database entry and verify base DN query.

![top ldif content](images/task2/Screenshot%20From%202026-03-23%2012-33-34.png)

![base dn ldapsearch verification](images/task2/Screenshot%20From%202026-03-23%2012-41-05.png)

We set LDAP client defaults and create organizational units.

![ldap client defaults file edit](images/task2/Screenshot%20From%202026-03-23%2012-44-32.png)

![ou ldif content](images/task2/Screenshot%20From%202026-03-23%2012-44-56.png)

![adding organizational units](images/task2/Screenshot%20From%202026-03-23%2012-45-18.png)

We first attempted  combined `userdata.ldif`, then switched to corrected `justuser.ldif`.

![userdata ldif draft](images/task2/Screenshot%20From%202026-03-23%2012-47-15.png)

![user password hash generation](images/task2/Screenshot%20From%202026-03-23%2012-47-35.png)

![userdata add error (objectClass syntax)](images/task2/Screenshot%20From%202026-03-23%2012-47-59.png)

![justuser ldif content](images/task2/Screenshot%20From%202026-03-23%2012-52-43.png)

![justuser add success](images/task2/Screenshot%20From%202026-03-23%2012-54-28.png)

Finally we opened the LDAP service in firewall.

![firewall ldap service open](images/task2/Screenshot%20From%202026-03-23%2012-48-33.png)

```bash
sudo ldapadd -D "cn=Manager,dc=h68,dc=dat151" -x -W -f top.ldif
sudo ldapadd -D "cn=Manager,dc=h68,dc=dat151" -x -W -f ou.ldif
sudo ldapadd -D "cn=Manager,dc=h68,dc=dat151" -x -W -f justuser.ldif
sudo ldapsearch -LLL -x -b "dc=h68,dc=dat151" dn
```

The final `ldapsearch` verification confirms that the base DN, both organizational units, and the user record exist in the directory.

---

## Task 3: SSSD

*Configure SSSD to use the LDAP from Task 2 for user authentication. You will also configure LDAPS (LDAP over SSL/TLS).*

### SSSD Configuration
*Configure `/etc/sssd/sssd.conf`, use `authselect` to select the PAM profile for SSSD, and enable `with-mkhomedir`.*

![Screenshot](https://github.com/user-attachments/assets/16c08c36-6390-44e9-9a15-3bd25a68c173)

![Screenshot](https://github.com/user-attachments/assets/da1a3e00-e016-41de-949f-08318e0ab1ad)





### Login Verification (Client)
*The client computer should be a different computer than the LDAP host, and the user should not exist on the client prior to log in. You must must a add screen shot to the report of a working log in on the client: `su - <some_user>` (To be run on the client, NOT the LDAP host).*

**Screenshot:**
<!-- Paste screenshot or terminal output of successful login -->
![Login Verification](path/to/image.png)


### LDAP Access Control
*With login working from the client using SSSD, the LDAP access rules should be restricted further by modifying the LDAP `olcAccess` parameter. See e.g. the man pages slapd-config(5) and slapd.access(5).*

<!-- LDIF or commands used to modify olcAccess to restrict access -->


### LDAPS Configuration
*Configure the LDAP server to use the encrypted `ldaps` channel. Create/Sign certificates, configure standard hostname resolution (/etc/hosts), and update LDAP configuration.*

**Certificate Setup:**
<!-- Explain how certificates were created/signed (openssl, CA setup) -->

**LDAP Cert Configuration:**
<!-- LDIF used to add olcTLSCertificateFile and olcTLSCertificateKeyFile to cn=config -->

**SSSD Update:**
<!-- Changes to sssd.conf to use ldaps URI and CA certificate -->


---

## Task 4: Kerberos

*Configure a host as a KDC (Key Distribution Center) and also use it as a Kerberos client to authenticate SSH logins.*

### KDC Configuration
*Configure KDC, create the database, and enable services (`kadmin`, `krb5kdc`). Create the KDC database with a secure password.*

At first we installed the krb5-server and the krb5-workstation.
![Screenshot](https://github.com/user-attachments/assets/6a7eca28-e500-4c29-b79d-61e2b17453ca)

We Configured the Kerberos Realm to use our Domain DATALAB.HVL.
![Screenshot](https://github.com/user-attachments/assets/e7b59d64-2321-4c4a-b598-e507e621f4d4)
![Screenshot](https://github.com/user-attachments/assets/66e89e53-e51c-4f3a-b2ae-954ac8ac7153)

Then we createt our database
![Screenshot](https://github.com/user-attachments/assets/057944d6-facd-44c5-be4d-2cbade75f516)
-s flag creates a "stash file" so the KDC can start automatically without you typing the master password every time it boots.
-r flag allows us to explicitly specify the name of the Kerberos realm.

We need the firewall to open the ports that kerberus and kadmin are mapped to.
![Screenshot](https://github.com/user-attachments/assets/25e3f571-434c-4d54-b63e-e5c50eb75bca)

In the Access Control List we give the admin full permission to add, delete, or modify other users (principals).
![Screenshot](https://github.com/user-attachments/assets/7d0148a9-13ad-47a1-aea8-2fe0821b6ff7)

We use this command to edit the Access Control List:
![Screenshot](https://github.com/user-attachments/assets/54f9a33f-5fab-4f43-ae80-41428dd76c36)



### Principals
*Create necessary principals (User, Admin, Host, Service). Since only root can run kadmin.local, and the default principal of root is “root/admin”, create an administrator principal “root/admin”.*

**Principals Created:**
*   User: testuser@DATALAB.HVL
*   Admin:root/admin@DATALAB.HVL
*   Host:host/hostname68.datalab@DATALAB.HVL)

**Commands:**
![Screenshot](https://github.com/user-attachments/assets/439bbc93-db4b-4c1b-bc16-d88745a9727a)

With ktadd we export the Host Principal into a keytab file. This is necessary for an authentification without a password.

### SSH with Kerberos
*Configure SSH to use Kerberos. Verify login in both directions.*

**Configuration Changes:**
![Screenshot](https://github.com/user-attachments/assets/31df3ffe-462c-40e6-9122-7bfa4f23303f)
![Screenshot](https://github.com/user-attachments/assets/727d5231-3c94-4744-ba3d-f4e1d38d3938)
![Screenshot](https://github.com/user-attachments/assets/43994574-19d1-40e5-9836-b9e0f3af6ae0)
![Screenshot](https://github.com/user-attachments/assets/130456e2-ebf8-4ca2-bd58-0b9a77d3438a)
![Screenshot](https://github.com/user-attachments/assets/6a8889e7-678d-47e0-8b01-16612a0c81f1)





**Verification:**
![Screenshot](https://github.com/user-attachments/assets/ad0ed468-a8d7-449f-992c-dfb6cb1b7ccf)


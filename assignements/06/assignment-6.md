# DAT151: Assignment 6 Report

**Group:** 5

**Group Members:** Soukup Jan, Fabienne Feilke

**Date:** April ..., 2026

---

## Task 1: LVM

In assignment 0 you were asked to keep at least 10GiB free space on the disk. You will need that free space in this task.
If the free space was assigned to a partition during install, the installation program will have set up a file system on the partition. You must remove the file system before you can use the partition with LVM.
If you did not leave 10GiB of free space on the disk, you must reinstall AlmaLinux on the computer, i.e. redo all of assignment 0.
Actions to remove the file system of the partition are:

- If the partition is mounted, unmount the partition.
- If the partition is listed in the /etc/fstab file, comment out or remove the entry from that file.
- If you have created a systemd mount unit for the partition, delete the mount unit file and reload the daemons.
- Use the dd tool to write zeros (/dev/zero) to the first 100MiB of the partition, e.g. /dev/sda4.
- If you pick the wrong partition, you will destroy your system!
- In order to recover if you clear the wrong partition, first use the dd tool to dump the first 100MiB to a file, e.g. /root/origs/sda4.img.
- Use e.g. the tool parted and delete the partition.
On the empty disk space, create a physical partition but leave 5GiB of empty space. The remaining 5GiB of empty space should for now not be assigned to a physical partition.
Set up the physical partition as a LVM physical volume (PV). Create a new logical volume group (VG) using the new PV.
When you have setup the VG, create some LVs, and make the LVs available to the system. Then save some files on the LVs.
After setting up LVM, increase the volume group by 5 GiB.
Describe and explain each step taken.

### a) Initial disk state and partition creation

We found our 10GiB Partition. It it the partition sda4.

![Screenshot](https://github.com/user-attachments/assets/14263d2c-aa3c-4091-a197-6d27007d6434)


![Screenshot](images/task1/Screenshot%20From%202026-04-13%2012-27-59.png)

We unmounted sda4:
```bash
unmount /dev/sda4
```

We also checked the /etc/fstab file:

```bash
sudo vim /etc/fstab
```
And checked for costum .mount files:

```bash
ls /etc/systemd/system/*.mount
```
To have a Backup we saved the first 100MiB before clearing the filesystem.
We used the dd tool to write zeros (/dev/zero) to the first 100MiB of the partition.
After we deleted the Partition and returned the free space

![Screenshot](https://github.com/user-attachments/assets/e8306d50-dda2-4b20-9b7d-132454376850)

We createt a physical partition of 5GiB to leave 5GiB of empty space.

![Screenshot](https://github.com/user-attachments/assets/2048500d-bccb-42b7-894d-bdbdaac5cca8)

### b) Create PV, VG and LVs

We set up the physical partition as a LVM physical volume (PV) and created a logical volume group (VG).

![Screenshot](https://github.com/user-attachments/assets/b1ee80f9-2e79-4526-89eb-28f81e466537)

We createt a logical volume. Formated it and mounted it.

![Screenshot](https://github.com/user-attachments/assets/6c6bdb23-373e-458f-8ddf-d3cab592de82)

### c) Extend VG with remaining 5GiB

We createt a new partition of the remaining free space and extended the VG with this new partition. 
In the end we varify that the Volume Group has been extendet.


![Screenshot](https://github.com/user-attachments/assets/3ebeca66-378e-4363-a50d-d41a61205ac7)


## Task 2: NFS

For this task you will need two computes, one for the NFS server and the other as the NFS client.
Make sure that the user account exists in the LDAP (task 2 of assignment 5), and that the user can log in using SSSD (task 3 of assignment 5).
Move the user home directory to /share/home on the NFS server. The user home directory should not exist on the client.
Set up the NFS server and export the directory /share/home to the client. The exported directory should be mounted as /share/home on the client. If necessary, modify the LDAP property homeDirectory to be consistent with a home directory below /share/home.
Configure the NFS server to start at boot, and configure the client to mount the exported directory at boot. You can either create a systemd mount unit, or modify the /etc/fstab file.
On the client, copy or make some changes to a file in the NFS mounted directory, and then check the attributes of that file on the server.
Please document all you have done for configuring the servers and clients.
Observe that you may need to adjust the firewall settings and/or configure SELinux to make NFS mount work. For SELinux, see e.g. Adjusting the policy for sharing NFS and CIFS volumes by using SELinux booleans.
For your information, the system can be configured to mount the home directory of a user when the user logs in, and unmount on logout. This can be very useful on a system with many users, to save resources.
Automatic mounting of home directories can be achieved e.g. through PAM by the pam_mount(3) module, or by an auto mounting system. Several auto mounting systems are available for Linux, e.g. using a systemd autmount unit, or the autofs daemon.
Observe, you are not asked to configure your system to mount home directories on user login. It is sufficient in this task to mount all home directories at boot.


### a) Server configuration


We installed the necessary packages on server and client and created the directory. Then we moved the user home directory to /share/home

```bash
sudo dnf install nfs-utils -y
```

![Screenshot](https://github.com/user-attachments/assets/aaffa243-e8fe-4348-a6dd-0b5482f49070)


![Screenshot](https://github.com/user-attachments/assets/2d50a905-b6e1-4921-bc5f-8ffd868c3663)



Opening   `nfs`, `rpc-bind`, and `mountd` services in firewall:
![Screenshot](https://github.com/user-attachments/assets/8b0c393e-0af2-435c-813d-04a0a2533b16):

Configure export rule for client IP in `/etc/exports`:

![Screenshot](https://github.com/user-attachments/assets/8e74b664-e0fe-4ee9-a07f-8c1920ddf06c)




We enable the `nfs-server` and run `exportfs -rav`:

![Screenshot](https://github.com/user-attachments/assets/61d8ff7a-e7a3-4089-b0f8-8091635ddd2e)






### b) Modify LDAP User
![Screenshot](https://github.com/user-attachments/assets/ffd4527a-530b-4468-af44-7222a8fd06ef)


![Screenshot](https://github.com/user-attachments/assets/5a9e4b83-1b06-4801-9fde-aa3e231f5aac)
![Screenshot](https://github.com/user-attachments/assets/74386a67-1063-4996-adb8-0637d77e489b)

### c) Client configuration and verification

For verification we created a testfile on the client and read the same file on the server.
Insert missing pic from Client.

![Screenshot](https://github.com/user-attachments/assets/69d5e047-210d-4842-a368-80ad7c0c99f3)




### b) Modify LDAP User

*Modify LDAP user so `homeDirectory` points to `/share/home/<user>`.*

Placeholder for final write-up and screenshots:

- LDIF used for `homeDirectory` replacement
- `ldapmodify` command
- `ldapsearch` verification



## Task 3: Custom PAM profile for pam_mount

The authselect tool on AlmaLinux 10 does not provide any ready made feature that enables the PAM module pam_mount.
The task here is to create a authselect profile with a a feature that will include pam_mount with SSSD.
Observe, the task here is not to use, nor configure pam_mount for logins, but only to create an authselect profile with a feature that adds the pam_mount module to the PAM system.
Do not modify PAM files below "/etc/pam.d/" that are managed by the authselect tool. You will need to modify though files below "/etc/authselect/custom/". Be careful not to modify any of the files of "/usr/share/authselect/default/", i.e. the files that you modify must not be soft links.
To solve this task you must create a custom profile based on the SSSD profile. Add a feature to the profile that adds the PAM module pam_mount. Name the feature with-mount.
The RPM package for the pam_mount module is not available in the default repositories of EL10. I have repacked the Fedora 43 version for EL10. The necessary packages are available from the host eple.hvl.no. The commands below will add the repository on eple to your Alma 10 computer:

### a) Install `pam_mount` package from EPLE repository

On AlmaLinux 10, `pam_mount` is not in the default repositories. We added the EPLE repository and installed the package with `--enablerepo`.

```bash
cat <<EOF | sudo tee /etc/yum.repos.d/eple.repo >/dev/null
[eple]
name=eple-repo
priority=1
baseurl=http://eple.hvl.no/repos/almalinux-10/x86_64/
skip_if_unavailable=True
enabled=0
EOF

sudo rpm --import http://eple.hvl.no/repos/keys/RPM-GPG-KEY-pmanager
sudo dnf clean all
sudo dnf --enablerepo=eple install pam_mount
```

![EPLE repository setup and pam_mount installation](images/task3/Screenshot%20From%202026-04-15%2010-19-53.png)

The output shows that `pam_mount` and its dependencies were resolved from the EPLE repository.

### b) Create custom authselect profile from SSSD

We created a custom profile named `sssd` with SSSD as base profile:

```bash
sudo authselect create-profile sssd -b sssd
```

This creates editable files under `/etc/authselect/custom/sssd/` (not symlinks to `/usr/share/authselect/default/`), which is required for safe customization.

![Custom profile creation based on sssd](images/task3/Screenshot%20From%202026-04-15%2010-28-55.png)

### c) Define feature description in profile README

We edited `/etc/authselect/custom/sssd/README` and added the `with-mount` feature description so it appears in the profile feature list.

![README optional features section before addition](images/task3/Screenshot%20From%202026-04-15%2010-26-58.png)
![README lower section showing added with-mount description](images/task3/Screenshot%20From%202026-04-15%2010-27-22.png)

The feature is documented as:

```text
with-mount::
    Enable pam_mount to mount volumes for user session
```

This is important because the assignment requires the feature to be listed and documented by `authselect`.

### d) Add `pam_mount` through feature guards in PAM templates

We edited `/etc/authselect/custom/sssd/system-auth` and added `pam_mount.so` in the required places guarded by the feature flag:

- In `auth` section (before `pam_deny.so`):
  `auth    optional    pam_mount.so    {include if "with-mount"}`
- In `session` section:
  `session optional    pam_mount.so    {include if "with-mount"}`

![system-auth with pam_mount lines guarded by with-mount](images/task3/Screenshot%20From%202026-04-15%2010-35-49.png)

This ensures `pam_mount` is injected only when the `with-mount` feature is enabled.

### e) Select custom profile and enable feature

After editing the profile, we selected the custom profile with the `with-mount` feature and verified it:

```bash
authselect list
sudo authselect select custom/sssd with-mount --force
authselect current
```

![custom/sssd selected and with-mount shown as enabled](images/task3/Screenshot%20From%202026-04-15%2010-38-22.png)

The output confirms:

- `custom/sssd` appears in `authselect list`
- Profile ID is `custom/sssd`
- `with-mount` is listed under enabled features

This proves the custom profile and feature were created correctly and activated.

---

## Task 4: Decoding PAM rules

In task three, PAM rules for a SSSD profile that includes the pam_mount module was shown.
Explain the consequences of line four in the listing of PAM rules, the yellow line with pam_usertype.so.
What would have been the consequences if the line was replaced with the line below?

### Rule under analysis

```text
auth [default=1 ignore=ignore success=ok] pam_usertype.so isregular
```

![PAM rules with highlighted pam_usertype line](<notes_images/task4/image (0).png>)

### Consequences of the original rule

`pam_usertype.so isregular` checks whether the account is a regular user.

Control behavior:

- `success=ok`: if user is regular, authentication continues to the next line normally.
- `default=1`: if the result is neither success nor ignore, PAM skips one following rule.
- `ignore=ignore`: ignore leaves flow unaffected.

In this stack, the practical effect is:

- Regular user: continue with `pam_localuser.so`, then continue through the stack.
- Non-regular/system-type user: skip one entry (the `pam_localuser.so` line), then continue at the next rule (`pam_unix.so`).

So the line is used to slightly alter control flow for user categories without immediately denying authentication.

### If replaced by `auth [default=ok success=6] pam_usertype.so isregular`

```text
auth [default=ok success=6] pam_usertype.so isregular
```

Consequences:

- If `isregular` succeeds, PAM skips 6 following `auth` lines.
- In the shown order, that jump lands at/near the final deny path (`pam_deny.so`), causing regular users to fail authentication.
- If `isregular` does not succeed, `default=ok` means no skip and flow continues normally for the non-regular case.

This replacement effectively inverts intended behavior and would block normal regular-user logins.

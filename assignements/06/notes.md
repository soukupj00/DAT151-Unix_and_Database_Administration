# Task 1: LVM

- First we check if the volume is mounted. We discovered, that we have free space on the disk, but the partition did not even exist, so we proceeded to create the partition.
- We have created new 5Gb partition in the free space called `/dev/sda7`

![image.png](<notes_images/task1/image (0).png>)

- Then we created physical volume from the created partition

![image.png](<notes_images/task1/image (1).png>)

- Then we created volume group called `vg_czechia`

![image.png](<notes_images/task1/image (2).png>)

- Then we created 2 logical volumes of 2GB each called `lv_1` and `lv_2` in the volume group `vg_czechia`

![image.png](<notes_images/task1/image (3).png>)

- Next we want to add some files to the volumes, but for that we need to create filesystem and mount the volume. First we created ext4 filesystem.

![image.png](<notes_images/task1/image (4).png>)

- Now we mounted the filesystem

![image.png](<notes_images/task1/image (5).png>)

![image.png](<notes_images/task1/image (6).png>)

- And now we can finally create some files in the logical volumes

![image.png](<notes_images/task1/image (7).png>)

- Now we want to increase the volume group size with another 5Gb. For that we need to create another partition with 5Gb

![image.png](<notes_images/task1/image (8).png>)

- Then we again make physical volume from the partition and then we can extend the volume group size by the new partition

![image.png](<notes_images/task1/image (9).png>)

# Task 2: NFS

- We installed `nfs-utils`
- We have created folder `/share/home/`
- Then we have changed `/etc/exports` to export our share folder

![image.png](<notes_images/task2/image (0).png>)

- We had to change firewall rules to allow the nfs to work

![image.png](<notes_images/task2/image (1).png>)

- Now we enabled nfs server and exported our folder

![image.png](<notes_images/task2/image (2).png>)

- We also had to modify the LDAP user to use this share home folder

![image.png](<notes_images/task2/image (3).png>)

After that we started to setup the client computer. On the client computer we installed `nfs-utils`.

- We created empty folder `/share/home`. This is the folder where the server’s share home folders will be mounted via NFS
- Then we changed fstab on the client computer to mount the remote folder

![image.png](<notes_images/task2/image (4).png>)

- Then we rebooted the computer
- Then we logged in as LDAP simpleuser at the client, new directory was created, we created a test file

![image.png](<notes_images/task2/image (5).png>)

- NFS is working, we can see the content on the server

![image.png](<notes_images/task2/image (6).png>)

# Task 3: Custom PAM profile for pam_mount

- We have installed the necessary `pam_mount` package based on your description in the task
    - `sudo dnf --enablerepo=eple-alma9 install pam_mount`

![image.png](<notes_images/task3/image (0).png>)

- We have created new profile using the `sudo authselect create-profile sssd-custom -b sssd` command
    - `-b` is for base-profile
- We changed the `README` under `/etc/authselect/custom/sssd-custom` to show the new feature

![image.png](<notes_images/task3/image (1).png>)

- We changed the `/etc/authselect/custom/sssd-custom/system-auth` and added lines 20 and 47

![image.png](<notes_images/task3/image (2).png>)

- Then we could select the new profile and enable the new feature

![image.png](<notes_images/task3/image (3).png>)

# Task 4: Decoding PAM rules

![image.png](<notes_images/task4/image (0).png>)

- If the pam_usertype.so returns isregular, the success=ok is applied and we go to the next line
- If it fails, default=1 is applied, so it skips 1 line (the line with `pam_localuser.so`) and the flow proceed with the line where `pam_unix.so` is called

### **What would have been the consequences if the line was replaced with the line below?**

`auth [default=ok success=6] pam_usertype.so isregular`

- That would mean that if the user isregular, we would skip 6 lines = go directly to `pam_deny.so`, so the auth would fail
- If the user was not regular, it would continue normally
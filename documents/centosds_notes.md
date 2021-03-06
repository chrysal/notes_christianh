# CentOS DS

This is here for historical puropses and because the `autofs` stuff is still handy. (really though...use FreeIPA)

## PREREQ

1. Install CENTOS
2. Install ALL updates
3. Install bind-chroot
4. Configure DNS for domain (maybe example.org with ldap ip of 192.168.1.248 ?)

## INSTALLATION/CONFIG


Install following repos

* epel
* Centos-testing

Install packages

```
root@host# yum --enablerepo=epel -y install java-1.6.0-openjdk.i386
root@host# yum --enablerepo=c5-testing -y install centos-ds
root@host# yum -y install xorg-x11-xauth bitstream-vera-fonts dejavu-lgc-fonts urw-fonts
```

Create CDS user/group

```
root@host# useradd -c "Centos Directory Server" -s /bin/true cds
root@host# echo "/bin/true" >> /etc/shells
```

Fix Logs

```
root@host# cd /var/log
root@host# chgrp cds dirsrv && chmod g+w dirsrv
```

Run `setup-ds-admin.pl` (Accepting defaults should be ok IF you set up dns/hostname correctly)

Start on boot

```
chkconfig dirsrv on
chkconfig dirsrv-admin on
```

Run `centos-idm-console` from a X11 session

```
           User ID: cn=directory manager
           Password: <the directory manager password>
           Administration URL: 127.0.0.1:9830
```

Click: `ldap.example.org` > `Directory Server` > `Open`

  * Click "Directory" tab
  * Right-Click your Domain (in this case "example" for example.org) to add user/group
  * For the Group Right-Click group name (after you created it) and select "Advanced Properties"
     * Click on "Objet Class"
     * Click on "Add Value"
     * Scoll down and select "posixgroup" and click OK
     * This creates a new field called "gidnumber" -- go ahead and add the POSIX gid number there 

Create "top" level home directory (i.e. if it's `/rhome/username` then...):

`root@host# mkdir -m 755 /rhome`

Add this to `/etc/pam.d/login` and `/etc/pam.d/sshd` (also `/etc/pam.d/gdm` for GUI logins) For BOTH ldap server/client

```
	auth sufficient pam_ldap.so
	account sufficient pam_ldap.so
	password sufficient pam_ldap.so
	(you may also need "session sufficient pam_ldap.so" maybe?)
```

For the LDAP server do this:

```
	root@host# authconfig --enableldap --enableldapauth --enablemkhomedir --ldapserver=127.0.0.1 --ldapbasedn="dc=example,dc=org" --update
```

For the LDAP client do this:

```
root@host# authconfig --enableldap --enableldapauth --enablemkhomedir --ldapserver=192.168.1.248 --ldapbasedn="dc=example,dc=org" --update
```

Set up automount of homedirs

Edit `/etc/auto.master` and add...

```
 /rhome	/etc/auto.rhome	--timeout=60
```

Edit/create `/etc/auto.rhome`

```
*	-rw,intr,soft	netapp-gln:/vol/home/&
```

If using SELinux

```
restorecon -v /etc/auto.rhome
```

Restart Service

```
root@host# service autofs restart
root@host# chkconfig autofs on
```

## Linux CA certification commands

* Debian  
  `update-ca-certificates`
   The command will read certificates from files line by line `/etc/ca-certificates.conf` and read certification from location `/usr/share/ca-certificates/` then generate certification in `/etc/ssl/certs/`, and create a main crt into `/etc/ssl/certs/ca-certificates.crt`. only `.crt` file under `/usr/share/ca-certificates/` will be recognized. user need 
   
  `/etc/ssl/certs/ca-certificates.crt`  
  A single-file version of CA certificates.  This holds all CA certificates that  you activated in /etc/ca-certificates.conf
  
  Add CA  
  1. Copy your CA to dir /usr/local/share/ca-certificates/  
  2. Use command: sudo cp foo.crt /usr/local/share/ca-certificates/foo.crt
  3. Update the CA store:  update-ca-certificates  
  Remove CA
  1. Remove your CA.
  2. Update the CA store:  update-ca-certificates --fresh
  
* Redhat  
  `update-ca-trust`  
 
  1.Install: `yum install ca-certificates`
  2.Enable the dynamic CA configuration feature: `update-ca-trust force-enable`
  3.Add it as a new file to /etc/pki/ca-trust/source/anchors/: `cp foo.crt /etc/pki/ca-trust/source/anchors/`
  4.Use command: `update-ca-trust extract`
  
* Mac


## linux version
* Redhat based
  * RHEL: Redhat enterprise edition 
  * Centos: Redhat free edition, 1 release every two years. compiled using RHEL source code. 
  * Fedora: inherit from Redhat personal edition RHL. 1 release every half year, use new technology first. 
  * OEL: Oracle linux, 
* Debian based
  * Debian
  * Ubuntu: every 6 month
  * Mint
* Slackware based
  * SUSE
* Unix based 
  * Solaris: SunOS
  * AIX: Run in IBM Power



---
## linux service nslcd, nscd and sssd

`nslcd`: local LDAP name service daemon. daemon for NSS and PAM lookups using LDAP. enables to configure the local system to load users and groups from an LDAP directory (remote), which could be used
in the scenario that, we have a ldap server running in remote, and in local linux, by configure the nlscd, we could 
query/monitor the remote ldap server user change and created as local user. we could also configure this services to make
the linux system using ldap authentication. `nslcd` service is configure using `/etc/nlscd.conf`

`sssd`: system deamon service. provide access to local or remote identify and authentication resource. provide caching 
and offline support. could be configure to support SSO. 
SSSD does not create user accounts on the local system. Instead, it uses the identities from the external data store and lets
the users access the local system

[How sssd works](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/system-level_authentication_guide/sssd)

`nscd`: Name Service Cache Deamon. speed up consecutive access to NSS data and increase the overall performance. should be run 
at boot time by /etc/init.d/nscd

`nss`: name service switch. configure file `/etc/nsswitch.conf`, lets you configure how various types of names are resolved. names 
includes hostname, user names and so on. provides a certral configuration for services to look up a number of configuration and name
resolution services. 
[nss configuration](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/deployment_guide/configuration_options-nss_configuration_options)
database supported by NSS
```
aliases: Mail aliases.
ethers: Ethernet numbers.
group: Groups of users.
hosts: Host names and numbers.
netgroup: Network wide list of host and users.
network: Network names and numbers.
protocols: Network protocols.
passwd: User passwords.
rpc: Remote procedure call names and numbers.
services: Network services.
shadow: Shadow user passwords.
```

`pam`: plugable authentication module. 


`pam` and `nss` could be configure to do authenticate, by providing the `pam_ldap.so` and `nss_ldap.so`, if the ldap is in
the remote side, the lookup would be very expensive, here we could use the `sssd` daemon services, which could have local cache for the remote catalogs. 

![ssd architecure](https://github.com/litaocdl/docs/blob/master/pics/sssd.png)

[how pam ssd nss work together](https://community.hortonworks.com/articles/175124/how-pam-nss-ssd-work-together-on-linux-os.html)


## setup nfs in ubuntu and centos

Server and client could be in different os. 

ubuntu: https://vitux.com/install-nfs-server-and-client-on-ubuntu/



centos:https://www.howtoforge.com/tutorial/setting-up-an-nfs-server-and-client-on-centos-7/





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

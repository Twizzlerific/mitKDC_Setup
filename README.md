# MIT KDC Setup
Steps I use when I need to create my own Kerberos environment. 


# Setting up KDC
Install: 

```
yum install krb5-server krb5-libs krb5-workstation -y
```

Setup Realm:
```
# vi /etc/krb5.conf
[libdefaults]
  renew_lifetime = 7d
  forwardable = true
  default_realm = KS.COM
  ticket_lifetime = 24h
  dns_lookup_realm = false
  dns_lookup_kdc = false
  default_ccache_name = /tmp/krb5cc_%{uid}
  #default_tgs_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5
  #default_tkt_enctypes = aes des3-cbc-sha1 rc4 des-cbc-md5

[logging]
  default = FILE:/var/log/krb5kdc.log
  admin_server = FILE:/var/log/kadmind.log
  kdc = FILE:/var/log/krb5kdc.log

[realms]
  KS.COM = {
    admin_server = ldap.example.com
    kdc = ldap.example.com
  }

```

Create krbDB
```
kdb5_util create -s
```

Start KDC:
```
systemctl start krb5kdc
systemctl start kadmin
```

Check kdc.conf
```
vi /var/kerberos/krb5kdc/kdc.conf

! Change Realm to KS.COM
! Check enc types
```

Create Admin principal:
```
kadmin.local -q "addprinc admin/admin"
```

Give Admin permissions
```
vi /var/kerberos/krb5kdc/kadm5.acl
*/admin@KS.COM  *

! Restart
systemctl restart kadmin
```


# Creating Principals

Using the LDAP Server I set up in my [openLDAP_Setup](https://github.com/Twizzlerific/openLDAP_Setup) experiments, I create a series of principals.

> Note: This simply creates kerberos entities matching the users in the LDAP. It does not link them. So password changes on LDAP will not impact the password for kerberos.

__Gathering SPN from LDAP:__
```
ldapsearch -D cn=oneaboveall,dc=ks,dc=com -w yourdesiredpassword -b "dc=ks,dc=com" "(objectClass=posixAccount)" uid | egrep "^uid" | awk '{print $2}' > userSPN.txt
```

__Create based on LDAP princ__
```
for i in `cat userSPN.txt`; do kadmin.local -q "addprinc -pw marvel $i"; done
```

---
DiBtP ("Done is Better than Perfect!")
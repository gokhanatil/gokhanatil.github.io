---
title: "How To Retrieve Passwords From The Named Credentials in EM12c?"
layout: post
---

The username, password, and role name of the named credentials are stored in the em_nc_cred_columns table. When we examine it, we can see that there’s one-to-many relation with em_nc_creds using the target_guid column, and the sensitive information is stored in the cred_attr_value column. That column is encrypted using the em_crypto package. The encryption algorithm uses a secret key which is stored in the "Admin Credentials Wallet" and a salt (random data for additional security). The wallet file is located in: 

$MIDDLEWARE_HOME/gc_inst/em/EMGC_OMS1/sysman/config/adminCredsWallet/cwallet.sso

And the salt data can be found in the cred_salt column of the em_nc_cred_columns table. Here’s what it looks like:

![encrypted_credentials](/assets/encrypted_credentials.png)

<!--more-->

To decrypt the information, we need to call the decrypt in the em_crypto package, but if we call it without opening the wallet, we get the following error:

````
ORA-06512: at line 1
28239. 00000 -  "no key provided"
*Cause:    A NULL value was passed in as an encryption or decryption key.
*Action:   Provide a non-NULL value for the key.
````

How can we read the secret key from that wallet? The easiest way is, to make Enterprise Manager open the wallet and store the secret key in the repository database. So we issue the following command:

````
oracle@db-cloud ~$ /u02/Middleware/oms/bin/emctl config emkey -copy_to_repos
Oracle Enterprise Manager Cloud Control 12c Release 4
Copyright (c) 1996, 2014 Oracle Corporation.  All rights reserved.
Enter Enterprise Manager Root (SYSMAN) Password :
The EMKey has been copied to the Management Repository. 
This operation will cause the EMKey to become unsecure.
After the required operation has been completed, 
secure the EMKey by running "emctl config emkey -remove_from_repos".
````

It asks for the SYSMAN password. If you enter the correct password, it reads the wallet file and stores the secret key in the repository database. Of course, it makes your system insecure. If you issue the command “emctl config emkey -remove_from_repos”, you can remove the key from the repository.

If you issued the above command and stored the secret key in the repository, you can use the following query to fetch the decrypted information:

```sql
SELECT c.cred_owner,
c.cred_name,
c.target_type, 
(SELECT em_crypto.decrypt(p.cred_attr_value, p.cred_salt) 
FROM em_nc_cred_columns p WHERE c.cred_guid  = p.cred_guid 
AND lower(P.CRED_ATTR_NAME) LIKE '%user%') username,
(SELECT em_crypto.decrypt(p.cred_attr_value, p.cred_salt) 
FROM em_nc_cred_columns p WHERE c.cred_guid  = p.cred_guid 
AND lower(P.CRED_ATTR_NAME) LIKE '%role%') rolename,
(SELECT em_crypto.decrypt(p.cred_attr_value, p.cred_salt) 
FROM em_nc_cred_columns p WHERE c.cred_guid  = p.cred_guid 
AND lower(P.CRED_ATTR_NAME) LIKE '%password%') password
FROM em_nc_creds c
WHERE c.cred_owner <> '<SYSTEM>'
ORDER BY cred_owner;
```

Sample output:

![decrypted_credentials](/assets/decrypted_credentials.png)

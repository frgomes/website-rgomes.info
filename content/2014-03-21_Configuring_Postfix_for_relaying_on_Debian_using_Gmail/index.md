+++
title = "Configuring Postfix for relaying on Debian using Gmail"
date = 2014-03-21T13:47:00Z
[taxonomies]
categories = ["articles", "recipe"]
tags = ["linux", "configuration", "postfix", "relay", "explained"]
+++
_This article explains how Postfix can be installed and configured to route emails to an external SMTP server, in particular, using Gmail._

## Configuration

Run the script below as root.
```bash
#!/bin/bash

apt-get install postfix sasl2-bin bsd-mailx -y

##################################
# Choose:
#   * 'Satelite system'
#   * leave relayhost blank
##################################

cd /etc/postfix

cp -p main.cf main.cf.ORIGINAL

cat << EOD >> main.cf
smtp_use_tls = yes
smtp_sasl_auth_enable = yes
smtp_sasl_password_maps = hash:/etc/postfix/sasl_passwd
smtp_sasl_mechanism_filter = plain, login
smtp_sasl_security_options = noanonymous
EOD

# Substitute server, username and password below by your own settings
SERVER=smtp.gmail.com
USERNAME=your.username@gmail.com
PASSWORD=your.password

cat << EOD > sasl_passwd
${SERVER} ${USERNAME}:${PASSWORD}
EOD

chmod 400 sasl_passwd

postmap /etc/postfix/sasl_passwd

/etc/init.d/postfix restart
```

## Testing your configuration

As a regular user, try something like this:
```bash
#!/bin/bash

echo 'It works!' | mailx -s test me@mydomain.com
```


## References

* [Setup postfix to relay outbound mail using sasl](http://blog.taragana.com/index.php/archive/how-to-setup-postfix-to-relay-outbound-mail-using-sasl/)

----

If you found this article useful, it will be much appreciated if you create a link to this article somewhere in your website. 

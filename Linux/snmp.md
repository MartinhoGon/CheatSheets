# SNMP Cheat Sheet

## SNMP v2c

Change the snmp config by altering this file: "/etc/snmp/snmpd.conf", but first backup the original file:
```
# sudo cp /etc/snmp/snmpd.conf /etc/snmp/snmpd.conf.bak
```

Open the file, remove the contents and paste the config bellow. Do not forget to change the community string:
```
sysLocation    VM VMWare Esxi
sysContact     <Name> <mail@mail.com>

sysServices    72


master  agentx

agentaddress udp:161,udp6:[::1]:161


view   systemonly  included   .1.3.6.1.2.1.1
view   systemonly  included   .1.3.6.1.2.1.25.1


rocommunity  <community_string> default
rocommunity6 <community_string> default

```

Restart the snmp service by running the following command:
```
# sudo systemctl restart snmpd.service
```

## SNMP v3

First, stop the snmpd service
```
# sudo systemctl stop snmpd.service
```

It is necessary to create the user to authenticate snmp v3 clients:

```
# sudo net-snmp-create-v3-user -ro -A <authentication_password> -X <private_password> -a SHA -x AES <username>
```


Then, after user cration, we need to add the user to the snmp config file "/etc/snmp/snmpd.conf". Append the following line to the end of the file:

```
rouser <username> authpriv
```

Start the snmp service

```
# sudo systemctl start snmpd.service
```


## Testing

First it is necessary to install "snmpwalk"

```
# sudo apt install snmpwalk
```

To test snmp v2c

```
# snmpwalk -v 2c -c <community_string> <snmp_target>
```

To test snmp v3

```
# snmpwalk -v3 -l authpriv -u <username> -a SHA1 -x AES -A <auth_password> -X <priv_password> <snmp_target>
```


### Bibliografia:
https://www.cbtnuggets.com/blog/technology/networking/how-to-configure-snmpv3-and-how-it-works

https://www.incredigeek.com/home/setup-snmp-v3-on-debian-or-ubuntu/


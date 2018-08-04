# mikrotik-ddns-dyfi-update
Dynamic DNS update script for dy.fi

(Based on <a href="https://wiki.mikrotik.com/wiki/Dynamic_DNS_Update_Script_for_No-IP_DNS">Dynamic DNS Update Script for No-IP DNS</a>. Tested on **RouterOS v6.42.6**)

**1. Create a new script named _dyfi-update_**

The following permissions are required for this script to run:
* write
* test
* read

**2. Paste the code below as _Source_**
```
# dy.fi automatic Dynamic DNS update

##############Script Settings##################

:local DYFIUser "dy.fi LOGIN"
:local DYFIPass "dy.fi PASSWORD"
:local DYFIHost "dy.fi HOSTNAME"
:local WANInter "MikroTik Router WAN Interface Name"

###############################################

:local DYFIDomain "$DYFIHost.dy.fi"
:local IpCurrent [/ip address get [find interface=$WANInter] address];
:for i from=( [:len $IpCurrent] - 1) to=0 do={ 
  :if ( [:pick $IpCurrent $i] = "/") do={ 
    :local NewIP [:pick $IpCurrent 0 $i];
    :if ([:resolve $DYFIDomain] != $NewIP) do={
      /tool fetch mode=http user=$DYFIUser password=$DYFIPass url="http://www.dy.fi/nic/update\3Fhostname=$DYFIDomain&myip=$NewIP" keep-result=no
      :log info "dy.fi Update: $DYFIDomain - $NewIP"
     }
   } 
}
```
Edit the values for DYFIUser, DYFIPass DYFIHost & WANInter to match your configuration e.g.
```
##############Script Settings##################

:local NOIPUser "testuser1"
:local NOIPPass "testpassword"
:local NOIPPass "testhost"
:local WANInter "ether1"

###############################################
```

**3. Add a new scheduler entry to run the script every 6 days**
```
/system scheduler add comment="Update dy.fi DDNS" disabled=no interval="6d 00:00:00" \
name="dy.fi ddns update" on-event=dyfi-update policy=read,write,test
```

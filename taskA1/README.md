# Task A1 Write-up
```
"We believe that the attacker may have gained access to the victim's network by phishing a legitimate users credentials and connecting over the company's VPN. The FBI has obtained a copy of the company's VPN server log for the week in which the attack took place. Do any of the user accounts show unusual behavior which might indicate their credentials have been compromised?"
```

From this point we are given a link to a log file from an OpenVPN server instance with various logins:

```
openvpn-server,Jessica.R,2022.02.03 14:46:16 EDT,17172,VPN,0,1,172.16.142.204,10.128.20.250,UDP,1194,917173692,
openvpn-server,Ann.T,2022.02.03 15:22:54 EDT,11838,VPN,0,1,172.26.121.73,10.128.20.19,UDP,1194,1436186160,
openvpn-server,Brittany.J,2022.02.03 15:43:50 EDT,10702,VPN,0,1,172.22.11.173,10.128.20.89,UDP,1194,2270429300,
...
openvpn-server,Jessica.R,2022.02.05 17:10:29 EDT,4054,VPN,0,1,172.21.240.5,10.128.20.115,UDP,1194,71139592,
openvpn-server,Barbara.B,2022.02.05 17:38:28 EDT,3978,VPN,0,1,172.19.46.203,10.128.20.9,UDP,1194,369023148,
openvpn-server,Jacqueline.N,2022.02.05 18:03:16 EDT,2484,VPN,0,1,172.27.122.84,10.128.20.53,UDP,1194,296363556,
```

When analyzing this log, it helps to sort the data. Some suggested using a spreadsheet, however I just used `cut`, `paste`, and `grep` to sort and analyze entries.

Here are some basic stats about the file:

Amount of lines:
```
domlord in ~/hacking/codebreaker-2022/taskA1  wc -l vpn.log
     161 vpn.log
```

Amount of usernames present:
```
domlord in ~/hacking/codebreaker-2022/taskA1  cat vpn.log | cut -d',' -f2 | sort | uniq | wc -l
      45
```

So we have a decent chunk of information to sort through. There are also error lines that should be noted:
```
domlord in ~/hacking/codebreaker-2022/taskA1  cat vpn.log | grep not
openvpn-server,Patrick.J,2022.01.31 13:30:06 EDT,,VPN,0,0,172.24.72.80,,UDP,1194,,user not found
openvpn-server,Virginia.X,2022.01.31 14:21:55 EDT,,VPN,0,0,172.29.53.77,,UDP,1194,,user not found
openvpn-server,Cheryl.O,2022.01.31 14:44:21 EDT,,VPN,0,0,172.20.236.215,,UDP,1194,,user not found
openvpn-server,Katherine.X,2022.02.01 17:29:52 EDT,,VPN,0,0,172.31.217.30,,UDP,1194,,user not found
openvpn-server,Patrick.Z,2022.02.02 12:33:13 EDT,,VPN,0,0,172.27.236.115,,UDP,1194,,user not found
openvpn-server,Walter.T,2022.02.02 18:29:00 EDT,,VPN,0,0,172.20.73.88,,UDP,1194,,user not found
openvpn-server,Janet.Q,2022.02.03 16:27:31 EDT,,VPN,0,0,172.18.83.250,,UDP,1194,,user not found
openvpn-server,Janet.X,2022.02.03 18:03:40 EDT,,VPN,0,0,172.20.246.101,,UDP,1194,,user not found
openvpn-server,Michelle.T,2022.02.04 07:03:41 EDT,,VPN,0,0,172.23.90.165,,UDP,1194,,user not found
openvpn-server,Alexander.E,2022.02.04 07:42:24 EDT,,VPN,0,0,172.29.102.60,,UDP,1194,,user not found
openvpn-server,Louis.L,2022.02.04 12:29:24 EDT,,VPN,0,0,172.26.184.224,,UDP,1194,,user not found
openvpn-server,Walter.H,2022.02.04 17:00:45 EDT,,VPN,0,0,172.28.177.84,,UDP,1194,,user not found
openvpn-server,Stephen.X,2022.02.04 18:44:04 EDT,,VPN,0,0,172.30.176.222,,UDP,1194,,user not found
openvpn-server,Emily.R,2022.02.05 07:26:59 EDT,,VPN,0,0,172.22.217.211,,UDP,1194,,user not found
openvpn-server,Samuel.N,2022.02.05 08:27:49 EDT,,VPN,0,0,172.29.198.95,,UDP,1194,,user not found
openvpn-server,Scott.P,2022.02.05 09:00:47 EDT,,VPN,0,0,172.21.51.144,,UDP,1194,,user not found
openvpn-server,Randy.J,2022.02.05 15:16:30 EDT,,VPN,0,0,172.17.87.92,,UDP,1194,,user not found
openvpn-server,Evelyn.G,2022.02.05 16:16:05 EDT,,VPN,0,0,172.31.41.74,,UDP,1194,,user not found
```
and
```
openvpn-server,Justin.M,2022.01.31 13:10:10 EDT,,VPN,0,0,172.18.206.143,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Deborah.J,2022.01.31 17:58:23 EDT,,VPN,0,0,172.16.70.205,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Deborah.J,2022.02.01 09:00:46 EDT,,VPN,0,0,172.22.147.175,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Sean.X,2022.02.01 09:53:17 EDT,,VPN,0,0,172.17.76.205,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Samuel.Y,2022.02.01 18:18:16 EDT,,VPN,0,0,172.17.108.91,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Jessica.R,2022.02.02 08:38:01 EDT,,VPN,0,0,172.28.64.254,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Jacqueline.N,2022.02.02 09:48:38 EDT,,VPN,0,0,172.26.182.251,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Justin.M,2022.02.02 13:02:58 EDT,,VPN,0,0,172.29.123.169,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Patrick.O,2022.02.02 18:51:17 EDT,,VPN,0,0,172.19.180.232,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Albert.B,2022.02.03 11:37:51 EDT,,VPN,0,0,172.17.161.105,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Debra.V,2022.02.03 12:17:30 EDT,,VPN,0,0,172.28.125.207,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Brittany.J,2022.02.03 12:41:12 EDT,,VPN,0,0,172.24.155.34,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Ann.T,2022.02.04 11:29:26 EDT,,VPN,0,0,172.30.30.170,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Jacqueline.N,2022.02.04 15:43:25 EDT,,VPN,0,0,172.25.96.41,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Henry.C,2022.02.04 16:33:38 EDT,,VPN,0,0,172.26.173.218,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Charlotte.R,2022.02.04 17:34:22 EDT,,VPN,0,0,172.27.205.246,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Brittany.J,2022.02.05 10:44:44 EDT,,VPN,0,0,172.22.236.45,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
openvpn-server,Betty.H,2022.02.05 14:21:53 EDT,,VPN,0,0,172.20.107.85,,UDP,1194,,LDAP invalid credentials on ldap://10.128.30.20: LDAPInvalidCredentialsResult - 49 - invalidCredentials - None - None - bindResponse - None (facility='initialize [10.128.30.20]'
```

However, the first thing we should do is learn about each of the entries and what they mean. The header of the file is as follows:
```
domlord in ~/hacking/codebreaker-2022/taskA1  head -1 vpn.log
Node,Username,Start Time,Duration,Service,Active,Auth,Real Ip,Vpn Ip,Proto,Port,Bytes Total,Error
```

We see that we have a `Start Time` parameter in position 3 ( 2 if you start counting by 0 ;) ) and also a `Duration` parameter given in seconds.

To be sweet and to the point, you simply get a list of all names, and then for each name take the start time, convert it to a timestamp and add the duration, and then convert that to the end time of the session. Then analyze each users sessions to see if they are overlapping. This can be done by sorting a user's sessions by start time, and then taking a sessions End Time timestamp and subtracting the following sessions Start Time timestamp. If the following session started before the current session ended, then the timestamp will be less than the End Time timestamp of the current session.

Here is the solving shell script:
```
#!/bin/bash

cat vpn.log | sort | grep -v invalid | grep -v not | tail +2 | cut -d',' -f2 | uniq > valid_names.txt

if [ -d "analysis" ]
then
    rm -rf analysis
fi

mkdir analysis
for name in `cat valid_names.txt`; do
    cat vpn.log | grep $name | grep -v not | grep -v invalid | sort > analysis/$name.txt
done

# I'm on macOS, so the date command may be different if you're on linux:
# https://stackoverflow.com/questions/10990949/convert-date-time-string-to-epoch-in-bash

IFS=$'\n'
for file in `ls analysis`; do
    start_times=()
    end_times=()

    for line in `cat analysis/$file`; do
        start_timestamp=`echo -n $line | cut -d',' -f3 | sed -e 's/EDT/UTC/' | xargs -0 date -juf "%Y.%m.%d %H:%M:%S %Z%n" +"%s"`
        end_timestamp=$(($start_times  tamp + `echo -n $line | cut -d',' -f4`))
        start_times+=( $start_timestamp )
        end_times+=( $end_timestamp )
    done

    # extra value to handle out-of-bounds access, mainly due to laziness
    start_times+=( 2000000000 )

    for i in ${!end_times[@]}; do
        if [ $((${end_times[i]}-${start_times[i+1]} )) -gt 0 ]
        then
            echo "${file} has suspicious session times" | sed -e 's/\.txt//g'
        fi
    done
done
```


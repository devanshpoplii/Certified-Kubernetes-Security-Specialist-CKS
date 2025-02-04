## Minimize external access to the network

In Linux, every network service listens on a specific port for incoming connections. When a process binds to a port, it reserves that port so it can send or receive data.

#### About netstat and it's Usage

netstat (short for **network statistics**) is a utilitiy for displaying detailed network information. It shows active connections, listening ports, and which services are using them.

**Show all listening ports:**
```bash
netstat -tulnp
```
- -t → Show TCP ports
- -u → Show UDP ports
- -l → Show only listening ports
- -n → Show numerical IP addresses (skip DNS lookup)
- -p → Show the process (PID) using the port

The output of the above command is as follows:
```
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:40200           0.0.0.0:*               LISTEN      1168/kc-terminal    
tcp        0      0 0.0.0.0:40205           0.0.0.0:*               LISTEN      1139/node           
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      34527/nginx: master 
tcp        0      0 127.0.0.1:36819         0.0.0.0:*               LISTEN      577/containerd      
tcp        0      0 127.0.0.53:53           0.0.0.0:*               LISTEN      16681/systemd-resol 
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      16400/sshd: /usr/sb 
tcp6       0      0 :::40300                :::*                    LISTEN      1105/runtime-scenar 
tcp6       0      0 :::80                   :::*                    LISTEN      34527/nginx: master 
tcp6       0      0 :::40305                :::*                    LISTEN      1154/runtime-info-s 
tcp6       0      0 :::22                   :::*                    LISTEN      16400/sshd: /usr/sb 
udp        0      0 127.0.0.53:53           0.0.0.0:*                           16681/systemd-resol 
udp        0      0 172.30.1.2:68           0.0.0.0:*                           16658/systemd-netwo 
udp        0      0 0.0.0.0:68              0.0.0.0:*                           797/dhclient        
udp        0      0 0.0.0.0:68              0.0.0.0:*                           749/dhclient        
udp        0      0 0.0.0.0:68              0.0.0.0:*                           639/dhclient
```

**Another netstat command to view open ports:**
```
netstat -anp | grep -w LISTEN
```
- -a → Show all sockets (listening and non-listening)

Note: `netstat -tulnp` would also return the same output as above on adding `| grep -w LISTEN`. The difference between `-anp` and `-tulnp` is that the later also shows udp connections.

Linux maintains a list of well-known ports and thier associated services `/etc/services`. You can search for the service associated with a port like this:
```bash
cat /etc/services/ | grep 22
```

---

### UFW (Uncomplicated Firewall)

**UFW** is a user-friendly command-line tool for managing **iptables** (default firewall in Linux).

To check if UFW is installed:
```bash
ufw status
```
If you get an output like "**Status: inactive**", UFW is installed but not active. 

If it's not installed, you can install it:
```bash
apt update && apt install ufw -y
```

#### Enable and Disable UFW
- **Enable the firewall** (default policy to deny incoming traffic):
```bash
ufw enable
```
The status becomes `Status: active`.
- **Disable the firewall:**
```bash
ufw disable
```
- Reset UFW (removes all rules and resets to default):
```bash
ufw reset
```

#### Default Firwall Policies
UFW applies default rules if no specific allow/deny rules exist.

To **deny all incoming traffic and allow outgoing traffic** (recommended default policy):
```bash
ufw default deny incoming
ufw default allow outgoing
```

#### Viewing Rules
To see **enabled rules and their status:**
```bash
ufw status verbose
```
Example Output:
```
Status: active
Logging: on (low)
Default: deny (incoming), allow (outgoing), deny (routed)
New profiles: skip
```

#### Adding Rules (Allow/Deny) on Port

To allow a specific IP (example: 192.169.1.100) to access SSH (port 22):
```bash
ufw allow from 192.168.1.100 to any port 22
```

To allow a CIDR range to access SSH (port 22):
```bash
ufw allow from 192.168.1.0/24 to any port 22
```

Mention protocol in the command:
```bash
ufw allow from 192.168.1.0/24 to any port 22 proto tcp
```

Add a deny rule:
```bash
ufw deny 8080
```

To allow a range of TCP ports, use:
```
ufw allow 5000:6000/tcp
```


#### Delete a Rule
If there is an existing rule that denies SSH, remove it via following:

Check existing rukes:
```bash
ufw status numbered
```
This shows:
```
Status: active

     To                         Action      From
     --                         ------      ----
[ 1] 8080                       DENY IN     Anywhere                  
[ 2] 8080 (v6)                  DENY IN     Anywhere (v6)
```

Delete the deny rule using number:
```bash
ufw delete 1
```

Delete the deny rule using port:
```bash
ufw delete deny 22/tcp
```

# How to Setup DNS Server on CentOS
<img width="895" height="1082" alt="image" src="https://github.com/user-attachments/assets/10fd366c-c097-4969-b727-e06cdebe7fe8" />

## 1️⃣. Install BIND (DNS Server Software)
Update packages:
```
sudo yum update -y
```
Install BIND and related tools:
```
sudo yum install bind bind-utils -y
```
Enable and start the service:
```
sudo systemctl enable named
sudo systemctl start named
```
Check status:
```
sudo systemctl status named
```
## 2️⃣ Configure BIND Primary Zone
Edit the main configuration file:
```
sudo nano /etc/named.conf
```
Change listen-on and allow-query so your server can answer DNS queries:
```

options {
    listen-on port 53 { any;192.168.1.2; };
    listen-on-v6 port 53 { any; };
    allow-query     { any;192.168.1.0/24; };

    recursion yes;
};
```
Save and exit.
##  3️⃣ Define Your DNS Zone
```
sudo nano /etc/named.rfc1912.zones
```
Add your domain zone (example: example.com):
```

zone "example.com" IN {
    type master;
    file "example.com.zone";
    allow-update { none; };
};
```
## 4️⃣ Create Zone File
Create the zone file in /var/named/:
```
sudo nano /var/named/example.com.zone
```
Paste the template below (modify domain & IP):
```
$TTL 86400
@   IN  SOA ns1.example.com. admin.example.com. (
        2024020101  ; Serial
        3600        ; Refresh
        1800        ; Retry
        604800      ; Expire
        86400 )     ; Minimum TTL

; Name servers
@       IN  NS      ns1.example.com.

; A records
ns1     IN  A       192.168.1.2
@       IN  A       192.168.1.2
www     IN  A       192.168.1.2
```
Set correct permissions:
```

sudo chown root:named /var/named/example.com.zone
sudo chmod 640 /var/named/example.com.zone
```
## 5️⃣ (Optional) Create Reverse DNS Zone
Edit configuration:
```
sudo nano /etc/named.rfc1912.zones
```
Add reverse zone (example for 192.168.1.x):
```

zone "1.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.1.rev";
    allow-update { none; };
};
```
Create reverse zone file:
```
sudo nano /var/named/192.168.1.rev
```
Example:
```
$TTL 86400
@   IN  SOA ns1.example.com. admin.example.com. (
        2024020101
        3600
        1800
        604800
        86400 )

@     IN  NS    ns1.example.com.
2    IN  PTR   example.com.
```
## 6️⃣ Check Configuration for Errors
Check named.conf:
```
sudo named-checkconf
```
Check your zone files:
```

sudo named-checkzone example.com /var/named/example.com.zone
sudo named-checkzone 1.168.192.in-addr.arpa /var/named/192.168.1.rev
```
If both say OK, continue.
## 7️⃣ Restart BIND
```
sudo systemctl restart named
```
Allow DNS through firewall:
```

sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --reload
```
## 8️⃣ Test DNS
Test A record:
```
dig @192.168.1.2 example.com
```
Test www:
```
dig @192.168.1.2 www.example.com
```
Test reverse lookup:
```
dig @192.168.1.2 -x 192.168.1.2
```
If you see ANSWER SECTION → DNS is working.

#  How to Configure DNS Slave Server

##  1. Install BIND on Slave Server
```
sudo yum install bind bind-utils -y
```
Enable and start it:
```

sudo systemctl enable named
sudo systemctl start named
```
## 2. Configure the Slave DNS Zone
Edit zone configuration:
```
sudo nano /etc/named.rfc1912.zones
```
Add:
```

zone "example.com" IN {
    type slave;
    masters { 192.168.1.2; };     // IP of Master DNS
    file "slaves/example.com.zone";
};
```
Create the slave directory:
```

sudo mkdir -p /var/named/slaves
sudo chown named:named /var/named/slaves
```
## IMPORTANT
The Master DNS must allow zone transfers to this slave IP.
Otherwise the slave will fail to load the zone.

## 3. Configure the Master DNS to Allow Zone Transfer
On the Master server, edit:
```
sudo nano /etc/named.rfc1912.zones
```
Modify master zone like this:
```
zone "example.com" IN {
    type master;
    file "example.com.zone";
    allow-transfer { 192.168.1.20; };   // Slave DNS IP
    also-notify { 192.168.1.20; };      // Optional: push changes
};
```
Restart master DNS:
```
sudo systemctl restart named
```
## 4. Allow DNS Traffic Through Firewall (On Slave)
```

sudo firewall-cmd --add-service=dns --permanent
sudo firewall-cmd --reload
```
Allow zone‑transfer port TCP 53:
```

sudo firewall-cmd --add-port=53/tcp --permanent
sudo firewall-cmd --reload
```
##  5. Restart BIND on Slave Server
```
sudo systemctl restart named
```
Check logs to confirm zone transfer:
```
sudo tail -f /var/log/message
```
You should see something like:
```
transfer of 'example.com' from 192.168.1.2#53: Transfer completed
```
Zone file should appear in:
```
/var/named/slaves/example.com.zone
```
##  6. Test DNS Slave Functionality
Test query from slave:
```
dig @192.168.1.20 example.com
```
Check zone file exists:
```
ls -l /var/named/slaves/
```
Test slave responds properly:
```
dig @192.168.1.20 www.example.com
```
If you see ANSWER SECTION, the slave DNS is fully working.

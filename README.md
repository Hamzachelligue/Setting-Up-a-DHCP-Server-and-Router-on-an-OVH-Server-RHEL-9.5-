# Setting Up a DHCP Server and Router on an OVH Server (RHEL 9.5)

## Overview
This guide will show you how to set up a **DHCP server** on an **OVH server running RHEL 9.5** and configure it as a **router** to provide internet access to client machines.

## Prerequisites
- A **dedicated or virtual server** at OVH running **RHEL 9.5**.
- Two network interfaces:
  - **eth0** (External - Connected to the internet)
  - **eth1** (Internal - Connected to LAN clients)
- **Root or sudo access** to the server.

---

## Step 1: Install the DHCP Server
Run the following command to install the DHCP server package:

```bash
sudo dnf install -y dhcp-server
```

Once installed, we need to configure the **DHCP service**.

---

## Step 2: Configure the DHCP Server
Edit the **DHCP configuration file**:

```bash
sudo vi /etc/dhcp/dhcpd.conf
```

Use the following example configuration (adjust the network range as needed):

```conf
option domain-name "local";
option domain-name-servers 8.8.8.8, 8.8.4.4;
default-lease-time 600;
max-lease-time 7200;

subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;
    option routers 192.168.1.1;
    option broadcast-address 192.168.1.255;
}
```

Save and exit the file.

---

## Step 3: Enable and Start the DHCP Service
Enable and start the **DHCP service**:

```bash
sudo systemctl enable --now dhcpd
```

Check if it's running:

```bash
sudo systemctl status dhcpd
```

---

## Step 4: Enable IP Forwarding
To allow your OVH server to act as a router, enable **IP forwarding**:

```bash
sudo vi /etc/sysctl.conf
```

Add or modify the following line:

```conf
net.ipv4.ip_forward = 1
```

Apply the change:

```bash
sudo sysctl -p
```

---

## Step 5: Configure NAT with iptables
To allow DHCP clients to access the internet, set up **NAT** (Network Address Translation):

```bash
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth1 -o eth0 -j ACCEPT
sudo iptables -A FORWARD -i eth0 -o eth1 -m state --state RELATED,ESTABLISHED -j ACCEPT
```

Save the rules:

```bash
sudo iptables-save > /etc/sysconfig/iptables
```

To ensure NAT rules persist after a reboot, install `iptables-services`:

```bash
sudo dnf install -y iptables-services
sudo systemctl enable iptables
```

---

## Step 6: Configure Firewall Rules
Allow **DHCP** and enable **masquerading**:

```bash
sudo firewall-cmd --permanent --add-service=dhcp
sudo firewall-cmd --permanent --add-masquerade
sudo firewall-cmd --reload
```

---

## Step 7: Verify the Setup
### Check DHCP Leases
Run the following command to verify that clients are getting IP addresses:

```bash
sudo cat /var/lib/dhcpd/dhcpd.leases
```

### Test Routing
1. **Connect a client to eth1.**
2. Run `ip a` to check if it received an IP address.
3. Try to ping `8.8.8.8` to confirm internet connectivity.

```bash
ping -c 4 8.8.8.8
```

If the ping is successful, your **DHCP server and router are working!** 🎉

---

## Conclusion
You have now successfully set up a **DHCP server** and **router** on an OVH **RHEL 9.5** server. Your clients should receive IP addresses and have internet access via your OVH server.

If you need additional configurations, such as **multiple subnets** or **PXE boot**, feel free to modify the settings accordingly. 🚀

**Need help?** Feel free to open an issue or contribute to this guide!


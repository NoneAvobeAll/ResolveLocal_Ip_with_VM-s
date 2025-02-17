# Accessing VMs Across Different Subnets

**Scenario**:  
- Host machine (local PC) is on the `192.168.1.x` subnet.  
- Virtual Machines (VMs) are on the `172.16.245.x` subnet.  
- Goal: Allow devices on `192.168.1.x` to access applications running on the VMs.

---

## Solutions

### Option 1: Change VM Network Mode to Bridged
Place VMs on the same subnet as your host (`192.168.1.x`).  

**Steps**:  
1. **Virtualization Software** (e.g., VirtualBox/VMware):  
   - Open VM settings → Network → Change adapter mode to **Bridged**.  
   - Restart the VM.  
2. **Assign IP Address**:  
   - Let the VM obtain a `192.168.1.x` IP via DHCP or set it manually.  
3. **Verify**:  
   - Check VM IP (e.g., `ipconfig` on Windows, `ifconfig` on Linux).  
   - Ping the host (`192.168.1.x`) from the VM and vice versa.  

**Result**:  
- Access VMs directly via their new `192.168.1.x` IP from other devices.

---

### Option 2: Route Traffic Between Subnets
Keep VMs on `172.16.245.x` but enable routing via the host.  

**Steps**:  
1. **Enable IP Forwarding on Host**:  
   - **Linux**:  
     ```bash
     sudo sysctl -w net.ipv4.ip_forward=1
     ```  
   - **Windows**:  
     - Set `HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters\IPEnableRouter` to `1` (via `regedit`).  
     - Reboot.  

2. **Add Static Route to Router**:  
   - Destination: `172.16.245.0`  
   - Subnet Mask: `255.255.255.0`  
   - Gateway: `192.168.1.x` (host’s IP).  

3. **Configure VM Gateway**:  
   - Set the VM’s default gateway to the host’s virtual interface IP (e.g., `172.16.245.1`).  

**Result**:  
- Devices on `192.168.1.x` can now reach `172.16.245.x` VMs.

---

### Option 3: Port Forwarding on Host
Forward specific ports from the host to VMs.  

**Steps**:  
1. **Linux Host**:  
   ```bash
   sudo iptables -t nat -A PREROUTING -p tcp --dport 80 -j DNAT --to-destination 172.16.245.3:80
   sudo iptables -t nat -A PREROUTING -p tcp --dport 443 -j DNAT --to-destination 172.16.245.3:443
   ```  
2. **Windows Host**:  
   ```cmd
   netsh interface portproxy add v4tov4 listenport=80 listenaddress=192.168.1.x connectport=80 connectaddress=172.16.245.3
   netsh interface portproxy add v4tov4 listenport=443 listenaddress=192.168.1.x connectport=443 connectaddress=172.16.245.3
   ```  

**Result**:  
- Access VMs via `http://HOST_IP:PORT` (e.g., `http://192.168.1.5:80`).

---

### Option 4: VPN Setup
Securely tunnel traffic to the `172.16.245.x` subnet.  

**Steps**:  
1. Install a VPN server (e.g., [WireGuard](https://www.wireguard.com/), [OpenVPN](https://openvpn.net/)) on the host or a VM.  
2. Configure the VPN to route traffic to `172.16.245.x`.  
3. Connect devices to the VPN.  

**Result**:  
- Devices on the VPN can access VMs via their `172.16.245.x` IPs.

---

## Summary Table
| Option              | Complexity | Use Case                                  |
|---------------------|------------|------------------------------------------|
| **Bridged Mode**    | Low        | Simplify access by placing VMs on the same subnet. |
| **Routing**         | High       | Keep VMs isolated but allow cross-subnet access. |
| **Port Forwarding** | Medium     | Expose specific VM ports to the network. |
| **VPN**             | Medium     | Secure remote access to VMs.             |

---

📝 **Notes**:  
- Replace `192.168.1.x`/`172.16.245.x` with your actual IPs.  
- Adjust firewall rules on VMs to allow incoming traffic.  


License: [CC0-1.0](https://creativecommons.org/publicdomain/zero/1.0/) (Free to use/modify).
```
Author: Abubakkar Khan Fazla Rabbi
Sysops

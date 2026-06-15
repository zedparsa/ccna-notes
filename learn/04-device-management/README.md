## 📖 Part 1 — Password Recovery (Router & Switch)

### 📝 Summary:
Password recovery on Cisco devices is a **physical-access** procedure.  
The common idea is to boot the device **without loading the startup configuration**, so you can regain access, restore the original configuration, and then set new credentials.

This guide covers:
- Router password recovery using **ROMMON** and the **configuration register**
- Switch password recovery using the **boot loader** (switch: prompt) and renaming config.text

### 🎯 Objectives:
- Understand why password recovery requires physical access (console + power cycle).
- Explain what the configuration register does (especially (0x2102) vs (0x2142)).
- Perform password recovery on a router using ROMMON.
- Perform password recovery on a Catalyst switch (e.g., 2960) using the boot loader.
- Restore the original configuration safely and reset new credentials.
- Understand the risk of no service password-recovery.

### 🧩 Topology:
- 1x Router (console access)
- 1x Switch (console access)
- 1x PC/Laptop (terminal emulator: Packet Tracer PC, PuTTY, Tera Term, etc.)
- Console cable for each device (or direct console in Packet Tracer)

### 🛠️ Step-by-Step:

### 🔹 A) Router Password Recovery (ROMMON)

#### ✅ Concept
Routers can enter **ROMMON** (like a low-level recovery mode).  
From ROMMON, you can temporarily change the boot behavior using the **configuration register**.

- Normal boot (loads startup-config): (0x2102)
- Ignore startup-config during boot (recovery mode): (0x2142)

Important correction:
- (0x2142) means: “Boot IOS normally, but **do not load startup-config** into running-config.”
- It does **not** mean “run running-config.” Running-config only exists after IOS boots (in RAM).

#### 1) Enter ROMMON
1. Power off the router.
2. Power on the router.
3. During early boot, send the **Break** sequence (varies by terminal):
   - Packet Tracer: usually a “Break” button/option in the terminal window
   - Real terminal apps: may be `Ctrl+Break` , `Ctrl+Fn+B `, etc.
4. You should land in ROMMON prompt, often like:
   - `rommon 1 >`

#### 2) Set the recovery configuration register and boot
```
rommon 1 > confreg 0x2142  
rommon 2 > reset  
```
(Some platforms use confreg interactively; the goal is still to set (0x2142).)

#### 3) Boot IOS without startup-config
After IOS loads, you may see the initial configuration dialog:
- `Answer: no`

Now you can enter privileged mode (because the startup-config was ignored):
```
Router> enable
```
#### 4) Restore the original config into RAM
This is the critical step to avoid losing IP addressing/routing settings.
```
Router# copy startup-config running-config
```
#### 5) Change credentials (recommended approach)
Do **not** use no username without specifying a username (that’s not a valid generic command in IOS).

Set or replace a local admin user:
```
Router# configure terminal  
Router(config)# username Parsa secret 4444  
Router(config)# enable secret 123  
Router(config)# end  
```
#### 6) Set config-register back to normal and save
Verify the current register:
```
Router# show version
```
If you see Configuration register is 0x2142, change it back:
```
Router# configure terminal  
Router(config)# config-register 0x2102  
Router(config)# end  
Router# write memory  
```
#### 7) Reload and test
```
Router# reload
```
After reload, confirm you can:
- SSH/console login with your user
- Enter privileged EXEC with enable

---

### 🔹 B) Switch Password Recovery (Boot Loader)

#### ✅ Concept
On many Catalyst switches (including common CCNA models), the startup configuration is stored as a file in flash:
- flash:config.text (startup configuration)
- Running-config lives in RAM

For password recovery, we temporarily rename config.text so the switch boots **without** it, then we load it back into RAM and change passwords.

#### 1) Enter switch boot loader (switch: prompt)
1. Power off the switch.
2. Press and hold the **MODE** button.
3. Power on the switch while holding MODE.
4. Release MODE when the LED behavior indicates boot loader mode (varies by model).
5. You should see:
- switch: prompt

#### 2) Initialize flash and rename the config
```
switch: flash_init  
switch: dir flash:  
switch: rename flash:config.text flash:config.text.old  
switch: boot  
```
#### 3) Boot without startup-config
The switch boots with a “clean” configuration. You may see the setup dialog:
- `Answer: no`

Enter privileged mode:
```
Switch> enable
```
#### 4) Load the original config into running-config (RAM)
Use the old file you renamed:
```
Switch# copy flash:config.text.old running-config
```
#### 5) Set new passwords/users
```
Switch# configure terminal  
Switch(config)# username Parsa secret 4444  
Switch(config)# enable secret 123  
Switch(config)# line console 0  
Switch(config-line)# login local  
Switch(config-line)# exit  
Switch(config)# line vty 0 15  
Switch(config-line)# login local  
Switch(config-line)# transport input ssh  
Switch(config-line)# end  
```
#### 6) Save and clean up
Save your new working config as the new startup-config:
```
Switch# write memory
```
Delete the old backup file:
```
Switch# delete flash:config.text.old
```
---

### 🧾 Command Caption Table

| Command | Where | Purpose |
|---|---|---|
| `confreg 0x2142` | Router ROMMON | Boot IOS but ignore startup-config (recovery mode) |
| `reset` | Router ROMMON | Reboot device using the new register value |
| `copy startup-config running-config` | Router IOS | Restore original configuration into RAM |
| `config-register 0x2102` | Router IOS | Return to normal boot behavior |
| `flash_init` | Switch boot loader | Initialize flash filesystem |
| `rename flash:config.text flash:config.text.old` | Switch boot loader | Prevent loading startup-config by hiding the file |
| `copy flash:config.text.old running-config` | Switch IOS | Load old config into RAM so you don’t lose settings |
| `write memory` | Router/Switch IOS | Save running-config as startup-config |

### ✅ Verification:
Router:
```
show version  
show running-config | include username|enable secret  
show running-config | section line  
```
Switch:
```
dir flash:  
show running-config | include username|enable secret  
show running-config | section line  
```
---
---
---
## 📖 Part 2 — IOS (OS) Recovery via TFTP (ROMMON)

### 📝 Summary:
IOS (operating system) recovery is used when a router cannot boot because the IOS image is missing, corrupted, or the device is stuck in ROMMON. A common recovery method is downloading an IOS image from a TFTP server over the network, then booting from that image (and often saving it to flash).

This guide covers:
- Preparing the Layer 2/Layer 3 basics so the router can reach the TFTP server
- Using ROMMON environment variables for TFTP download
- Running the ROMMON TFTP download process safely

### 🎯 Objectives:
- Understand why IOS recovery is typically a physical-access procedure (console + ROMMON).
- Build a basic recovery topology with a switch, a TFTP server, and a router in ROMMON.
- Configure the switch management SVI so you can test connectivity and manage the lab.
- Explain why PortFast can help in recovery labs.
- Set ROMMON IP parameters correctly (router IP, mask, gateway if needed, TFTP server IP, filename).
- Download the IOS image using tftpdnld and boot the router.

### 🧩 Topology:
- 1x Router in ROMMON (IOS missing/corrupted)
- 1x Switch (all devices connected to the switch)
- 1x Server (TFTP server)
- 1x PC/Laptop (console/terminal access to the router)

Example addressing (recommended to avoid IP conflicts):
- Router (ROMMON IP_ADDRESS): `192.168.1.1/24`
- Switch (VLAN 1 SVI): `192.168.1.2/24`
- TFTP Server: `192.168.1.100/24`
- Default gateway: not required if everything is in the same /24

### 🛠️ Step-by-Step:

### 🔹 A) Prepare the Switch (basic management + faster link-up)

#### 1) Configure VLAN 1 SVI (management IP)
    Switch> enable
    Switch# configure terminal
    Switch(config)# interface vlan 1
    Switch(config-if)# ip address 192.168.1.2 255.255.255.0
    Switch(config-if)# no shutdown
    Switch(config-if)# exit

Optional (only if you want the switch to reach other networks):

    Switch(config)# ip default-gateway 192.168.1.1

#### 2) Enable PortFast on access ports connected to end devices
Apply PortFast on ports that connect to the server/PC (and optionally to the router if you are sure there is no switching loop).

Example:

    Switch(config)# interface gigabitEthernet 0/1
    Switch(config-if)# spanning-tree portfast
    Switch(config-if)# exit

Why PortFast?
- Normal STP can delay a port from forwarding for roughly 30 seconds while it transitions states
- In recovery labs, you want the link to start forwarding immediately so TFTP traffic can work right away
- PortFast should be used on ports that connect to end devices, not on switch-to-switch links

### 🔹 B) Prepare the TFTP Server
On the server:
- Set the IP address to 192.168.1.100/24
- Enable the TFTP service
- Place the IOS image file in the TFTP root directory (the filename must match exactly)

### 🔹 C) Router IOS Recovery (ROMMON + TFTP)

#### ✅ Concept
In ROMMON, the router has no IOS, so you must provide temporary IP settings directly in ROMMON variables. The router will use these values to reach the TFTP server and download the IOS image.

Important correction:
- `IP_ADDRESS` must be the router interface IP address used for TFTP (not the switch SVI IP)
- `DEFAULT_GATEWAY` is only needed if the TFTP server is in a different subnet

#### 1) Enter ROMMON
- Power cycle the router
- Send Break during boot (or use Packet Tracer break option) to reach the ROMMON prompt

You should see something like:
```
rommon 1 >
```
#### 2) Set ROMMON network parameters
Example (same subnet, no gateway needed):

    rommon 1 > IP_ADDRESS=192.168.1.1
    rommon 2 > IP_SUBNET_MASK=255.255.255.0
    rommon 3 > TFTP_SERVER=192.168.1.100
    rommon 4 > TFTP_FILE=c2900-universalk9-mz.SPA.155-3.M4a.bin

If your TFTP server is in another subnet, add a gateway:

    rommon X > DEFAULT_GATEWAY=192.168.1.254

To verify what you set:

    rommon X > set

#### 3) Run the TFTP download
Start the download process:

    rommon X > tftpdnld

Note:
- On many platforms, tftpdnld can overwrite flash contents and then boot from the new image
- Read the ROMMON prompt carefully and confirm only if you are sure the filename and server IP are correct

#### 4) Boot IOS and continue normal configuration
After a successful download and boot:
- Verify the image and interfaces
- Save configuration as needed

---

### 🧾 Command Caption Table

| Command / Setting | Where | Purpose |
|---|---|---|
| `interface vlan 1`, `ip address`, `no shutdown` | Switch IOS | Give the switch a management IP for testing/management |
| `spanning-tree portfast` | Switch IOS | Make the port forward immediately, avoiding STP startup delay |
| `IP_ADDRESS` | Router ROMMON | Sets the router IP used for the TFTP recovery |
| `IP_SUBNET_MASK` | Router ROMMON | Sets the subnet mask for the router ROMMON interface |
| `DEFAULT_GATEWAY` | Router ROMMON | Sets gateway for reaching a TFTP server in another subnet (optional) |
| `TFTP_SERVER` | Router ROMMON | Sets the TFTP server IP address |
| `TFTP_FILE` | Router ROMMON | Sets the IOS image filename to download (must match exactly) |
| `set` | Router ROMMON | Displays current ROMMON environment variables |
| `tftpdnld` | Router ROMMON | Downloads the IOS image from TFTP (often writes to flash and boots) |

### ✅ Verification:
Switch:

    show ip interface brief
    ping 192.168.1.100

Router (after IOS boots):

    show version
    show ip interface brief
    dir flash:

End-to-end:
- Confirm the router successfully boots into IOS (not ROMMON)
- Confirm you can reach the router via console and configure it normally

### ⚠️ Note:
- IOS recovery requires physical access because ROMMON access typically requires console + reboot/break.
- Keep all recovery devices in the same subnet to avoid needing a gateway during ROMMON recovery.
- If tftpdnld fails, the most common causes are wrong IP settings, wrong filename, TFTP service disabled, or the switch port not forwarding yet (PortFast helps).
- Always double-check the IOS filename spelling and case, because mismatches cause silent failures.

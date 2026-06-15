# 🔐 Port security — Tutorial
- When you buy a Cisco device like a switch or router, 
it comes with a basic configuration and no security enabled by default. 
- You are responsible for securing it. 
- How? Stay with me and we will figure it out together.
## 📖 Part 1 — Console Password (Type 7):
### 📝 Summary:
Set a basic console line password to secure device access (weak, Type 7 encryption).
### 🎯 Objective
- Understand the need for securing console access on Cisco devices.  
- Configure a basic console line password using `line console 0`.  
- Verify console login with the configured password.  
- Be aware of Type 7 password weakness.  

### 🧩 Topology
 
<p align="center">
  <img src="topologies/port-security-1.png" alt="Port Security Topology" />
</p>

- One PC and one Switch 
- One PC connected to one Switch via a console cable (RS-232 → Console)

### 🛠️ Step-by-Step
Use the following commands to configure a console password :
  ```cisco
  Switch> enable
  Switch# configure terminal
  Switch(config)# line console 0
  Switch(config-line)# password 123
  Switch(config-line)# login
  Switch(config-line)# end
```
<p align="center">

| Command              | Description                         |
|----------------------|-------------------------------------|
| `line console 0 `      | Enter console line configuration    |
| `password 123`         | Set console password (123)          |
| `login`                | Enable password checking on console |
| `end` / `Ctrl+Z`         | Exit to privileged EXEC mode        |

</p>

### 🔑 Extra: Securing with `service password-encryption`

Another method is to use the command `service password-encryption`.  
This command hides all plain-text passwords in the running configuration by converting them into Type 7.  

If you add the command service password-encryption in global configuration: 
```cisco
Switch(config)# service password-encryption
```
Then in show running-config the password will appear encrypted (Type 7):
```cisco
line console 0
  password 7 0822455D0A16
  login
```

And if you don't add :
```cisco
line console 0
  password 123
  login
```

### 🔑 Extra: Removing Encryption

If you want to remove the encryption applied by `service password-encryption`,  
use the command:

```cisco
Switch(config)# no service password-encryption
```
Now, when you check the running configuration again:
- The command **does not actually decrypt stored values**;
it simply stops encrypting **new or updated passwords**.
- Already encrypted ones may remain until reconfigured
### ✅ Verification:
- Exit the console session (close terminal).
- Reconnect to the switch via console.
- You should now be prompted for a password before accessing User EXEC mode.

### ⚠️ Note:
This method is weak because:
> - Type 7 encryption is easily reversible.  
> - Only one shared password for all users.  
> - No usernames, so no individual accountability.  
> - Works only on console line (no centralized authentication).  

So what should we do? 🤔  
- In the next part you will get the answer.  
- For now, just practice this method to understand the basics.


---
---
---
## 📖 Part 2 — Local Authentication:


### 📝 Summary:
Using only passwords can lead to many problems. For example, if we have three users who need to log in to a Cisco device with different access levels, we must create **three separate users**.
Another problem that we may face is that if there are issues in the switch configuration, we won’t know who caused them because multiple users share access.
This can be solved and accomplished using **local authentication**.

### 🎯 Objectives:
- Understand why simple console passwords are insufficient when multiple users access a device.
- Learn to create multiple local user accounts with different privilege levels.
- Configure local authentication on a Cisco switch or router.
- Verify which user is logged in and track user activity.
- Recognize the benefits of using local authentication over a single shared password.

### 🧩 Topology:

<p align="center">
  <img src="topologies/port-security-1.png" alt="Port Security Topology" />
</p>

>  Agian
- One PC and one Switch 
- One PC connected to one Switch via a console cable (RS-232 → Console)

### 🛠️ Step-by-Step:
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# line console 0
Switch(config-line)# login local  
Switch(config-line)# exit
Switch(config)# username parsa password 123456   
Switch(config)# exit
Switch# write memory                    
```
| Command / Step                               | Description / Purpose                                         |
|---------------------------------------------|---------------------------------------------------------------|
| `line console 0`                             | Enter console line configuration mode                         |
| `login local`                                | Enable login using usernames & passwords                       |
| `exit`                                       | Exit console line configuration mode                           |
| `username parsa password 123456`            | Create a user named "parsa" with a password (level 0)         |
| `write memory` or `wr`                       | Save the configuration to NVRAM                                 |

### ✅ Verification:

1. **Test login on console:**
   - Disconnect and reconnect the console session.
   - When prompted, type:
     ```cisco
     Username: parsa
     Password: 123456
     ```
   - You should enter privileged or user mode depending on configuration.

2. **Check users in running configuration:**
    ```cisco
    Switch> enable
    Switch# show running-config
    ```
- You should see something like:
  ```
  username parsa password 0 123456
  line console 0
    login local
  ```
3. **Optional: Test multiple users**
- Create additional users and verify each can log in with their own credentials.

  
### 🔹Key Points
> Before we move to hashed passwords, here are some important things about local authentication:
> - User-based access: Each user can have a unique username and password, making it easier to track who did what on the device.
> - Multiple users: Avoid using a single password for everyone — local authentication allows multiple users with different credentials.
> - Plaintext warning: Using username <name> password <pass> stores the password as level 0 (plain text) — visible in show running-config.
> - Login local: Applying login local on console or VTY lines forces the device to ask for username + password instead of just a password.
> - Privilege levels: Users can be assigned privilege levels (0–15) to control access, but for now, just focus on creating separate users.

### ⚠️ Note:
Using plain passwords (level 0) **is not secure**. Later, you will learn how to use hashed passwords for stronger security.

---
### 📖 Part 2.2 — Hash / Salted Password
**📝 Summary**  
Using plaintext passwords is weak. Cisco allows hashed passwords (Type 5/9) to store credentials more securely.

💡 **Related Concepts:**
> - Learn the basics of [Hash Functions](https://github.com/zedparsa/Extra-Notes/blob/main/Security/Hash.md) — what they are and why they're used in password protection.
> - Understand [Salt & Pepper](https://github.com/zedparsa/Extra-Notes/blob/main/Security/Salt%20%26%20Pepper%20.md) — and how they make password hashes more secure.

**🎯 Objectives**
- Understand the difference between plaintext and hashed passwords.
- Learn how to configure a hashed password for a user.
- See how hashing with salt improves security.
- Compare Type 0 (plaintext) vs Type 5 (MD5) vs Type 9 (scrypt) in `show running-config`.

**🛠️ Step-by-Step**
- Configure a user with a hashed password:
```cisco
Switch> enable
Switch# configure terminal
Switch(config)# username parsa secret 123456
Switch(config)# username yazdan secret 654321
Switch(config)# line console 0
Switch(config-line)# login local
Switch(config-line)# end
```
| Command                                | Description                                                                 |
|----------------------------------------|-----------------------------------------------------------------------------|
| `username parsa secret 123456`           | Create a local user named "parsa" with a secret password (type 5 hashed)  |
| `username yazdan secret 654321`             | Create another local user named "yazdan" with a secret password              |

- Check users in running configuration:
 ```cisco
  Switch> enable
  Switch# show running-config
 ```
- Result :  
```cisco
  Switch# show running-config
  ...
  username parsa secret 5 $1$azVP$APCQAdM.O6BqTkXAgPGXB0
  username yazdan secret 5 $1$hKXA$T8bJ4ZV4nC5lQRuxdXY5v.
  
  line con 0
   login local
  ...
```

---
---
---

## 📖 Part 3 — TelNet & SSH :

### 📖 Part 3.1 — TelNet :

**📝 Summary** :  
Telnet provides remote access to network devices over an IP network, but it sends all data — including usernames and passwords — in plaintext, making it insecure.

**🎯 Objectives**:  
- Understand how Telnet remote access works
- Configure Telnet access on a Cisco device
- Use local authentication for Telnet
- Verify Telnet connectivity
- Recognize Telnet security weaknesses

**🧩 Topology**:
- One PC
- One Router or Switch
- PC connected to the device over the network
- IP addresses must already be configured on both sides

**🛠️ Step-by-Step**:
```cisco
Router> enable
Router# configure terminal
Router(config)# username admin password 123456
Router(config)# line vty 0 15
Router(config-line)# login local
Router(config-line)# transport input telnet
Router(config-line)# end
```

| Command              | Description                         |
|----------------------|-------------------------------------|
| `line vty 0 15`      | Enters virtual terminal (remote access) configuration mode   |
| `login local`                | Forces Telnet to use local usernames and passwords |
| `transport input telnet`         | Allows Telnet connections on VTY lines        |

- Why this configuration is needed:  
> Telnet requires user authentication to prevent unauthorized access.  
> Local authentication allows the device to prompt for a username and password.

**✅ Verification**:  
- From the PC command prompt:
```
telnet 192.168.1.1
```
| Command              | Description                         |
|----------------------|-------------------------------------|
| `telnet 192.168.1.1`      | Initiates a Telnet session to the device   |

- Login prompt:
```
Username: admin
Password: 123456
```

- If successful, you should see:
```
Router>
```

**⚠️ Note**:  
- Telnet does not encrypt data
- Usernames and passwords are sent in clear text
- Vulnerable to packet sniffing and man-in-the-middle attacks
- Telnet should be used only for labs and learning
- In real networks, SSH must always be used instead
---

### 📖 Part 3.2 — SSH :

**📝 Summary** :  
SSH (Secure Shell) provides secure remote access to network devices by encrypting all transmitted data, including usernames, passwords, and commands.  
It is the recommended replacement for Telnet.

**🎯 Objectives**:  
- Understand why SSH is preferred over Telnet
- Configure SSH on a Cisco device
- Generate RSA keys required for SSH
- Enable SSH version 2 only
- Restrict remote access to SSH
- Verify SSH connectivity

**🧩 Topology**:
- One PC
- One Router or Switch
- PC connected to the device over the network
- IP addresses must already be configured on both sides

**🛠️ Step-by-Step**:
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname R1
Router(config)# ip domain-name test.local
Router(config)# crypto key generate rsa
Router(config)# ip ssh version 2
Router(config)# username admin secret 123456
Router(config)# enable secret 123
Router(config)# line vty 0 15
Router(config-line)# login local
Router(config-line)# transport input ssh
Router(config-line)# end
```

| Command              | Description                         |
|----------------------|-------------------------------------|
| `hostname R1`      | Sets the device name (required for SSH key generation)  |
| `ip domain-name test.local` | Sets the domain name used during RSA key generation |
| `crypto key generate rsa`         | Generates RSA keys required for SSH encryption      |
| `ip ssh version 2` | Forces the device to use only SSH version 2 |
| `transport input ssh`         |   Allows only SSH connections on VTY lines        |

- Why this configuration is needed:   
> SSH requires a hostname and domain name to generate RSA keys used for encryption.  
> Local users are required because SSH uses username-based authentication.  
> Restricting VTY access to SSH prevents insecure protocols like Telnet.  

**✅ Verification**:  
- From the PC command prompt:
```
ssh -l admin 192.168.1.1
```
| Command              | Description                         |
|----------------------|-------------------------------------|
| `ssh -l admin 192.168.1.1`  | Connects to the device using SSH |

- Enter the password when prompted:
```
Password: 123456
```

- If successful, you should see:
```
R1>
```

- Enter privileged mode:
```
R1> enable
R1#
```
**⚠️ Note**:  
- SSH encrypts all session data
- SSH version 1 is insecure and must be disabled
- RSA key size should be at least 768 bits (1024 recommended)
- SSH should always be used instead of Telnet in real networks


---
---
---

## 📖 Part 4 — AAA 

### 📝 Summary:
AAA [ Authentication, Authorization, Accounting ] is a security framework used to control access to network devices and services.
It defines **who** can access the network, **what** they are allowed to do, and **what actions** they performed.
Cisco devices implement AAA using external servers such as RADIUS or TACACS+.

### 🎯 Objectives:
- Understand the AAA security model
- Learn the difference between Authentication, Authorization, and Accounting
- Understand the role of RADIUS and TACACS+
- Configure basic AAA authentication using a RADIUS server
- Apply AAA authentication to remote (VTY) access
- Verify AAA-based login using SSH


### 🧩 Topology:
- One Router (Cisco IOS)
- One PC or Laptop (SSH client)
- One AAA Server (RADIUS server)
- Router and RADIUS server in the same network
- IP connectivity verified

Example:
- Router: `192.168.1.1`
- RADIUS Server: `192.168.1.2`

### 🛠️ Step-by-Step:

#### 🔹 Background Configuration (SSH Access)
```cisco
Router> enable
Router# configure terminal
Router(config)# hostname r1
r1(config)# ip domain-name test.local
r1(config)# crypto key generate rsa
r1(config)# ip ssh version 2
r1(config)# username admin secret 123456
r1(config)# enable secret 123
r1(config)# interface g0/0/0
r1(config-if)# ip address 192.168.1.1 255.255.255.0
r1(config-if)# no shutdown
r1(config-if)# exit
r1(config)# line vty 0 4
r1(config-line)# login local
r1(config-line)# exit
```
| Command | Description |
|--------|-------------|
| `line vty 0 4` | Enters VTY (remote access) line configuration mode |

Why this is needed:
> SSH and local authentication must be functional before applying AAA      
> so we can still access the device if AAA fails.

🔹 **Step 1** — **Enable AAA**
```
r1(config)# aaa new-model
```

| Command | Description |
|--------|-------------|
| `aaa new-model	` | Enables the AAA framework on the device |

🔹 **Step 2** — **Configure RADIUS Server**
```
r1(config)# radius server myserv
r1(config-radius-server)# address ipv4 192.168.1.2
r1(config-radius-server)# key cisco123
r1(config-radius-server)# exit
```

| Command | Description |
|--------|-------------|
| `radius server myserv` |	Creates a RADIUS server profile |
| `address ipv4 192.168.1.2` |	Specifies the RADIUS server IP address |
| `key cisco123`	| Shared secret between router and RADIUS server |

🔹 **Step 3** — **Configure AAA Authentication Methods**
```
r1(config)# aaa authentication enable default group radius local
r1(config)# aaa authentication login myremote group radius local
```
| Command | Description |
|--------|-------------|
| `aaa authentication enable default group radius local` |	Uses RADIUS for enable mode, with local fallback |
| `aaa authentication login myremote group radius local` |	Creates a login authentication method list |

Why this is needed:
> The router first tries RADIUS authentication  
> If the RADIUS server is unavailable, it falls back to local users

🔹 **Step 4** — **Apply AAA to VTY Lines**
```
r1(config)# line vty 0 4
r1(config-line)# login authentication myremote
r1(config-line)# exit
```
| Command | Description |
|--------|-------------|
| `login authentication myremote` |	Applies AAA login authentication to remote access | 

### ✅ Verification:

- Connect to the router using SSH
```
ssh -l admin 192.168.1.1
```

>  If RADIUS is reachable ---> Authentication is handled by the RADIUS server  
> If RADIUS is unreachable ---> Router falls back to local authentication

- Check AAA status
```
r1# show running-config | section aaa
```

### ⚠️ Note:
- AAA must be enabled carefully — misconfiguration can lock you out
- Always configure local users before enabling AAA
- RADIUS is commonly used for end-user access (VPN, Wi-Fi)
- TACACS+ is preferred for administrator access on Cisco devices
- Cisco Identity Services Engine (ISE) is Cisco’s enterprise AAA server
- AAA separates Authentication, Authorization, and Accounting for better security and control

---
---
---

## 📖 Part 5 — Cisco IOS Banners

### 📝 Summary:
Cisco IOS banners are used to display messages to users when they access a network device.
They are commonly used for **security warnings**, **legal notices**, and **informational messages**.
Different banner types are shown at different stages of the login process.


### 🎯 Objectives:
- Understand the purpose of Cisco IOS banners
- Learn the different types of banners
- Configure MOTD and Login banners
- Identify when each banner is displayed
- Recognize the importance of banners in security and compliance


### 🧩 Topology:
- One Router or Switch
- Local or remote access (Console, Telnet, or SSH)
- No specific network configuration required


### 🛠️ Step-by-Step:

#### 🔹Banner Types Overview

- Cisco IOS supports **four main banner types**:

> - **MOTD (Message of the Day)**  
  Displayed **before login**  
  Most important and commonly used banner  
  Typically used for security and legal warnings  
> - **Login Banner**  
  Displayed **before entering username and password**  
> - **Exec Banner**  
  Displayed **after a successful login**  
> - **Incoming Banner**  
  Displayed for incoming connections (rarely used)



#### 🔹 Step 1 — Configure Login Banner
```cisco
r1> enable
r1# configure terminal
r1(config)# banner login @
Authorized users only!
@
```

| Command | Description |
|--------|-------------|
| `banner login @` |	Defines the start delimiter for the login banner |
| `Authorized users only!` |	Message displayed to users |
| `@` |	End delimiter |  

Why this is needed:  
> The login banner warns users before authentication, often used for legal notice or access restrictions.

#### 🔹 Step 2 — Configure MOTD Banner
```
r1> enable
r1# configure terminal
r1(config)# banner motd @
Welcome!
@
```

| Command | Description |
|--------|-------------|
| `banner motd @` |	Defines the start delimiter for the MOTD banner |

Why this is needed:
>  The MOTD banner is shown before login and is the most visible banner.  
> It is commonly required for security policies and legal compliance.

### ✅ Verification:

1. Exit the current session.
2. Reconnect to the device using console, Telnet, or SSH.
3. Observe the displayed banners:  
> MOTD banner appears first  
> Login banner appears before authentication  
> Exec banner (if configured) appears after login

Optional verification:
```
r1# show running-config | include banner
```

### ⚠️ Note:  
- The delimiter can be any character, but it must be the same at the start and end  
- The delimiter character must not appear inside the message  
- MOTD is the most important banner and commonly required in real networks  
- Banners do not provide security by themselves, but support legal and policy enforcement  
- Incorrect banner configuration can confuse users or violate compliance requirements

---
---
---

## 📖 Part 6 — NTP 


### 📝 Summary:
NTP (Network Time Protocol) is used to synchronize time across network devices.
It operates using **UDP port 123** and distributes time through a **hierarchical stratum system**.
Accurate time is essential for logging, security, troubleshooting, and network monitoring.


### 🎯 Objectives:
- Understand the purpose of NTP
- Learn how the NTP stratum system works
- Configure a Cisco device as an NTP client
- Configure a Cisco device as an NTP master
- Verify NTP synchronization status

### 🧩 Topology:
- One Router (or Switch)
- Optional external NTP server (Internet-based)
- Network connectivity available

Example:
- External NTP Server: Public Stratum 1 or 2 server
- Internal Devices: NTP clients


### 🛠️ Step-by-Step:

#### 🔹 NTP Stratum Concept (Overview)

Stratum in NTP represents how close a device is to the original reference clock.

- **Stratum 0**  
  Reference clocks such as atomic clocks or GPS  
  Do not directly participate in NTP

- **Stratum 1**  
  Servers directly connected to Stratum 0 devices

- **Stratum 2–15**  
  Servers and clients that obtain time from lower-stratum servers

- **Stratum 16**  
  Indicates the device is **unsynchronized** and time should not be trusted

Lower stratum numbers indicate **higher accuracy**.


#### 🔹 Step 1 — Configure NTP Client (Using External NTP Server)
```cisco
Router> enable
Router# configure terminal
Router(config)# ntp server <ip-address>
```

| Command | Description |
|--------|-------------|
| `ntp server <ip-address>` | 	Configures the device to synchronize time with an external NTP server |

Why this is needed:
> This command configures the router to synchronize its clock with an external NTP server.  
> The server is usually a public Stratum 1 or Stratum 2 server available on the Internet.

#### 🔹 Step 2 — Configure NTP Master (Internal Time Source)

```
Router(config)# ntp master
```

| Command | Description |
|--------|-------------|
| `ntp master` |	Sets the device as the NTP master for the network |

Why this is needed:  
> This command makes the device act as the NTP master for other network devices.  
> It is commonly used when:  
> - Internet access is unavailable  
> - A centralized internal time source is required


By default, the device becomes a Stratum 8 NTP server.

### ✅ Verification:

Check NTP synchronization status:
```
Router# show ntp status
```

Check NTP associations:
```
Router# show ntp associations
```
| Command | Description |
|--------|-------------|
| `show ntp status`	| Displays NTP synchronization state and stratum |
| `show ntp associations` |	Shows configured NTP servers and relationships |

### ⚠️ Note:

- NTP uses UDP port 123  
- Devices adjust time gradually to avoid sudden jumps  
- Lower stratum means higher accuracy  
- ntp master is suitable for small or isolated networks  
- For enterprise networks, use external or centralized NTP servers  
- Incorrect system time can cause issues with logs, certificates, and security protocols

---
---
---

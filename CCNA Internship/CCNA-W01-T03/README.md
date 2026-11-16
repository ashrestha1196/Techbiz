# Cisco IOS Security, SSH Configuration & Password Recovery

**Tech Biz Academy | CCNA Networking Internship | Week 1 — Task 3**

## Project Overview

This lab focuses on securing and managing Cisco network devices using the Cisco IOS Command Line Interface (CLI).

Using Cisco Packet Tracer, I configured two Cisco 2901 routers and two Catalyst 2960 switches, implemented secure remote management through SSH, verified device configurations, and performed password recovery using ROMMON.

The objective was to gain hands-on experience with Cisco IOS administration, device authentication, configuration persistence, and troubleshooting.

## Lab Environment

| Component | Details |
|---|---|
| Simulation Platform | Cisco Packet Tracer |
| Routers | 2 × Cisco 2901 (R1, R2) |
| Switches | 2 × Cisco Catalyst 2960 (SW1, SW2) |
| End Devices | PCs for console and SSH access |
| Remote Management | SSH Version 2 |
| Authentication | Local user database and enable secret |

### Management IP Addresses

| Device | IP Address |
|---|---|
| R1 | 192.168.1.1 |
| R2 | 192.168.1.10 |
| SW1 | 192.168.1.2 |
| SW2 | 192.168.1.11 |

## Lab Objectives

- Understand Cisco IOS user EXEC, privileged EXEC, and configuration modes.
- Configure device hostnames and MOTD security banners.
- Secure console access and privileged EXEC mode.
- Configure local user authentication and SSHv2.
- Restrict VTY lines to SSH connections.
- Verify interface status and device configurations.
- Save configurations and verify persistence after reboot.
- Perform router password recovery using ROMMON.

## 1. Cisco IOS Device Security

Configured the following security features on R1, R2, SW1, and SW2:

- Unique device hostnames
- MOTD authorization banners
- Enable secrets for privileged EXEC access
- Console passwords with login authentication
- Local user accounts for SSH access
- VTY authentication restricted to SSH

Example configuration (credentials replaced with placeholders):

```cisco
hostname R1

enable secret <ENABLE_SECRET>

banner motd #Authorized Access Only!#

username admin secret <SSH_SECRET>

line console 0
 password <CONSOLE_PASSWORD>
 login
 exit
```

These settings provide separate authentication controls for console access, remote management, and privileged EXEC mode.

## 2. SSH Configuration and Testing

Configured SSH Version 2 for secure remote management.

### SSH Configuration

```cisco
ip domain-name lab.local

crypto key generate rsa
! Selected 2048-bit modulus when prompted

ip ssh version 2

line vty 0 4
 login local
 transport input ssh
```

### SSH Verification

From a Packet Tracer PC:

```text
PC> ssh -l admin 192.168.1.1
Password:

R1> enable
Password:
R1#
```

Successfully tested remote SSH connections to all four devices.

**Result:** SSH authentication and privileged EXEC access were verified through Packet Tracer PC sessions.

## 3. Switch Management Using an SVI

A Layer 2 Catalyst 2960 switch does not require an IP address for normal Ethernet frame forwarding. However, an IP address is required for remote management through SSH.

Configured a Switch Virtual Interface (SVI) to provide management connectivity.

Example based on SW2:

```cisco
interface vlan 1
 ip address 192.168.1.11 255.255.255.0
 no shutdown
```

The management SVI allows administrators to communicate with the switch over IP.

For remote management from another subnet, the switch also requires an appropriate default gateway.

## 4. IOS Configuration Verification

Used the following commands to inspect device configurations and operational status.

| Command | Purpose |
|---|---|
| `show running-config` | Displays the active configuration stored in RAM |
| `show ip interface brief` | Summarizes interface IP addresses and operational states |
| `show version` | Displays IOS version, hardware details, uptime, and configuration register |
| `show interfaces status` | Displays switchport connectivity and operational information |

### Interface Status Verification

On R1, the `show ip interface brief` output showed:

| Interface | IP Address | Status | Protocol |
|---|---|---|---|
| GigabitEthernet0/0 | Unassigned | Administratively down | Down |
| GigabitEthernet0/1 | 192.168.1.1 | Up | Up |
| Vlan1 | Unassigned | Administratively down | Down |

**Key learning:** The Status column primarily reflects the physical/administrative interface state, while Protocol represents the Layer 2 line-protocol state.

## 5. Saving and Reloading Configurations

Saved the active configuration using:

```cisco
copy running-config startup-config
```

This copies the configuration from RAM to persistent startup configuration storage.

After reloading R1, I verified that the hostname, interface configuration, console authentication, and enable secret remained available.

**Result:** Configuration persistence was demonstrated after reboot.

## 6. Cisco 2901 Password Recovery Using ROMMON

Performed an authorized password-recovery exercise on R1.

### Recovery Procedure

**Step 1 — Enter ROMMON**

Interrupted the router boot process using `Ctrl+C` in Packet Tracer.

```text
rommon 1 >
```

**Step 2 — Bypass Startup Configuration**

```text
rommon 1 > confreg 0x2142
rommon 2 > reset
```

The router restarted without applying the saved startup configuration.

**Step 3 — Restore the Original Configuration**

After booting into privileged EXEC mode:

```cisco
copy startup-config running-config
```

This restored the previously saved configuration into RAM.

**Step 4 — Configure a New Enable Secret**

```cisco
configure terminal
enable secret <NEW_ENABLE_SECRET>
```

**Step 5 — Restore the Configuration Register**

```cisco
config-register 0x2102
end
```

**Step 6 — Save and Reload**

```cisco
copy running-config startup-config
reload
```

After restarting, I verified access using the updated enable secret and confirmed that the original device settings were retained.

### Important Lesson

The `0x2142` configuration register allows the router to bypass applying its startup configuration without deleting it.

Saving the temporary default running configuration before restoring the original startup configuration could overwrite the saved settings.

## 7. Troubleshooting Experience

During the switch configuration, I encountered an authentication issue after entering:

```cisco
enable secret 5 Secret55
```

The `5` parameter indicates that the supplied value is already a Type 5 password hash.

For a new plaintext password, the correct syntax is:

```cisco
enable secret <NEW_SECRET>
```

This troubleshooting experience reinforced the importance of understanding Cisco IOS command syntax rather than simply memorizing commands.

## 8. Lab Results

| Test | Result |
|---|---|
| Device hostname and banner configuration | Passed |
| Console authentication | Passed |
| Enable secret authentication | Passed |
| SSH access to R1 and R2 | Passed |
| SSH access to SW1 and SW2 | Passed |
| IOS configuration verification | Passed |
| Configuration save and reload | Passed on R1 |
| Cisco 2901 ROMMON password recovery | Passed |

## 9. Evidence and Screenshots

The following screenshots document the configuration and verification process:

- R1 and R2 device security configurations
- SW1 and SW2 device security configurations
- Successful SSH sessions from Packet Tracer PCs
- `show running-config` output
- `show ip interface brief` output
- `show version` output
- ROMMON password-recovery procedure
- Post-recovery authentication and interface verification

Screenshots and configuration exports should have reusable credentials redacted before publication.

## 10. Key Takeaways

Through this lab, I gained practical experience in:

- Cisco IOS administration and privilege levels
- Console and SSH authentication
- Secure remote management using SSHv2
- Management IP addressing on Layer 2 switches
- Device configuration verification and troubleshooting
- Configuration persistence after reboot
- Cisco router password recovery using ROMMON

The lab helped connect Cisco IOS security concepts with practical network administration and troubleshooting tasks.

---

**Tools:** Cisco Packet Tracer, Cisco IOS CLI, SSH, ROMMON  
**Internship:** Tech Biz Academy — CCNA Networking  
**Task:** Week 1, Task 3

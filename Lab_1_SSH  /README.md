# Lab 1: SSH

## Lab Objective

The objective of this lab exercise is to learn and understand how to enable SSH access to a device — in this case, a Cisco router.

## Lab Purpose

It's never a good idea to permit Telnet access to network devices, especially in corporate settings. SSH is a secure way to connect to network devices. In order to configure SSH you need to:

1. Create a hostname.
2. Create a domain name.
3. Generate a crypto key.

## Lab Tool

Cisco Packet Tracer

## Lab Topology

Two routers connected back-to-back via a crossover/serial link (no switch required):

```
Router0 (ISR4331) ---- Router1 (ISR4321)
   192.168.1.1              192.168.1.2
```

## Tasks

- **Task 1:** Change the hostnames
- **Task 2:** Add IP addresses
- **Task 3:** Secure Router1 using SSH
- **Task 4:** Connect both routers using SSH

---

## Lab Walkthrough

### Task 1: Change the Hostnames

Drag two routers onto the canvas and connect them. When each router boots, always answer `no` to the initial configuration dialog (the router will otherwise drop into a question-and-answer self-configuration mode).

Configure the hostnames on Router0 and Router1 as shown in the topology.

```
--- System Configuration Dialog ---
Continue with configuration dialog? [yes/no]: no

Press RETURN to get started!

Router>enable
Router#config t
Enter configuration commands, one per line. End with CNTL/Z.
Router(config)#hostname R0
R0(config)#
```

Repeat the same steps on the second router, giving it the hostname `R1`.

### Task 2: Add IP Addresses

Add an IP address to each router's Ethernet interface and bring the interface up with `no shutdown`.

**On R0:**

```
R0(config)#interface g0/0
R0(config-if)#ip address 192.168.1.1 255.255.255.0
R0(config-if)#no shut
%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to up
```

**On R1:**

```
R1(config)#interface g0/0
R1(config-if)#ip address 192.168.1.2 255.255.255.0
R1(config-if)#no shut
%LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to up
R1(config-if)#end
```

Verify connectivity across the link:

```
R1#ping 192.168.1.1

Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 192.168.1.1, timeout is 2 seconds:
.!!!!
Success rate is 80 percent (4/5), round-trip min/avg/max = 0/0/0 ms
```

### Task 3: Secure Router1 with SSH

Secure R1 so it accepts incoming SSH connections. This requires setting a domain name and generating an RSA key pair. Optionally, set the authentication retry count and an idle timeout.

```
R1#conf t
Enter configuration commands, one per line. End with CNTL/Z.
R1(config)#ip domain-name 101labs.net
R1(config)#crypto key generate rsa
The name for the keys will be: R1.101labs.net
Choose the size of the key modulus in the range of 360 to 2048 for your General Purpose Keys.
Choosing a key modulus greater than 512 may take a few minutes.

How many bits in the modulus [512]: 1024
% Generating 1024 bit RSA keys, keys will be non-exportable...[OK]

R1(config)#ip ssh time-out 60
R1(config)#ip ssh authentication-retries 2
R1(config)#line vty 0 15
R1(config-line)#transport input ssh
R1(config-line)#password cisco
R1(config-line)#end
```

There are 16 VTY lines available on most Cisco devices (0–15). The `transport input ssh` command restricts these lines to SSH-only access (no Telnet).

Verify SSH is enabled:

```
R1#show ip ssh
SSH Enabled - version 1.99
Authentication timeout: 60 secs; Authentication retries: 2
```

### Task 4: Connect Both Routers Using SSH

From Router0, connect to Router1 over SSH. You'll be prompted for the VTY password (`cisco`).

```
R0#ssh -l paul 192.168.1.2
Open
Password:
R1>
```

> **Note:** Use the letter `l` after `ssh -`, not the number `1`.

To end the SSH session, press `Ctrl + Shift + 6`, release, then press `X`.

### Task 5: Confirm Telnet Is Refused

Attempt to Telnet from Router0 to Router1 to confirm the connection is rejected (since only SSH is permitted on the VTY lines).

```
R0#telnet 192.168.1.2
Trying 192.168.1.2 ...Open
[Connection to 192.168.1.2 closed by foreign host]
R0#
```

---

## Summary

| Step | Command | Purpose |
|------|---------|---------|
| 1 | `hostname R0 / R1` | Identify each router |
| 2 | `ip address` / `no shut` | Bring up interfaces and enable connectivity |
| 3 | `ip domain-name` | Required before generating RSA keys |
| 3 | `crypto key generate rsa` | Generates the key pair SSH depends on |
| 3 | `ip ssh time-out` / `ip ssh authentication-retries` | Hardens SSH access |
| 3 | `transport input ssh` | Restricts VTY lines to SSH only |
| 4 | `ssh -l <user> <ip>` | Connects securely to the remote router |

**Key takeaway:** SSH requires a hostname, a domain name, and a generated RSA key before it can be enabled — and restricting `transport input` to `ssh` is what blocks insecure Telnet access.

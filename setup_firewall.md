# Firewall

Reference:
https://linuxhint.com/debian_linux_firewall_best_practices/

The defatult method for incoming traffic is `restrictive`: all traffic or packets which are not defined among its rules isn’t allowed to pass.

For outgoing traffic, the default is `permissive` policy.


# UFW

UFW brought the capability to quickly setup a customized firewall without learning unfriendly syntax.


## Install

Install UFW by running:
```bash
apt install ufw
``` 

## Enable UFW:

Before enable, make sure `ufw` is allowing ssh, otherwise the `enable` command may disrupt existing ssh connections:
To allow incoming connections to a port:
```bash
ufw allow ssh
```

Enable command:
```bash
ufw enable
```

The command to disable UFW:
```bash
ufw disable
```

## Status

If you want to carry out a fast check on your firewall status run:

```bash
ufw status
```


## Basic Configuration

In order to restrict all incoming traffic by default using ufw run:

```bash
ufw default deny incoming
```

To allow all outgoing traffic we just replace “deny” for “allow”, to allow outgoing traffic unconditionally run:

```bash
ufw default allow outgoing 
```

We can limit the login attempts to prevent brute force attacks by setting a limit running:
```bash
ufw limit ssh
```


## Services Configuration

To allow incoming connections to a port:
```bash
ufw allow 22
```

If we want to remove a specific rule, we can do it with the parameter “delete”:
```bash
ufw delete allow 22
```


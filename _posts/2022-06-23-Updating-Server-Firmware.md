---
layout: page
title: Updating Dell R710 iDRAC
permalink: /Updating-Dell-R710
---

Updating iDRAC on a Dell R710.

In this post I will cover my experience trying to update iDRAC on my Dell R710.

## Missing Dependenices

While trying to run the downloaded script `./ESM_Firmware_KPCCC_LN32_2.92_A00.BIN` from the Dell support website, I received the following error:

```
./getSystemId: not found 
not a dynamic executable 
This Update Package is not compatible with your system. 
spsetup.sh: 658: ./sputility.bin: not found
```

The mentioned files are stored inside the ESM_Firmware BIN file and can be extracted using the following command
`./ESM_Firmware_KPCCC_LN32_2.92_A00.BIN --extract .`
Now that I have the files that the script is trying to execute I can inspect why these issues are occuring.

Running `./getSystemId` returned a strange result
```
root@proxmox1:~# ./getSystemId
-bash: ./getSystemId: No such file or directory
```

Hmm... This is very strange. So I wanted to inspect the file more so I used the file command:
```
root@proxmox1:~# file getSystemId
getSystemId: ELF 32-bit LSB executable, Intel 80386, version 1 (SYSV), dynamically linked, interpreter /lib/ld-linux.so.2, for GNU/Linux 2.2.5, stripped
```

So I cann see that it is a 32-bit Linux executable that is dynamically linked to interpreter /lib/ld-linux.so.2
Does this proxmox system even have `/lib/ld-linux.so.2`???

```
root@proxmox1:~# ls /lib/ld-linux.so.2
ls: cannot access '/lib/ld-linux.so.2': No such file or directory
```

Ok so I need to install the 32-bit version of the package that contains this interpreter.
The debian package can be found [here.](https://packages.debian.org/cgi-bin/search_contents.pl?word=ld-linux.so.2&searchmode=searchfiles&case=insensitive&version=stable&arch=i386)

To install it I ran
```
dpkg --add-architecture i386
apt update
apt install libc6:i386
```

I then had to find out where it placed the ld-linux.so.2 file that I was after so I ran this command
```
root@proxmox1:~# dpkg -L libc6:i386 | grep ld-linux.so.2
/lib/i386-linux-gnu/ld-linux.so.2
/lib/ld-linux.so.2
```

Now I have the interpreter that was needed by the iDRAC update script, lets try run it again.

And another error

```
root@proxmox1:~# ./ESM_Firmware_KPCCC_LN32_2.92_A00.BIN
./getSystemId: error while loading shared libraries: libstdc++.so.6: cannot open shared object file: No such file or directory
Error while loading shared libraries: libstdc++.so.5:
cannot open shared object file: No such file or directory.
Install the following dependencies:
RHEL: compat-libstdc.i686
SLES: libstdc++-33-32bit
Please check Dell Update package User guide for instructions
for installing the dependencies.
./sputility.bin: error while loading shared libraries: libxml2.so.2: cannot open shared object file: No such file or directory
```
The libxml2 package can be found [here](https://packages.debian.org/buster/libxml2)

So trying to install it gives
```
root@proxmox1:~# apt install libxml2
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
libxml2 is already the newest version (2.9.10+dfsg-6.7+deb11u2).
0 upgraded, 0 newly installed, 0 to remove and 48 not upgraded.
```

Ok, so it's already installed. Let's find it...

```
root@proxmox1:~# dpkg -L libxml2 | grep libxml2.so.2
/usr/lib/x86_64-linux-gnu/libxml2.so.2.9.10
/usr/lib/x86_64-linux-gnu/libxml2.so.2
```

Let's create a symbolic link to this file in the /lib directory
`ln -s /lib/libxml2.so.2 /usr/lib/x86_64-linux-gnu/libxml2.so.2`


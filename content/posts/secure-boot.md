---
title: "Secure Boot"
date: 2024-01-01T14:06:37-03:00
draft: false
toc: false
images:
tags: 
    - linux
    - english
    - tech
---

# Manually configuring secure boot in Linux


Configuring secure boot on linux is a pain in the ass.
Luckily most popular distros like Fedora, Ubuntu and openSuse already do this automatically,
so you should only need to follow this if you are using a DIY distro or for some
reason want control for all secure boot keys in your firmware (which is a valid reason).
The main reason for me to write this is that when I had to enable secure boot in my
Gentoo installation I had a lot of problems following through existing tutorials,
so I re-did all the configuration and tried to keep notes to keep things as simples
as possible

Theres something that are important to keep in mind about this post:
    - This will reset every key in your firmware, if you want to keep old keys, look at the resources as some teach how to do that
    - There will be no dbx list (dbx is the known bad keys list)
    - There will be no Microsoft keys added, if like me you're dual booting, you have to sign your Windows Boot Loader

There's a link to the Microsoft secure boot management article with links for all the microsoft's KEK and db keys
that you can append to your local key database, I just don't want their keys.

Required softwares:
    - C compiler (no C knowledge required tho)
    - [git](https://repology.org/project/git/versions)
    - [refind](https://repology.org/project/refind/versions)
    - [mokutil](https://repology.org/project/mokutil/versions)

## Let's start

first go to your bios, disable secure boot, and clean your key database

First we have to disable the secure boot and clean the keys in the BIOS.
I can't help with that as the process is different for each motherboard,
but all the settings should be in the Boot section.

After disabling secure boot and cleaning the keys, boot your system and open a
terminal as root.
Add create a `keys` folder in the root home directory

### Creating keys

First we set the name for the certificates, this can be your user, real name or whatever really
```sh
$ export NAME="<nome for the certificates>"
```

Then we create the certificates and der files that will be used later
```sh
$ openssl req -new -x509 -newkey rsa:2048 -subj "/CN=$NAME PK/" -keyout PK.key \
        -out PK.crt -days 3650 -nodes -sha256
$ openssl req -new -x509 -newkey rsa:2048 -subj "/CN=$NAME KEK/" -keyout KEK.key \
        -out KEK.crt -days 3650 -nodes -sha256
$ openssl req -new -x509 -newkey rsa:2048 -subj "/CN=$NAME DB/" -keyout DB.key \
        -out DB.crt -days 3650 -nodes -sha256
$ openssl req -new -x509 -newkey rsa:2048 -subj "/CN=$NAME DB/" -keyout MOK.key \
        -out MOK.crt -days 3650 -nodes -sha256
```

```sh
$ openssl x509 -in PK.crt -out PK.cer -outform DER
$ openssl x509 -in KEK.crt -out KEK.cer -outform DER
$ openssl x509 -in DB.crt -out DB.cer -outform DER
$ openssl x509 -in MOK.crt -out MOK.cer -outform DER
```

Then we create auth lists from the certificates
```sh
$ cert-to-efi-sig-list -g "$(uuidgen)" PK.crt PK.esl
$ sign-efi-sig-list -t "$(date --date='1 second' +'%Y-%m-%d %H:%M:%S')" \
                  -k PK.key -c PK.crt PK PK.esl PK.auth

$ cert-to-efi-sig-list -g "$(uuidgen)" KEK.crt KEK.esl
$ sign-efi-sig-list -t "$(date --date='1 second' +'%Y-%m-%d %H:%M:%S')" \
                  -k PK.key -c PK.crt KEK KEK.esl KEK.auth

$ cert-to-efi-sig-list -g "$(uuidgen)" DB.crt DB.esl
$ sign-efi-sig-list -t "$(date --date='1 second' +'%Y-%m-%d %H:%M:%S')" \
                  -k KEK.key -c KEK.crt db DB.esl DB.auth
```

And lastly, we set readonly permission to the files
```sh
$ chmod -v 400 *.key
```

### Uploading keys

First we run `efi-readvar` just to make sure the secureboot keys are clear the output should be:
```sh
$ efi-readvar
Variable PK has no entries
Variable KEK has no entries
Variable db has no entries
Variable dbx has no entries
```

If you get an error like `Failed to update KEK: Operation not permitted`
or `Cannot write to KEK, wrong filesystem permissions`
change the immutable attribute from the keys
```sh
$ chattr -i /sys/firmware/efi/efivars/{PK,KEK,db}*
```

Finally we upload the keys
```sh
$ efi-updatevar -e -f DB.esl db
$ efi-updatevar -e -f KEK.esl KEK
$ efi-updatevar -f PK.auth PK
$ mokutil -i MOK.cer
```

Technically, here the secure boot is configured, you could go to the BIOS and enable it.
BUT, there's no boot loader or kernel signed

### Signing

First thing we need is to compile shim and install refind.
Shim source code can be downloaded [here](https://github.com/rhboot/shim/releases).
I am using version 15.7.

After downloading, move it to the root home folder.
```sh
$ tar xvf shim-15.7.tar.bz2 
$ cd shim-15.7
$ make VENDOR_CERT_FILE=/root/keys/db.cer
$ refind-install --shim shimx64.efi
```

Now that the bootloaders are actually installed let's sign all the binaries.

First we have to sign shim and refind.
Both files should be at `/boot/efi/EFI/refind/`
```sh
$ sbsign --key ~/keys/DB.key --cert ~/keys/DB.crt --output shimx64.efi shimx64.efi
$ sbsign --key ~/keys/MOK.key --cert ~/keys/MOK.crt --output grubx64.efi grubx64.efi
```

And sign the ext4 driver in the `drivers_x64` folder
```sh
$ sbsign --key ~/keys/MOK.key --cert ~/keys/MOK.crt --output ext4_x64.efi ext4_x64.efi
```

Lastly we have to sign the kernel in `/boot/`
Change `<vmlinuz>` to the actual name of your kernel file
```sh
$ sbsign --key ~/keys/MOK.key --cert ~/keys/MOK.crt --output <vmlinuz> <vmlinuz>
```

If you are using only Linux, you are done here.
You can safely reboot and enable secure boot at the BIOS.

If you have dual boot with Windows, there's one last step.
We have to sign the Windows boot file in `/boot/efi/EFI/Microsoft/Boot/`.
```sh
$ sbsign --key ~/keys/MOK.key --cert ~/keys/MOK.crt --output bootmgfw.efi bootmgfw.efi
```

Now we are done.
You can safely reboot and enable secure boot at the BIOS.


Resources
- https://wiki.gentoo.org/wiki/User:Sakaki/Sakaki%27s_EFI_Install_Guide/Configuring_Secure_Boot
- https://www.rodsbooks.com/efi-bootloaders/secureboot.html#add_keys
- https://www.rodsbooks.com/efi-bootloaders/controlling-sb.html
- https://github.com/rhboot/shim
- https://learn.microsoft.com/en-us/windows-hardware/manufacture/desktop/windows-secure-boot-key-creation-and-management-guidance?view=windows-11


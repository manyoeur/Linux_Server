# How to Reset Root Password in CentOS (7 / 8 / Stream)
<b>Step 1 — Reboot your system</b>
* Restart the server or VM.

<b>Step 2 — Interrupt GRUB</b>

When you see the GRUB boot menu:
* Press <b>e</b> to edit the default boot entry.
<img width="953" height="604" alt="image" src="https://github.com/user-attachments/assets/93a3b321-2dd5-404a-b572-90ce88dbcbe3" />

<b>Step 3 — Modify kernel boot parameters</b>
Find the line starting with:
```
linux16
```
 or on CentOS 8:
 ```
linux
```
At the end of that line, append:
```
rd.break
```
Example:
```
linux16 /vmlinuz-3.10.0... ro rd.break
```

<img width="1268" height="795" alt="image" src="https://github.com/user-attachments/assets/d6200bf8-7b71-4217-aa18-6f1855d6c807" />

<b>Step 4 — Boot into emergency mode </b>
Press:
```
Ctrl + X
```
<b>Step 5 — Remount the system writable </b>
Root filesystem is mounted read-only at /sysroot.
```
mount -o remount,rw /sysroot
```
<b>Step 6 — Chroot into the real system </b>
```
chroot /sysroot
```
<b>Step 7 — Change the root password </b>
```
passwd root
```
Enter new password twice.
<b>Step 8 — Relabel SELinux (important!) </b>
```
touch /.autorelabel
```
<b>Step 9 — Exit and reboot </b>
Exit chroot:
```
exit
```
Exit emergency shell:
```
exit
```
System reboots normally.
# ✅ Done!

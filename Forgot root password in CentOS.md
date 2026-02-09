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

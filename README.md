# linux-embedded-patches
a set of patches for embedded linux-based systems
the patches were tested on kernel version 6.8.12

## pathes
* disable_vt_swithing_from_keyboard.patch - disables VT switching at the kernel level, but VT switching can still work from x11. it completely kills VT switching from the keyboard, but does not prevent VT switching from userspace (for example, via chvt)

## recommended kernel parameters
* sysrq=0 - for obvious reasons, sysrq should be disabled on an embedded device
* mitigations=off - you can add this to the kernel parameter to disable protection against CPU hardware vulnerabilities like Spectre and Meltdown. this will improve performance on older processors like intel atom. DO NOT DO THIS ON SECURITY-CRITICAL DEVICES

## disabling switching VT at the x11 level
path: /etc/X11/xorg.conf.d/10-novtswitch.conf  
content:
```
Section "ServerFlags"
    Option "DontVTSwitch" "true"
EndSection
```
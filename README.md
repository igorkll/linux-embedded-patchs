# linux embedded patches
a set of patches for embedded linux systems
the patches were tested on kernel version 6.8.12
these patches should also work on newer kernel versions, because the `patch` utility applies changes using contextual lines and can automatically adjust offsets if the surrounding code has shifted
in order for these patches to work, make sure that `CONFIG_WERROR` is NOT enabled in the kernel config

## pathes
* disable_vt_swithing_from_keyboard.patch - disables VT switching at the kernel level, but VT switching can still work from x11. it completely kills VT switching from the keyboard, but does not prevent VT switching from userspace (for example, via chvt). please note that if you disabled VT switching using the patch, it will only work in tty! switching processing can still occur at the graphics session level, it's easy to disable in x11, but it depends on the composer in wayland
* disable_sysrq.patch - it completely prohibits the operation of sysrq, regardless of the kernel parameters
* disable_cad.patch - blocks restarting by pressing ctrl+alt+del
* disable_printk.patch - will make the kernel shut up

## apply patch
patches are applied to the kernel source code before it is build
run this command for the necessary patches from the kernel source folder:
patch -p1 < kernel_patch.patch

## disabling switching VT at the x11 level
path: /etc/X11/xorg.conf.d/10-novtswitch.conf  
content:
```
Section "ServerFlags"
    Option "DontVTSwitch" "true"
EndSection
```
# linux embedded patches
a set of patches for embedded linux systems
the patches were tested on kernel version 6.8.12
these patches should also work on newer kernel versions, because the `patch` utility applies changes using contextual lines and can automatically adjust offsets if the surrounding code has shifted
in order for these patches to work, make sure that `CONFIG_WERROR` is NOT enabled in the kernel config

## you may also be interested in
* https://github.com/igorkll/custom-debian-initramfs-init - custom /init script for debian initramfs
* https://github.com/igorkll/syslbuild - creating custom embedded linux systems
* https://github.com/igorkll/WinBox-Maker - a program for creating embedded Windows images

## pathes
* disable_vt_swithing_from_keyboard.patch - disables VT switching at the kernel level, but VT switching can still work from x11. it completely kills VT switching from the keyboard, but does not prevent VT switching from userspace (for example, via chvt). please note that if you disabled VT switching using the patch, it will only work in tty! switching processing can still occur at the graphics session level, it's easy to disable in x11, but it depends on the composer in wayland
* disable_sysrq.patch - it completely prohibits the operation of sysrq, regardless of the kernel parameters
* disable_cad.patch - blocks restarting by pressing ctrl+alt+del
* disable_printk.patch - will make the kernel shut up
* disable_vt_swithing_from_wayland.patch - blocks VT switching from the wayland side. in some composers, this can be disabled normally, but for example, for plasma, I did not find such a way. please note that this patch is implemented VERY STRANGELY inside and, I would say, incorrectly. it works with kde plasma by blocking VT switching for its session, but I'm not sure that it will work fine with other composers (if your composer supports disabling VT switching normally, then use its setting and not this patch) the algorithm of this patch is as follows: it prevents switching via ioctl from tty being in "graphical" mode, but it allows you to do this once from the moment the kernel is started (without this, the cursor remains on a black background, apparently switching from DM to DE but I'm not sure) anyway, this terrible solution works in cases where sddm+plasmawayland is used. the correct solution would be to patch the wayland composer rather than the core

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
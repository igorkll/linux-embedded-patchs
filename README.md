# linux embedded patches
![logo](https://raw.githubusercontent.com/igorkll/linux-embedded-patchs/refs/heads/main/logo.png)  
a set of patches for embedded linux systems
the patches were tested on kernel version 6.8.12
these patches should also work on newer kernel versions, because the `patch` utility applies changes using contextual lines and can automatically adjust offsets if the surrounding code has shifted
in order for these patches to work, make sure that `CONFIG_WERROR` is NOT enabled in the kernel config  
please note that there are other patch versions for newer kernel versions  

## you may also be interested in
* https://github.com/igorkll/custom-debian-initramfs-init - custom /init script for debian initramfs
* https://github.com/igorkll/syslbuild - creating custom embedded linux systems
* https://github.com/igorkll/WinBox-Maker - a program for creating embedded Windows images
* https://github.com/igorkll/embedded-plymouth - plymouth with a patch to disable ESC key processing (so that the console cannot be displayed during boot)
* https://github.com/igorkll/Gnubox-Maker - a tool for building linux with a single application. It is part of syslbuild and directly uses some of these patches
* https://github.com/igorkll/linux-embedded-setup-scripts - scripts for configuring kiosks in linux

## roadmap
* patch to completely disable core dump creation
* a patch for a complete visual shutdown of VT. not breaking userspace dependent on VT, but completely destroying any possibility of input/output via VT and displaying anything
* patch for complete removal of OOM killer (possible, I haven't decided yet if this is a good idea)
* a patch to completely disable keyboard echo
* a patch to completely disable cursor blinking
* a patch that adds the "fakeconsole" driver so that you can specify "console=fakeconsole". this will allow the output to be routed from anywhere without breaking the userspace as "console=null" does

### completed
* a patch to remove reboot and shutdown messages

## buildtest
to check the build of all patches, you can use the "testbuild.json" by running it through syslbuild  
this will check the build of all patches on multiple versions of the kernel  

## pathes
* 7.0/* - some patches have been redesigned for the new 7.0 kernel version.
* alternatives/* - if not one of the patch options is suitable for your kernel. you can search for a suitable one in this directory
* disable_vt_swithing_from_keyboard.patch - disables VT switching at the kernel level, but VT switching can still work from x11. it completely kills VT switching from the keyboard, but does not prevent VT switching from userspace (for example, via chvt). please note that if you disabled VT switching using the patch, it will only work in tty! switching processing can still occur at the graphics session level, it's easy to disable in x11, but it depends on the compositor in wayland
* disable_sysrq.patch - it completely prohibits the operation of sysrq, regardless of the kernel parameters
* disable_cad.patch - blocks restarting by pressing ctrl+alt+del
* disable_printk.patch - will make the kernel shut up. provides COMPLETELY SILENT loading in any situation, completely breaking printk (makes debugging almost impossible. use it only when you are sure that everything is working.)
* disable_vt_swithing_from_wayland.patch - blocks VT switching from the wayland side. in some compositors, this can be disabled normally, but for example, for plasma, I did not find such a way. please note that this patch is implemented VERY STRANGELY inside and, I would say, incorrectly. it works with kde plasma by blocking VT switching for its session, but I'm not sure that it will work fine with other compositors (if your compositor supports disabling VT switching normally, then use its setting and not this patch) the algorithm of this patch is as follows: it prevents switching via ioctl from tty being in "graphical" mode, but it allows you to do this once from the moment the kernel is started (without this, the cursor remains on a black background, apparently switching from DM to DE but I'm not sure) anyway, this terrible solution works in cases where sddm+plasmawayland is used. the correct solution would be to patch the wayland compositor rather than the core
* disable_keyboard_echo_by_default.patch - disables echo tty mode in the kernel by default. this is necessary because plymouth starts up 100-300 milliseconds after the kernel is started, and for some time (let's say the minimum time) you can print on an empty screen and the letters will appear. of course, it does not interpret commands, but it still violates my principles of a COMPLETELY clean boot screen (COMPLETELY slient boot) and I want to make an apple-level slient boot
* disable_tty_control_flow.patch - disables control flow at the kernel level (ctrl+s, ctrl+q). it is important on embedded because it sometimes allows the user to interrupt the boot or initialization process.
* disable_tty_signals.patch - disables sending SIGINT, SIGQUIT, SIGTSTP signals at the kernel level. it is important on embedded because it sometimes allows the user to interrupt the boot or initialization process.
* disable_reboot_and_shutdown_emerg_messages.patch - disables emerg messages from the kernel about rebooting and shutting down. they infuriate me because they cannot be disabled with regular kernel arguments.
* make_all_emerg_messages_with_alert_loglevel.patch - this patch causes all EMERG messages to be output as an alert. which allows you to prevent their output using: quiet loglevel=0
* driver_fakeconsole.patch - adds a fake console driver that can be specified as console=fakeconsole so that the kernel output goes "nowhere"

## recommended patch sets for different tasks

### embedded device
* disable_vt_swithing_from_keyboard.patch
* disable_sysrq.patch
* disable_cad.patch
* disable_tty_control_flow.patch
* disable_tty_signals.patch
* disable_keyboard_echo_by_default.patch
* disable_reboot_and_shutdown_emerg_messages.patch
* make_all_emerg_messages_with_alert_loglevel.patch

### desktop without VT
* disable_vt_swithing_from_keyboard.patch
* disable_vt_swithing_from_wayland.patch
* disable_tty_control_flow.patch
* disable_tty_signals.patch
* disable_keyboard_echo_by_default.patch
* disable_printk.patch

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

## disabling VT switching at the wayland level of the "weston" composer
### in "weston.ini", set "vt-switching" to false in the "keyboard" section
```
[keyboard]
vt-switching=false
```

## warnings
* if you disabled VT switching in the kernel through the "disable_vt_swithing_from_keyboard.patch" patch but did not disable it at the graphics session level, then you may get a situation where it is possible to switch from the graphics session but you cannot switch back. you also need to disable this at the X11 level (it's always easy) or wayland (depends on the compositor, and in some cases you need to patch the compositor or the core with the "disable_vt_swithing_from_wayland.patch" patch)
* plymouth handle the ESC key to show the boot log, this is not allowed on embedded devices, and that's why I made this patch: https://github.com/igorkll/embedded-plymouth

## create kernel patch guide

### if you make patches in a directory created by syslbuild, all files and directories will be owned by root, so change this
1. sudo chmod -R 755 .
2. sudo chown -R $USER: .

### if the .git directory doesn't exist yet
1. download and unpack the kernel source code
2. open the terminal in this directory
3. command: git init
4. command: git add .
5. command: git commit -m "Initial commit"

### make patch
1. make you changes
2. command: git add .
3. command: git commit -m "kernel patch"
4. export: git diff HEAD~1 HEAD > example.patch
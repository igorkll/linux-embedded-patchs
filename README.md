# linux-embedded-patches
a set of patches for embedded linux-based systems

## pathes
* disable_vt_swithing_from_keyboard.patch

## recommendations
* mitigations=off - you can add this to the kernel parameter to disable protection against CPU hardware vulnerabilities like Spectre and Meltdown. this will improve performance on older processors like intel atom. DO NOT DO THIS ON SECURITY-CRITICAL DEVICES
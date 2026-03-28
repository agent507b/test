 Display backlight control issue under Linux (Fedora 43)

Device information:

Model: Lenovo Legion 5 15IPH11 (Product 83RW)
BIOS version: T4CN38WW (Date: 01/05/2026)
CPU: Intel Core Ultra 9 386H (Panther Lake architecture)
GPU: Intel Xe (integrated) / Nvidia RTX series (discrete)
OS: Fedora Linux 43 / KDE Plasma 6.6.3
Kernel: 6.19.7-200.fc43.x86_64
Problem description:
Screen brightness remains stuck at 100% (maximum intensity) regardless of the user’s action. The KDE Plasma brightness slider and the physical hotkeys both update the software state, but the physical panel output does not change. This causes significant eye strain and prevents use in low-light environments.

 
Detailed troubleshooting performed:
1. Driver isolation and conflict resolution
Several conflicting modules identified: nvidia_wmi_ec_backlight, ideapad_laptop, and xe.

Action: Blacklisted nvidia_wmi_ec_backlight and ideapad_laptop to isolate the native Intel controller.
Result: The system successfully defaulted to the native Intel Xe driver, but physical brightness remained unresponsive.
2. Kernel parameter testing (GRUB)
Tried various ACPI/Video abstractions to force hardware sync:

acpi_backlight=vendor / video / native
i915.enable_dpcd_backlight=1 / 0
i915.invert_brightness=1
Result: No parameter triggered a hardware PWM response from the backlight controller.
3. Hardware interface verification (Critical observation)
The system correctly exposes the interface at: /sys/class/backlight/intel_backlight/.

Maximum brightness: 192,000.
Proof of desynchronization: Moving the brightness slider correctly updates the value in the brightness file (down to “1”). However, the physical panel remains at a 100% duty cycle. This indicates the OS is sending the signal, but the hardware/firmware is ignoring the PWM or DPCD command.
4. DDC/CI and software tools
ddcutil detect returns: “This monitor does not support DDC/CI. (I2C slave address x37 is unresponsive)”.
Software-level dimming (xrandr/gamma) does not resolve the hardware-level backlight lock.
 
Technical note: Secure Boot disabled (OS-less hardware)
Current firmware state: The system is operating with Secure Boot DISABLED (device purchased as No-OS/FreeDOS model). This results in an “Unsigned” boot environment that may block critical ACPI power-management exchanges required by the Panther Lake display engine.

Observed impact:

Backlight power path: In this Secure Boot–off state, the embedded controller (EC) on the 83RW appears to bypass the PWM signal from the Intel Xe driver.
VBIOS handoff failure: Switching to Discrete Graphics mode with Secure Boot off leads to immediate display corruption and an inaccessible user interface. This suggests the Nvidia VBIOS cannot negotiate eDP link training nor backlight routing without the secure exchange typically expected by the T4CN38WW firmware.
 
Final verdict / Evidence of a firmware mapping failure:
Complete driver isolation: Verified via /proc/cmdline (all non-Intel modules blacklisted).
Software–hardware desynchronization: Writing “1” to /sys/class/backlight/intel_backlight/brightness does update the system state, but the physical PWM duty cycle remains locked at 100% on the eDP panel.
Conclusion: This confirms that the Intel Xe driver is writing to a register that the 83RW firmware does not route to the panel’s backlight controller. This is a VBT (Video BIOS Table) or EC mapping error specific to model 83RW under Linux.
Requested action: Please escalate this report to the BIOS/Engineering team. A UEFI update is needed to properly expose the ACPI Backlight methods (_BCL, _BCM) and fix VBIOS handoff stability for the Panther Lake platform on the Legion 5 (83RW).

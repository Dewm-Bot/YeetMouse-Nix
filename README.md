!NOTE]
> I found it really inefficient relying on nix based configuration for editing yeetmouse, especially across multiple devices and machines. This repo consists of small patches to allow yeetmouse to operate properly within the NixOS ecosystem!

# Description
**YeetMouse** is a kernel module that allows for fast and customizable mouse acceleration.
I created **YeetMouse** to build upon the great work of *Klaus Zipfel's* **[LeetMouse](https://github.com/systemofapwne/leetmouse)** and add features such as `Jump` (at first it was mostly it, the Jump acceleration mode).
But it quickly grew into something bigger as I decided to add a GUI to it for better control.

*Basic functions presentation:*
![Basic functions presentation](media/YeetMouseGithub.gif)

# Installation & Uninstallation

## Preparation
* Make sure you have the required build tools (`build-essentials` on Debian), `dkms`, and the `linux-headers` for your currently installed kernel.
* The GUI also requires OpenGL and GLFW development files to build.
* The installation script should be run as a **regular user** and will use `sudo` internally when needed.

### Installation
The easiest way to install YeetMouse is to use the provided `./install.sh` script:

```

```text
File saved successfully as yeetmouse-readme.md

```sh
git clone [https://github.com/AndyFilter/YeetMouse.git](https://github.com/AndyFilter/YeetMouse.git)
cd YeetMouse
./install.sh

```

The installer always installs the driver, `yeetmousectl`, config file, service, and uninstaller.
The GUI is built and installed **optionally** - if its dependencies are present.

If your user is added to the `yeetmouse` group during installation (should be by default), you must start a new login session before the GUI can access the driver.
Logging out and back in is usually enough; on some systems a reboot may be required.

## GUI dependencies

The GUI requires:

* OpenGL
* GLFW3 development files
* `pkg-config`

If these are missing, the main installation still succeeds, but the GUI is skipped.
Once the dependencies are installed, you can install the GUI separately with:

```sh
sudo make install_gui

```

## Configuration model

YeetMouse uses a persistent configuration file:

```sh
/etc/yeetmouse/settings.conf

```

Runtime configuration is handled by `yeetmousectl`:

```sh
yeetmousectl apply /etc/yeetmouse.conf
yeetmousectl dump
yeetmousectl save my-config.conf

```

On systemd-based systems, the installed `yeetmouse.service` automatically applies `/etc/yeetmouse/settings.conf` during boot.

## Uninstallation

If YeetMouse was installed system-wide, use:

```sh
sudo yeetmouse-uninstall

```

The repository `./uninstall.sh` script will prefer the installed uninstaller if it exists, and otherwise fall back to the repository copy.

## Running the GUI

### Permissions

The GUI is intended to run **without sudo**.
Access to the driver parameters is granted through the `yeetmouse` group.

After installation, your user may be added to that group automatically. This change applies only to **new login sessions**.
If the GUI cannot access the driver right after installation:

* log out and log back in,
* or reboot if your desktop session does not fully refresh group membership.

### Prerequisites

The GUI requires:

* OpenGL
* GLFW3 development files

Exact package names depend on the distribution. On Debian/Ubuntu systems, this usually means packages such as:

```sh
sudo apt install libglfw3 libglfw3-dev

```

### Building

To build only the GUI:

```sh
cd gui/
make

```

### Starting

After installation, simply run:

```sh
yeetmouse

```

The GUI should not need `sudo`, provided your user session already has the `yeetmouse` group.

## Arch

For Arch based systems, a `PKGBUILD` has been written for seamless integration into pacman.

**Installation**

```sh
make pkgarch
sudo pacman -U yeetmouse*.zst
sudo modprobe yeetmouse
# Optional, if using YeetMouseGUI or the systemd service for local config at /etc/yeetmouse/settings.conf
sudo systemctl enable --now yeetmouse.service
sudo usermod -a -G yeetmouse <your user name>
# Now log out and log in or alternatively restart your system

```

**Uninstallation**

```sh
sudo pacman -R yeetmouse yeetmouse-driver-dkms yeetmouse-tools yeetmouse-gui

```

## Ubuntu (tested on 24.04 and 20.04)

Use the installation script as a regular user:

```sh
./install.sh

```

After installation, start a new login session before testing the GUI if group permissions have just been granted.

To uninstall:

```sh
sudo yeetmouse-uninstall

```

## NixOS

Please refer to [NixOS instructions](https://www.google.com/search?q=nix/).

## Other distros

Other distributions' package-managers are not yet fully supported, so manual installation may be required.

### Installation

The easiest method is:

```sh
./install.sh

```

This installs:

* the DKMS source tree,
* the kernel module through DKMS,
* `yeetmousectl`,
* the persistent config file in `/etc/yeetmouse/settings.conf`,
* the systemd service (if systemd is present),
* and the GUI files if built/installed through the project Makefile.

If systemd is not present, you can still apply the config manually at boot using your init system:

```sh
/usr/bin/yeetmousectl apply /etc/yeetmouse/settings.conf

```

**Uninstallation**
Use the installed uninstaller:

```sh
sudo yeetmouse-uninstall

```

The uninstaller disables the service, removes installed files, removes the DKMS module, and can optionally remove `/etc/yeetmouse/settings.conf`.

# Manual compile & insmod

If you want to compile this module only for testing purposes or development, you do not need to install the whole package to your system.

Compile the module, remove previously loaded modules and insert it:

```sh
make clean && make
sudo rmmod yeetmouse
sudo insmod ./driver/yeetmouse.ko

```

# Command line usage

## Applying a config

Apply a persistent config file to the running driver:

```sh
yeetmousectl apply /etc/yeetmouse/settings.conf

```

## Dumping current runtime values

Print the currently active driver parameters:

```sh
yeetmousectl dump

```

## Saving current runtime values

Save the currently active settings to a file:

```sh
yeetmousectl save my-config.conf

```

# FAQ

## How to set custom parameter value?

* Ctrl + Left Click on the parameter box to start inputting the values manually.

## How are settings preserved between reboots?

* Settings are stored in `/etc/yeetmouse/settings.conf`.
* To apply them immediately:
```sh
yeetmousectl apply /etc/yeetmouse/settings.conf

```


* On systemd systems, the installed `yeetmouse.service` automatically applies them on boot.
* To export the currently active settings:
```sh
yeetmousectl save my-config.conf

```



## Why does the GUI say it is running without yeetmouse group privileges?

* Your user was likely added to the `yeetmouse` group during installation.
* Group changes usually apply only to a new login session.
* Log out and log back in, or reboot if necessary.

## Mouse feels off (too fast / slow)

* On some distros (for example Ubuntu 20.04) the system adds an additional sensitivity on top of the driver.
To combat this, configure the desktop sensitivity appropriately.
For Ubuntu 20.04 users, one useful value is:
```sh
gsettings set org.gnome.desktop.peripherals.mouse speed -0.666

```



## How do I convert my RawAccel settings?

* For the simple modes like *Linear, Classic, Power* just use the RawAccel's values (same for *Jump*).
* For *Motivity* and *Natural*, you're out of luck for now. Motivity is implemented, but it does not support `Gain`. Natural on the other hand is not implemented, and not planned as of for now.
* LuT (Look up Table) is just what you put in it, there is no difference between YeetMouse and RawAccel.
* Keep in mind that the names are not 1:1 for every parameter.
* To check how your new curve compares to RawAccel's, just take a screenshot of RawAccel with your curve and compare the two.

## How to apply LUT curves as velocity (and use the EPP (Windows default acceleration) curve)?

* YeetMouse currently doesn't support applying the LUT as velocity.
* To convert a velocity LUT into a sensitivity (normal) LUT, use the Python script from this comment: https://github.com/AndyFilter/YeetMouse/issues/67#issuecomment-3620653542.
* For the EPP (Enhance Pointer Precision) curve specifically, use the config from this comment: https://github.com/AndyFilter/YeetMouse/issues/67#issuecomment-3620653542.

## More questions

If you have any issues / questions regarding this project, feel free to ask them either directly through GitHub or on the [Discord server](https://discord.gg/z2qCD2zNTa).

# Fixed-Point Performance Analysis

*Functions Performance Comparison*


| Instruction | Fixed-Point / FPU | Mop/s | ns/op | Clock cycles/op |
| --- | --- | --- | --- | --- |
| **Multiplication** | Fixed-Point 64 | 542.905367 | 1.911 | 7.029038 |
|  | Fixed-Point 64 (128bit) | 540.682695 | 1.913 | 7.012462 |
|  | FPU (double) | 788.524105 | 1.29 | 4.722532 |
| **Division** | Fixed-Point 64 (Precise) | 91.446419 | 11.299 | 41.756461 |
|  | Fixed-Point 64 (128bit) | 203.819151 | 5.097 | 18.797924 |
|  | FPU (double) | 188.035704 | 5.392 | 19.879064 |
| **Exponent** | Fixed-Point 64 | 66.550845 | 15.561 | 57.525454 |
|  | Fixed-Point 64 (Fast) | 92.775366 | 11.285 | 41.702182 |
|  | FPU (double) | 116.396443 | 8.741 | 32.276506 |
| **Sqrt** | Fixed-Point 64 (Precise) | 18.059895 | 57.307 | 211.97892 |
|  | Fixed-Point 64 | 64.558792 | 15.675 | 57.956097 |
|  | FPU (double) | 133.474534 | 7.9 | 29.179384 |
| **Pow** | Fixed-Point 64 | 31.81294 | 32.221 | 119.111214 |
|  | Fixed-Point 64 (Fast) | 40.524527 | 26.043 | 96.310556 |
|  | FPU (double) | 77.804544 | 17.113 | 63.251944 |
| **Log** | Fixed-Point 64 | 51.117073 | 21.033 | 77.768302 |
|  | Fixed-Point 64 (Fast) | 61.341951 | 16.638 | 61.497848 |
|  | FPU (double) | 53.326065 | 19.876 | 73.491065 |

#### *More in-depth performance and precision analysis can be found [here](Performance.md).*

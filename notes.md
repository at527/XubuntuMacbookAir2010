# Macbook Air 2010 (3,2) Xubuntu 20 Install

## Basic Install

Everything seems to work ok with the standard install from a Xubuntu USB.
However, proprietary drivers are recommended.
While the Nouveau drivers work, they run very hot.

1. Install using flashed usb, include proprietary drivers
2. apt update and upgrade

    ```bash
    sudo apt update
    sudo apt upgrade
    ```

## askUbuntu Nvidia Install Resource

This askUbuntu answer describes the procedure to enable Nvidia drivers on
Macbooks. The next section is a summary of the page.

<https://askubuntu.com/a/613573>

The page is titled "Proprietary NVidia drivers with EFI on Mac, to prevent
overheating" :
<https://askubuntu.com/questions/264247/proprietary-nvidia-drivers-with-efi-on-mac-to-prevent-overheating>

## Preparing for Installing Nvidia Drivers (from askUbuntu above)

1. Install Ubuntu in UEFI mode. Do not install Nvidia drivers yet.
2. Find PCI-E bus identifiers with the following command.

    ```bash
    sudo lshw -businfo -class bridge -class display
    ```

    Output:

    ```bash
    Bus info          Device     Class          Description
    =======================================================
    pci@0000:00:00.0             bridge         MCP89 HOST Bridge
    pci@0000:00:03.0             bridge         MCP89 LPC Bridge
    pci@0000:00:15.0             bridge         NVIDIA Corporation
    pci@0000:00:17.0             bridge         MCP89 PCI Express Bridge
    pci@0000:02:00.0             display        MCP89 [GeForce 320M]

    ```

    Note the Nvidia (GeForce) driver Bus info (Here it's 02:00.0)

    and the PCI bridge above it (Here it's 00:17.0)

    They may be different on your machine.

3. Create a GRUB script for setting the PCI-E registers during boot

    Use the following command to create and edit the grub file.

    ```bash
    sudo nano /etc/grub.d/01_enable_vga.conf
    ```

    Then paste the following into the file. Be sure to use your PCI-E bus
    info found in step 2.

    ```bash
    cat << EOF
    setpci -s "00:17.0" 3e.b=8
    setpci -s "02:00.0" 04.b=7
    EOF
    ```

4. Update your grub with the following TWO commands

    ```bash
    sudo chmod 755 /etc/grub.d/01_enable_vga.conf
    ```
    
    ```bash
    sudo update-grub
    ```


    ```bash
    sudo setpci -s "02:00.0" 04.b
    ```

5. Reboot your machine

6. Verify the changes were made.
    
    ```bash
    sudo setpci -s "02:00.0" 04.b
    ```
    
    ```bash
    sudo setpci -s "00:17.0" 3e.b
    ```
    
7. Continue to install nVidia drivers.

## Install Nvidia Drivers

1. OPTION 1: use the "Additional Drivers" program to install Nvidia drivers.
2. Option 2: install using the command line

    ```bash
    ubuntu-drivers -install
    ```

3. rebooting will be required for Nvidia drivers to be enabled. This can be done
after the next step.

## Enabling Brightness Control when using Nvidia Drivers

The following comes from
<https://askubuntu.com/questions/76081/brightness-not-working-after-installing-nvidia-driver>
titled "Brightness not working after installing NVIDIA driver"

1. Create and edit the xorg configuration

    ```bash
    sudo nano /usr/share/X11/xorg.conf.d/10-nvidia-brightness.conf
    ```

2. Paste the following into the file above

    ```txt
    Section "Device"
        Identifier     "Device0"
        Driver         "nvidia"
        VendorName     "NVIDIA Corporation"
        Option         "RegistryDwords" "EnableBrightnessControl=1"
    EndSection
    ```

3. Install xbacklight with the following command

    ```bash
    sudo apt install xbacklight
    ```

4. reboot

5. To set the brightness using the following command

    ```bash
    xbacklight -set 50
    ```

    replacing 50 with the desired brightness percentage.

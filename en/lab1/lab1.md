# 1. Lab: Virtualization and network setup

## Instructions

1. Install any virtualization hypervisor, such as [VirtualBox](https://www.virtualbox.org/).
2. Install any distribution of the Linux operating system, such as [Debian](https://www.debian.org/).
3. Use cloning of the virtual computer to set up another one.
4. Make sure that virtual computers can connect over a network using NAT, Internal Network, and Bridged Adapter.

## Additional information

Commonly used hypervisors:

- [Oracle VirtualBox](https://www.virtualbox.org/)
- [VMware Workstation Player](https://www.vmware.com/products/workstation-player.html)
- [GNS3](https://www.gns3.com/) 
- [QEMU](https://www.qemu.org/)
- [Hyper-V](https://learn.microsoft.com/en-us/virtualization/hyper-v-on-windows/)

Important reserved address blocks on an Internet network:

| Address block  | Interval                      | No. of addresses | Description          |
|----------------|-------------------------------|------------------|----------------------|
| 10.0.0.0/8     | 10.0.0.0 - 10.255.255.255     | 16.777.216       | Local addresses      |
| 127.0.0.0/8    | 127.0.0.0 - 127.255.255.255   | 16.777.216       | Loopback addresses   |
| 169.254.0.0/16 | 169.254.0.0 - 169.254.255.255 | 65.536           | Link-Local addresses |
| 172.16.0.0/12  | 172.16.0.0 - 172.31.255.255   | 1.048.576        | Local addresses      |
| 192.168.0.0/16 | 192.168.0.0 - 192.168.255.255 | 65.536           | Local addresses      |
| 224.0.0.0/4    | 224.0.0.0 - 239.255.255.255   | 268435456        | Multicast addresses  |

The [`ip`](https://linux.die.net/man/8/ip) command allows us to display and manage network devices, network adapters, tunnels, and routing.

The [`ping`](https://linux.die.net/man/8/ping) command allows us to send ICMP ECHO_REQUEST packets to the selected network device based on its IP address.

The [`apt-get`](https://linux.die.net/man/8/apt-get) command is a package manager that allows us to install, modify, and delete programs, and search software through accessible repositories.

The [`su`](https://man7.org/linux/man-pages/man1/su.1.html) command allows us to change the user by running commands in the shell.

The [`mkdir`](https://man7.org/linux/man-pages/man1/mkdir.1.html) command is used to create new directories.

The [`mount`](https://man7.org/linux/man-pages/man8/mount.8.html) command allows us to mount a file system to the operating system.

The [`cd`](https://man7.org/linux/man-pages/man1/cd.1p.html) command  allows us to change the working directory.

The [`sh`](https://man7.org/linux/man-pages/man1/sh.1p.html) command executes Standard Common Language (Shell) commands.

The [`shutdown`](https://man7.org/linux/man-pages/man8/shutdown.8.html) command allows us to shutdown or restart the operating system.

The [`ls`](https://man7.org/linux/man-pages/man1/ls.1.html) command prints the contents of any directory.

## Detailed instructions

### 1. Task

On Windows and macOS, install the VirtualBox hypervisor by downloading the installation files from the official [website](https://www.virtualbox.org/) and following the instructions during installation. On Linux, however, install VirtualBox from the repositories of your distribution. On distributions based on Debian, we do this with the package manager "apt".

    apt install virtualbox

### 2. Task

After a successful installation, open VirtualBox and select `New ...` from the menu above to open the wizard for creating new virtual computers. In the first window, specify the `Name`, `Machine folder`, `Type` and `Version` of the new virtual computer. For example, we choose `Debian` as the name, `/home/USER/VirtualBox VMs` as the folder in which the virtual machine will be stored, `Linux` as the type, and `Debian (64-bit)` as the version. Then press the `Next` button.

![The first window of the wizard allows us to set basic properties.](images/lab1-vbox1.png)

In the second window, we determine the amount of memory (RAM) to be allocated to our virtual computer. For example, we specify `4096 MB` of memory and press the `Next` button.

![The second window of the wizard allows us to set the amount of memory.](images/lab1-vbox2.png)

In the third window, we specify the virtual hard disk, namely, we can choose between three options: `Do not add a virtual hard disk`, `Create a virtual hard disk now` and `Use an existing virtual hard disk file`. For example, we choose to create a new virtual hard disk for our new virtual computer and press the `Create` button.

![The third window of the wizard allows us to set up the virtual hard disk.](images/lab1-vbox3.png)

In the fourth window, we specify the format of the virtual hard disk that will be used by our virtual computer. We have three options: `VDI (VirtualBox Disk Image)`, `VHD (Virtual Hard Disk)` and `VMDK (Virtual Machine Disk)` . For example, we select the `VDI` format and press the `Next` button.

![The fourth window of the wizard allows us to choose the format of the virtual hard disk.](images/lab1-vbox4.png)

In the fifth window, we define the method of allocating space for the virtual hard disk, which can be `Dynamically allocated` or `Fixed size`. For example, we select dynamic allocation and press the `Next` button.

![The fifth window of the wizard allows us to choose between the dynamic and static allocation of space to the virtual hard disk.](images/lab1-vbox5.png)

In the sixth window, we specify where in the file system our new virtual hard disk will be created and what its maximum size will be. For example, we choose `/home/USER/VirtualBox VMs/Debian/Debian.vdi` for the location of our virtual disk and set its maximum size to `200.00 GB`. Then click the `Create` button.

![The sixth window of the wizard allows us to choose the format of the virtual hard disk.](images/lab1-vbox6.png)

If we successfully created our `Debian` virtual machine, it appeared in the column on the left side of the VirtualBox window. In the column on the right side, detailed settings are displayed, which we specified during creation or were set to default values automatically.

![Our new virtual machine with associated parameters in the main window.](images/lab1-vbox7.png)

Before the first launch, we can set the parameters of our virtual machine even more precisely by clicking on the `Settings ...` button in the menu above. For example, under the `System\Processor` tab, we can set the number of processor cores to assign to our virtual computer.

![Settings menu `System\Processor` where we can set the number of processor cores that our virtual computer can use.](images/lab1-vbox8.png)

We have now finished setting up our new virtual machine and can start it by pressing the `Start` button. At the first start, a window will automatically appear for selecting the installation image of the operating system we want to install (e.g. [Debian](https://cdimage.debian.org/cdimage/unofficial/non-free/cd-including-firmware/11.5.0+nonfree/amd64/iso-cd/)). The installation image of the operating system located on the disk can be selected by clicking on the button with the icon of the yellow folder with a green arrow. Otherwise, we can get to the selection of the installation image of the operating system if we go to the menu `Devices\Optical drives\Select disk file...` in the window of our virtual computer.

![Window for choosing the installation image of the operating system.](images/lab1-vbox15.png)

![A window to select the installation image of the operating system from our local disk.](images/lab1-vbox16.png)

Now a bootloader window appears, in which we can choose between `Graphical installation` and `Installation`. For example, we decide on `Graphical installation` and press the `Enter` key on the keyboard.

![Bootloader window for installing the Debian operating system.](images/lab1-debian1.png)

In the next step, you can choose any language in which the installation will be performed. For example, select `English` or `Slovenian` and press the `Continue` button.

![Choose the language in which the installation will be performed.](images/lab1-debian2.png)

Now we choose where we are. For example, we get to Slovenia by selecting `other\Europe\Slovenia` and pressing the `Continue` button.

![Choosing our location.](images/lab1-debian3.png)

Then we select the localization. For example, we select `United States - en_US.UTF-8` and press the `Continue` button.

![Select the locale.](images/lab1-debian4.png)

Now let's choose the keyboard layout. For example, select `Slovenian` and press the `Continue` button.

![Selecting the keyboard layout.](images/lab1-debian5.png)

In the next step, we define the name of our virtual computer. We can just use the default name `debian` and press the `Continue` button.

![Enter the name of the virtual machine.](images/lab1-debian6.png)

We can then select the network domain that our virtual machine will be a part of. We don't need it, for now, so we leave the field empty and press the `Continue` button.

![Enter network domain.](images/lab1-debian7.png)

Now enter any administrator password (`root`) in both fields and press the `Continue` button.

![Enter the administrator password.](images/lab1-debian8.png)

Next is the selection of the name of the new user, for which we can choose any name and press the `Continue` button.

![Enter the name of the new user.](images/lab1-debian9.png)

Then we define any user name of the new user and press the `Continue` button.

![Enter the username of the new user.](images/lab1-debian10.png)

In the end, enter any password of the new user in both fields and press the `Continue` button.

![Enter the new user's password.](images/lab1-debian11.png)

In the next step, we select the disks and set the partitions, where we can choose the default setting `Guided - use entire disk`, which will use the entire disk, and press the `Continue` button.

![Choosing how to select disks and set partitions.](images/lab1-debian12.png)

Now we select the disk on which we want to install our operating system. In our case, we select the only virtual disk we have and press the `Continue` button.

![Selecting the disk on which we want to install our virtual disk.](images/lab1-debian13.png)

The following is the determination of the partitions on our disk, where one partition is sufficient for our needs, in which we save everything `All files in one partition (recommended for new users)` and press the `Continue` button.

![Determining the partitions on our disk.](images/lab1-debian14.png)

Then the currently defined partitions that we will use and the option to manually set partitions are displayed. The default setting defines two partitions, namely: a primary partition for storing data with the EXT4 file system ([Fourth extended file system](https://en.wikipedia.org/wiki/Ext4)) and a logical partition for expanding the system memory. We confirm the proposed partitions by selecting `Finish partitioning and write changes to disk` and press the `Continue` button.

![Inspection of default disk partitions and manual setting of only these.](images/lab1-debian15.png)

Confirmation of the selected sections follows as the partitions and data that were previously written to the disk are deleted at this point. We select `Yes` to agree to the change and press the `Continue` button, thus starting the installation of the operating system.

![Confirmation of selected partitions.](images/lab1-debian16.png)

During the installation, we refuse the installation of additional programs from other media by selecting the `No` option and pressing the `Continue` button.

![Refuse the installation of additional programs from other media.](images/lab1-debian17.png)

Now we select the country in which we want to have a server that will be used by the package manager to install programs. A good practice is to select a country close to our location, for example, `Slovenia` and press the `Continue` button.

![Choose the country in which we want to have a server that will be used by the package manager to install programs.](images/lab1-debian18.png)

In the next step, we choose between the servers offered in the selected country for installing the programs. We can leave the default option `deb.debian.org` selected and press the `Continue` button.

![Choosing a server in the selected country to install programs.](images/lab1-debian19.png)

The installation also allows us to set up a proxy server for Internet access, but we don't need it, so we leave the field empty and press the `Continue` button.

![Proxy server setup.](images/lab1-debian20.png)

Next, disable the sending of anonymous data about our operating system used to the Debian servers by selecting the `No` option and pressing the `Continue` button.

![Setting the anonymous operating system usage data.](images/lab1-debian21.png)

In the next step, we select the graphical environment we want to use and possibly some additional software. For example, we will use the default values ​​where we have the following options `Debian desktop environment`, `GNOME` and `standard system utilities` selected and then press the `Continue` button to continue the installation.

![Choice of graphic environment and of additional software.](images/lab1-debian22.png)

Now let's install the bootloader, which takes care of starting the operating system. We select the `Yes` option and press the `Continue` button.

![Selecting the bootloader installation.](images/lab1-debian23.png)

Then select the disk on which to install the bootloader. We select our single installation disk by selecting `/dev/sda (HARD_DISK_ID)` and pressing the `Continue` button.

![Selecting the disk on which to install the bootloader.](images/lab1-debian24.png)

We have successfully installed the Debian operating system and now restart our virtual computer by pressing the `Continue` button.

![Choosing to reboot after installation is complete.](images/lab1-debian25.png)

After restarting, log into the operating system and test the operation of the operating system and the web.

In addition, we can also install `VirtualBox Guest Additions`, which allow us to automatically adjust the desktop resolution to the current window size, copy from a physical computer to a virtual computer and vice versa, and a bunch of other useful functionalities. We start the installation by updating the repositories of our operating system and then installing the packages we need before starting the installation.

    su -

    apt update

    apt install build-essential dkms linux-headers-$(uname -r)

Now insert the `VirtualBox Guest Additions` installation image by selecting `Devices\Insert Guest Additions CD Image ...` in the virtual machine window. If we don't have the installation image yet, VirtualBox offers us to download it by confirming the download from the web by pressing the `Download` button twice and then the `Insert` button once more.

![First confirmation of the download of the installation image `VirtualBox Guest Additions`.](images/lab1-guest1.png)

![Second confirmation of the download of the installation image `VirtualBox Guest Additions`.](images/lab1-guest2.png)

![Selecting the `VirtualBox Guest Additions' installation image.](images/lab1-guest3.png)

In case of successful insertion of the `VirtualBox Guest Additions` installation image, we now have to mount it into the operating system.

    mkdir -p /mnt/cdrom

    mount /dev/cdrom /mnt/cdrom

    cd /mnt/cdrom

Then we run the installation program and after successful installation, we restart our virtual computer.

    sh ./VBoxLinuxAdditions.run --nox11

    shutdown -r now

### 3. Task

The easiest way to get a usable copy of a virtual computer is through the cloning process. We do it by right-clicking on the virtual computer in the left column of the main VirtualBox window to open an additional menu and click on the `Clone ...` option. A window for selecting cloning settings opens. You can specify `Name`, `Path`, `MAC Address Policy` and `Additional options`. `MAC Address Policy` allows us to `Include all network adapter MAC addresses`, `Include only NAT network adapter MAC addresses` or `Generate new MAC addresses for all network adapters`. For example, we choose `Debian Clone` as the name, we choose `/home/USER/VirtualBox VMs` as the folder where the virtual machine will be stored, we choose to create new MAC addresses for all network cards regardless of the settings and we leave the additional options unchecked. Press the `Next` button.

![Selecting clone settings.](images/lab1-clone1.png)

In the next step, we choose between the `Full clone` and `Linked clone` cloning modes. A full clone duplicates all of a virtual machine's files, while a linked clone keeps the same virtual machine files in shared files. The other files, which are different, are stored in files that are specific to each virtual computer. A linked clone takes up less disk space compared to a full clone. On the other hand, in a full clone, the clone provides faster access to files because they are written independently on the disk, while in a linked clone, several virtual machines access the same files on the disk at the same time, which can slow down access. Linked clones are dependent on each other, which can lead to unexpected changes in shared data and, as a result, corrupted files. In our example, we select the `Full Clone` option and press the `Clone` button to start cloning.

![Choosing the cloning method.](images/lab1-clone2.png)

### 4. Task

Below we present four possible network settings that at least partially allow us to connect via the network between several virtual computers.

Under the `Network` tab, we can assign up to four network adapters to our virtual computer. For example, we set the `Adapter 1` of all our VMs to `NAT`, `Internal Network`, `Bridged Adapter` or `NAT Network`.

The `NAT` option sets us up to translate IP addresses from the network of our physical computer to the network of our virtual computer by changing the IP addresses in the IP packet headers ([NAT - Network address translation](https://en.wikipedia.org/wiki/Network_address_translation) ). In this case, our virtual computer has its network by default and cannot directly access other virtual computers, our physical computer and the physical network. Conditionally, we can provide access to specific applications by forwarding specific ports. 

![The `Network` setting tab where we can set that `Interface 1` uses a `NAT` network.](images/lab1-vbox9.png)

We set the network port forwarding by clicking on the `Advanced` tab and then clicking on the `Port forwarding` button.

![The `Advanced` setting tab where we can set `Port forwarding` on the `NAT` network.](images/lab1-vbox19.png)

Each port is forwarded by setting the `Name`, `Protocol`, `Host IP`, `Host Port`, `Guest IP` and `Guest Port`.

![Menu for setting port forwarding on the `NAT` network.](images/lab1-vbox20.png)

Accessing a virtual computer with a secure command shell (SSH) can be carried out via the `NAT` network by port forwarding. We install an SSH server on the virtual machine.

    apt install openssh-server

We now forward the port by setting the `Name` to any name, `Protocol` to `TCP`, `Host IP` to the IP address of our physical computer, `Host port` to any port not used by other applications, such as `2222`, `Guest IP` to the IP address of the virtual computer network adapter connected to the `NAT` network and `Guest port` to port `22`, used by default by the SSH server.

![Set port forwarding on the 'NAT' network for using SSH.](images/lab1-vbox21.png)

We test the forwarded port by connecting from the physical computer's command line using SSH to the virtual machine and specifying the network port we forwarded on the physical computer, the user who will be used to log in to the virtual machine and the IP address of the physical computer.

    ssh -p 2222 USER@IP_OF_THE_PHYSICAL_COMPUTER

The `Internal network` option connects several virtual computers to a network without a router, where they have to take care of establishing IP addresses themselves. The virtual computers can see each other after the network is set up correctly, but cannot access the web or other networks unless one virtual computer also acts as a router.

![The `Network` settings tab where we can set that `Adapter 1` uses a common `Internal network` named `intnet`.](images/lab1-vbox17.png)

The IP address can be set on both virtual computers with the `ip` command, where we correctly set the IP address, network mask and network adapter to enable connectivity.

On the first virtual computer with the `enp0s3` network adapter, set the network IP address for example to `192.168.1.1/24`.

    ip addr add 192.168.1.1/24 dev enp0s3

On the second virtual computer with the network adapter `enp0s3`, we set the network IP address in the same network as the first virtual computer, for example, to `192.168.1.2/24`.

    ip addr add 192.168.1.2/24 dev enp0s3

The `Bridged adapter` option enables us to share the network adapter of our physical computer with the virtual computer so that both obtain an IP address from the physical network. Our virtual machine acts as if it is connected to a physical network and by default has access to all computers on that network.

![Settings tab `Network` where we can set `Adapter 1` to use `Bridged adapter`.](images/lab1-vbox10.png)

The option `NAT network` also sets up the translation of IP addresses from the network of our physical computer to a virtual network to which several virtual computers can connect. In this case, by default, our virtual computer has access to all virtual computers connected to the NAT network, which is defined by the name, but it does not have direct access via the network to our physical computer and the physical network. We can conditionally enable access to the physical network for certain applications by opening certain network ports (port forwarding).

![Settings tab `Network` where we can set `Adapter 1` to use a common `NAT network`.](images/lab1-vbox11.png)

If we select the `NAT network` option, this will not work for us by default, as we have to create it beforehand. We do this by selecting the `File\Options ...` menu in the main VirtualBox window. Under the `Network` tab, click on the button with the network adapter icon and the green plus symbol and create a new NAT network with the name `NatNetwork`.

![Settings tab `Network` where we can create, manage and delete NAT networks.](images/lab1-vbox12.png)

The default values ​​of the NAT network are sufficient for our needs, but we can change them by selecting the desired NAT network and clicking on the button with the network adapter icon and the orange gear symbol. We can also forward the ports for the entire NAT network. The selected NAT network can also be deleted by pressing the button with the network adapter icon and the red cross symbol.

![Settings tab `Network` where we can manage NAT network parameters in detail.](images/lab1-vbox13.png)

If we use VirtualBox from version 7.0 onwards, then we manage the NAT network by selecting the menu `File\Tools\Network Manager` and then selecting the `NAT Networks` tab.

![Settings tab `NAT network` where we can create, manage and delete NAT networks.](images/lab1-vbox18.png)

Now we can go back to the settings of our virtual machine by clicking on the `Settings ...` button in the menu above. Then, in the `Network` tab, we select that `Adapter 1` uses the common `NAT Network` with the name `NatNetwork` that we previously created.

![The `Network' settings tab where we can set that `Adapter 1` uses a common `NAT network` named ``NatNetwork''.](images/lab1-vbox14.png)

The IP addresses in individual virtual computers can be checked with the `ip` command.

    ip a

    1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
    2: enp0s3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 08:00:27:40:07:ad brd ff:ff:ff:ff:ff:ff
    inet 10.0.2.15/24 brd 10.0.2.255 scope global dynamic noprefixroute enp0s3
       valid_lft 85104sec preferred_lft 85104sec
    inet6 fe80::a00:27ff:fe40:7ad/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever

The connectivity between individual virtual computers over the network can be checked with the `ping` command.

    ping 10.0.2.15 -c 4

    PING 10.0.2.15 (10.0.2.15) 56(84) bytes of data.
    64 bytes from 10.0.2.15: icmp_seq=1 ttl=64 time=0.691 ms
    64 bytes from 10.0.2.15: icmp_seq=2 ttl=64 time=1.44 ms
    64 bytes from 10.0.2.15: icmp_seq=3 ttl=64 time=4.03 ms
    64 bytes from 10.0.2.15: icmp_seq=4 ttl=64 time=0.538 ms

    --- 10.0.2.15 ping statistics ---
    4 packets transmitted, 4 received, 0% packet loss, time 3027ms
    rtt min/avg/max/mdev = 0.538/1.676/4.034/1.403 ms

![Test the reachability of two virtual computers over the network with the `ping` command.](images/lab1-result.png)
# Enable SocketCAN functionality in WSL

This tutorial will give a brief walkthrough to enable SocketCAN functionality in WSL. To be specific I am using WSL2 with Ubuntu image and this is the system details:

OS: Ubuntu 24.04.1 LTS on Windows 10 x86_64
Kernel: 5.15.146.1-microsoft-standard-WSL2+
CPU: AMD Ryzen 7 5800H with Radeon Graphics (16) @ 3.193GHz
GPU: 219e:00:00.0 Microsoft Corporation Basic Render Driver

SocketCAN does not come shipped with the kernel by default. We need to compile kernel from source in order to enable the same. For that the steps are:

- Update the apt repositories and upgrade all the required packages:

		sudo apt-get update -y
		
		sudo apt-get upgrade -y

- Clone the MS WSL2 repository by entering the following URL. In this case I am using 5.15.146.1 version as I was successful in enabling SocketCAN with this version:

		git clone https://github.com/microsoft/WSL2-Linux-Kernel.git --depth=1 -b linux-msft-wsl-5.15.146

- Install all the required components and packages to build the kernel:

		sudo apt update && sudo apt install build-essential flex bison libssl-dev libelf-dev libncurses5-dev

- Clean make and mrproper:

  		sudo make clean
  
  		sudo make mrproper

- Configure the kernel by entering into the folder and using menuconfig:

		cd WSL2-Linux-Kernel
		
		make menuconfig KCONFIG_CONFIG=Microsoft/config-wsl

- Enable CANBus functionalityt from the menuconfig:

	``Networking Support -> CAN BUS subsystem support -> change all to "M" -> CAN Device Driver -> change first 4 to "M" ``

- Save the configuration and exit

- Compile the new kernel: 

		make -j$(nproc) KCONFIG_CONFIG=Microsoft/config-wsl
		
		sudo make modules_install headers_install

- New bzImage will be created. We need to copy it from WSL storage to User storage. Remember to replace USER with windows username in all steps:

		mkdir -p /mnt/c/Users/USER/.wsl-kernels/
		
		cp arch/x86/boot/bzImage /mnt/c/Users/USER/.wsl-kernels/

- Now create a .wslconfig file with the content:

		[wsl2] 
		kernel=C:\\Users\\USER\\.wsl-kernels\\bzImage

- Power off the WSL instance and shut it down:

		sudo poweroff
		
		wsl --shutdown
		
		wsl --list

- Start the instance again by typing "wsl" in powershell

- Verify that the instance is using the new kernel by typing 

		uname -r

- Install CAN-Utils:

		sudo apt update
		
		sudo apt install can-utils

- Initialize the CAN-Utils components:

		sudo modprobe can
		
		sudo modprobe vcan
		
		sudo modprobe can-raw


Following the above procedure, all CANBus interfaces will be added. You can verify the same using "sudo dmesg" command and searching for CAN.

To add vcan interface, type:

		sudo ip link add dev vcan0 type vcan
		
  		sudo ip link set vcan0 up type vcan

You can test can-utils functionality:

   terminal1: ``cangen vcan0 -v -v``

   terminal2: ``candump vcan0``


For ease, I have also attached the bzImage to this repository. But I would strongly suggest you compile kernel from the source.

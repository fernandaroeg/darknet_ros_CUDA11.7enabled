# Installation Guide
Here is a step-by-step guide on how to install and configure the required Nvidia drivers to be able to run Darknet-ROS on a Nvidia GPU to get real-time performance of the YOLO network

## Requirements
- Ubuntu 20.04.6 LTS
- ROS Noetic
- Nvidia GPU (for this particular example the GeForce GTX 1650 is used)
- CUDA: 11.7
- cuDNN: 8.5.0

## Pre-Install Actions
This steps were implemented in a clean install of Ubuntu with no Nvidia software previously installed. Please make sure you don't have any Nvidia drivers previously installed to avoid conflicts and if so purge them using the following commands: 
	
Check which nvidia packages are installed with: 

	dpkg -l | grep -i nvidia
	
Uninstall nvidia drivers:

	sudo apt-get remove --purge '^nvidia-.*'
	sudo apt-get autoremove
	
The installation of Nvidia drivers activates the BIOS secure boot, so it is also recomended to do this installation in a BIOS with the default values. The installation instructions in this guide were done on a notebook that had the BIOS reset to its default values. 

## Installation Steps

1. **NVIDIA DRIVER**

There are several methods to install the Nvidia drivers in Ubuntu: using the Software & Updates tool in Ubuntu, install via command line using apt, or downloading the .run file of the driver. The first option is recomended as it is the easiest and will be described in this guide.
More information on the other installation methods can be found here [Nvidia Drivers Installation in Ubuntu](https://phoenixnap.com/kb/install-nvidia-drivers-ubuntu#ftoc-heading-11)

1.1 Find the recomended driver for your GPU here [Official Drivers | Nvidia](https://www.nvidia.com/download/index.aspx)
![Recommended_Driver](Recommended_Driver.png)

1.2 Make sure the driver is compatible with CUDA 11.7

For the GPU in this this example, GeForce GTX 1650, the recomended driver (April, 2023) in the Nvidia webpage is 525 as it can be seen in the image in step 1.1 
This driver is **NOT** compatible with CUDA 11.7 as it can be seen in table 3 in [CUDA Compatibility](https://docs.nvidia.com/deploy/cuda-compatibility/)
![CUDA_compatibility](CUDA_compatibility.png)
	 
**So the previous driver release compatible with this GPU is going to be installed, the Nvidia driver 515.**
	 
1.3 Now that the driver version to be installed is identified, open the **Software & Updates** tool in Ubuntu and go to the **Additional Drivers** section. Select the desired driver and click on **Apply Changes**
![Software&Updates](Software&Updates_AdditionalDrivers.png)

1.4 The system is going to give you a warning about the secure boot being enabled and ask you to set a password. Once this is done you need to reboot. 
	
	`sudo reboot`
	
1.5 During reboot you will be asked to enroll the Mok key. Click on yes.

	imagen 

1.6 Now you can check the correct installation of the drivers using the command:
	
	`nvidia-smi`	
	
And you should see something similar:
imagen
	
Altough a CUDA version is shown next to the driver version it is not installed yet. You can check using the command:
	
	`nvcc --version`
	
2. **CUDA**

-pues resulta que el 10.2 no esta disponible en el repo de 20.04, asi que voy a seguir las intrucciones del 18.04 -> NO FUNCIONO!!!! broken pkg warning
---lo voy a intentar con todo lo del driver 470, usando el cuda 11.4 y a ver si jala
#https://developer.nvidia.com/cuda-10.2-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal
#https://developer.nvidia.com/cuda-11-4-4-download-archive?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=deb_local

wget https://developer.download.nvidia.com/compute/cuda/repos/ubuntu2004/x86_64/cuda-ubuntu2004.pin
sudo mv cuda-ubuntu2004.pin /etc/apt/preferences.d/cuda-repository-pin-600
wget https://developer.download.nvidia.com/compute/cuda/10.2/Prod/local_installers/cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
sudo dpkg -i cuda-repo-ubuntu1804-10-2-local-10.2.89-440.33.01_1.0-1_amd64.deb
sudo apt-key add /var/cuda-repo-10-2-local-10.2.89-440.33.01/7fa2af80.pub
sudo apt-get update
sudo apt-get -y install cuda
reboot
nvcc --version
--despues del reboot aqui le daba nvcc --version y no me aparecia nada, faltaba 
sudo nano ~/.profile
 add at the end of file
 if [ -d "/usr/local/cuda/bin/" ]; then
     export PATH=/usr/local/cuda/bin${PATH:+:${PATH}}
     export LD_LIBRARY_PATH=/usr/local/cuda/lib64${LD_LIBRARY_PATH:+:${LD_LIBRARY_PATH}}
 fi

3. cuDNN
Make nvidia developer account
 #download 3 files, https://developer.nvidia.com/rdp/cudnn-archive
 sudo dpkg -i libcudnn7_7.6.5.32–1+cuda10.2_amd64.deb
 sudo dpkg -i libcudnn7-dev_7.6.5.32–1+cuda10.2_amd64.deb
 sudo dpkg -i libcudnn7-doc_7.6.5.32–1+cuda10.2_amd64.deb

 #check libcudnn version
 /sbin/ldconfig -N -v $(sed 's/:/ /' <<< $LD_LIBRARY_PATH) 2>/dev/null | grep libcudnn
 
 4. DOWNGRADE GCC version  https://stackoverflow.com/questions/65605972/cmake-unsupported-gnu-version-gcc-versions-later-than-8-are-not-supported
 
 5.  INSTALL ROS NOETIC and clone repo --recursive (bc there is a submodule), set ssh key for git
 
 6. configure darknet repo
 6.1 enable flags, 
 6.2 set GPU architecture
 6.3 make, compile darknet library
 6.4 make, compile darknet_ros pkg


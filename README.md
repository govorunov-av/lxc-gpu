# lxc-gpu
Creating lxc containers with gpu passthrough (Nvidia) on Proxmox 7.4+

Open pve shell and do it:

    apt install pve-headers-$(uname -r)
    curl -fSsL https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/3bf863cc.pub | gpg --dearmor | tee /usr/share/keyrings/nvidia-drivers.gpg > /dev/null 2>&1
    apt update
    apt install dirmngr ca-certificates software-properties-common apt-transport-https dkms -y
    echo deb [signed-by=/usr/share/keyrings/nvidia-drivers.gpg] https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/ / | tee /etc/apt/sources.list.d/nvidia-drivers.list
    apt install nvidia-driver cuda nvidia-smi nvidia-settings nvtop
    apt install nvidia-container-toolkit
    nvidia-ctk runtime configure –runtime=docker
    reboot
    
    For test use
    
    nvtop


Create special container:
    
    bash -c “$(wget -qLO – https://github.com/govorunov-ag/Proxmox/raw/main/ct/docker.sh)”
    
    During installation use custom parameters -> debian -> 11 (For Proxmox 7.4, and lateest for 8.*)
    
    
    Open container shell and do it 
    
    apt update && apt install gpg nvtop
    curl -fSsL https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/3bf863cc.pub | gpg --dearmor | tee /usr/share/keyrings/nvidia-drivers.gpg > /dev/null 2>&1
    apt install dirmngr ca-certificates software-properties-common apt-transport-https dkms curl -y
    echo deb [signed-by=/usr/share/keyrings/nvidia-drivers.gpg] https://developer.download.nvidia.com/compute/cuda/repos/debian11/x86_64/ / | tee /etc/apt/sources.list.d/nvidia-drivers.list
    apt update
    apt install nvidia-driver cuda nvidia-smi nvidia-settings -y
    apt install nvidia-container-toolkit
    nvidia-ctk runtime configure --runtime=docker
    reboot

List you nvidia devices in proxmox:
  
    ls -al /dev/nvidia*
    
    In ansfer you get:
    
    crw-rw-rw- 1 root root 195,   0 Oct 30 18:51 /dev/nvidia0
    crw-rw-rw- 1 root root 195, 255 Oct 30 18:51 /dev/nvidiactl
    crw-rw-rw- 1 root root 195, 254 Oct 30 18:51 /dev/nvidia-modeset
    crw-rw-rw- 1 root root 510,   0 Oct 30 18:53 /dev/nvidia-uvm
    crw-rw-rw- 1 root root 510,   1 Oct 30 18:53 /dev/nvidia-uvm-tools
    
    It is important to us gid`s and nvidia devices except modeset
    
    Add gid`s and nvidia devices in /etc/pve/lxc/<lxc_id>

    My file for example:
    
      lxc.cgroup2.devices.allow: c 510:* rwm
    	lxc.cgroup2.devices.allow: c 195:* rwm
    	lxc.mount.entry: /dev/nvidia0       dev/nvidia0       none bind,optional,create=file
    	lxc.mount.entry: /dev/nvidiactl     dev/nvidiactl     none bind,optional,create=file
    	lxc.mount.entry: /dev/nvidia-uvm     dev/nvidia-uvm     none bind,optional,create=file
    	lxc.mount.entry: /dev/nvidia-uvm-tools     dev/nvidia-uvm-tools     none bind,optional,create=file

Configure file vim /etc/nvidia-container-runtime/config.toml on Container:

     ...
     no-cgroups = false
     ...

  Reboot container

That's it, you can use container(s) with gpu passthrough! 

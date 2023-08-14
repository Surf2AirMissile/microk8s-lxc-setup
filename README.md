# microk8s-lxc-setup
Provisioning steps for a microk8s node using Debian 12 LXC container

Step 1: Pull the Debian 12 LXC Container using Proxmox VE Helper Script: bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/debian.sh)"
  Make the container privileged, create a root password, allow root SSH access

Step 2: Provision the container further

  Set static IP of the container.

  Make swap amount 0.
  Enable FUSE.

  SSH into the Proxmox host and edit the container's config in /etc/pve/lxc/<CT id>.conf and add the following lines:

    lxc.apparmor.profile: unconfined
    lxc.mount.auto: proc:rw sys:rw
    lxc.mount.entry: /sys/kernel/security sys/kernel/security none bind,create=file 0 0

  Start the server and run crontab -e and add the line: @reboot ln -s /dev/console /dev/kmsg
  Run: apt install -y snapd squashfuse fuse sudo && reboot - container will restart

Step 3: Install & Provision MicroK8s

   sudo snap install microk8s --classic --channel=1.27
   sudo usermod -a -G microk8s $USER
   sudo chown -f -R $USER ~/.kube
   su - $USER
   microk8s start
   microk8s status

Step 4: Join to main node

    microk8s join 10.0.0.54:25000/7f5e3f5cd9f3593c61149262915436d4/8cfa202a5fc7

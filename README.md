# microk8s-lxc-setup
Provisioning steps for a PVE microk8s node using Debian 12 LXC container

Step 1: Pull the Debian 12 LXC Container using Proxmox VE Helper Script: bash -c "$(wget -qLO - https://github.com/tteck/Proxmox/raw/main/ct/debian.sh)"
  Make the container privileged, create a root password, allow root SSH access

Step 2a: Provision the container further

  Set static IP of the container.

  Make swap amount 0.
  Enable FUSE.

  SSH into the Proxmox host and edit the container's config in /etc/pve/lxc/<CT id>.conf and add the following lines:

    lxc.apparmor.profile: unconfined
    lxc.mount.auto: proc:rw sys:rw
    lxc.mount.entry: /sys/kernel/security sys/kernel/security none bind,create=file 0 0

Step 2b: [optional only if using cephfs/nfs/shared persistent storage on pve]

  also add a mount point to the .conf (e.g. "mp0: /mnt/pve/cephfs/microk8s,mp=/mnt/cephfs,shared=1")

Step 2c:

  Start the server and run `crontab -e` and add the line: `@reboot ln -s /dev/console /dev/kmsg`
  Run: `apt install -y snapd squashfuse fuse sudo && reboot` - container will restart

Step 3: Install & Provision MicroK8s:

`   sudo snap install microk8s --classic --channel=1.27
`   
`   sudo usermod -a -G microk8s $USER
`   
`   sudo chown -f -R $USER ~/.kube
`   
`   su - $USER
`   
`   microk8s start
`   
`   microk8s status
`

Step 3b: Missing AppArmor profiles for microk8s on the LXC will mean after reboot the microk8s command cannot be run
(https://sleeplessbeastie.eu/2020/07/20/how-to-deal-with-missing-apparmor-profiles-for-microk8s-on-lxd/)

`   $ echo -e '#!/bin/bash\n\napparmor_parser --replace /var/lib/snapd/apparmor/profiles/snap.microk8s.*\nexit 0\n' | sudo tee /etc/rc.local
`   
`   #!/bin/bash
apparmor_parser --replace /var/lib/snapd/apparmor/profiles/snap.microk8s.*
exit 0
`   
`   sudo chmod +x /etc/rc.local
`   
`   $ /usr/lib/systemd/system-generators/systemd-rc-local-generator
`   
`   reboot
`   
`   $ systemctl status rc-local
`   



Step 4: Actions on the Control Plane Node

  You need to make sure the control plane node can also resolve the hostname of the workers.

  Add on the /etc/hosts of the control plane node: e.g.:

  192.168.0.10 workerName

Step 6: Get a refreshed add-node token from the control plane node

    microk8s add-node

Step 7: Run on the slave node

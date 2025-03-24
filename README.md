# microk8s-proxmox

# Installing microk8s in an LXC container

I wanted to run Microk8s on a Proxmox 6 host inside of an LXC container. These are my notes from the journey.

1. Create a privileged LXC container through the Proxmox web interface
  * Enable nesting and FUSE
    * In Proxmox UI, select container, then Options > Features > Check nesting and FUSE boxes
2. SSH into the Proxmox host and edit the container's config in /etc/pve/lxc/<cid>.conf
	* Add the following lines
		- lxc.apparmor.profile: unconfined
		- lxc.cap.drop:
		- lxc.mount.auto: proc:rw sys:rw
3. Start (or restart) the container
4. SSH into the container and create a symlink for /dev/kmsg, which is missing in Ubuntu 19.10 containers
	* ln -s /dev/console /dev/kmsg
	* Has to be repeated on container reboot, which is annoying.
5. Install snapd: apt install snapd
6. Install microk8s: snap install microk8s --classic

The snap commands may need to be run more than once to get past errors.

#### Troubleshooting

If you get "cannot change profile for the next exec call: No such file or directory", try running: apparmor_parser -r /var/lib/snapd/apparmor/profiles/*





# SOLUTION

1- Download debian 12 (debian-12-standard_12.2-1_amd64.tar.zst) ct template ( i don't know why, but i couldn't make it work with ubuntu ):

pveam download local debian-12-standard_12.2-1_amd64.tar.zst
2- Make a new CT with the following configurations:

Make sure "Unprivileged Container" is unchecked, so it becomes privileged.
Use downloaded debian template.
Make sure sure swap is 0 in memory tab.
NOTE: Don't start the CT yet.
3- On the created CT go to options tab, double click on "Features" and enable "Fuse" and "Nesting".
4- SSH into your Proxmox node and navigate to /etc/pve/lxc/, and open your {ct_id}.conf and add these few lines in:

lxc.apparmor.profile: unconfined
lxc.cap.drop:
lxc.mount.auto: proc:rw sys:rw
lxc.mount.entry: /sys/kernel/security sys/kernel/security none bind,create=file 0 0
5- Start the CT
6- Open your crontab using crontab -e and add this line at the end:

@reboot ln -s /dev/console /dev/kmsg
7- Install required apt packages:

apt update && apt upgrade -y && apt install snapd squashfuse fuse sudo -y
8- Reboot for changes to take effect.
9- After the reboot, finally you can install microk8s:

snap install microk8s --classic
10- Enjoy your first microk8s on Proxmox 🎉

# ampere kvm setup on rhel9 aarch64
Projects and documentation organization for what I'm doing with the Ampere platform with KVM

Repositories used
epel #Needed for bridge-utils package
rhel-9-for-aarch64-appstream-rpms #Should be able to use media with a local repo
rhel-9-for-aarch64-baseos-rpms #Should be able to use media with a local repo

Package Installation
````
dnf install qemu-kvm libvirt virt-install tpm2-tools tpm2-abrmd bridge-utils
````
Create a bridge
````
nmcli con add type bridge ifname br0
````
Add bridge to interface
I’m using a single interface on the host, which is LAN2, and the device name is enP3p3s0f1

````
nmcli con add type bridge-slave ifname enP3p3s0f1 master br0
````
Restart interface so that the bridge is active
````
ifconfig enP3p3s0f1 down; ifconfigenP3p3s0f1 up
````
Add your primary user account to the required groups for running the virt tools
````
usermod -aG libvirt,libstoragemgmt <user account>
````
Enable the daemons
````
for i in qemu network nodedev nwfilter secret storage interface
  do sudo systemctl enable --now virt${i}d{,-ro,-admin}.socket
done
````
Add a configuration file, either echo it or use a heredoc like my example. This is for your primary user to access virt tools, sort of like exporting the KUBECONFIG
````
cat >>~/.config/libvirt/libvirt.conf <<EOF
uri_default = "qemu:///system"
EOF
````
At this point, I cloned my internal repo, which has scripts for standing up KVM instances using cloud-init.

Let me know if you have any issues, and I’ll do what I can to help you get this up and running.

I connect to the console of a domain (VM/guest in libvirt speak) so that I can get the serial output and see if my cloud init process with the post deployment “runcmd” section is functioning:
![image](https://github.com/user-attachments/assets/4c64cb82-a1d3-43a6-a397-8abdf270121f)

I’m playing with cockpit as I need to implement controls and session recording at work, testing and validating with this host before I go break production. This shows that the domain has completed post in the KVM environment and is at a cloud-init stage as part of the deployment:
![image](https://github.com/user-attachments/assets/3df01e44-06f7-41c7-be70-46ce92757805)


# Setting up a local devstack (updated to 2020-03-31)

## Requirements
vagrant with virtualbox

## Getting started

First, bring up a Vagrant box. This uses `centos/7 (1905.1)` box, 5GB of RAM and 40GB of (copy-on-write, so will lazily allocate as you consume) disk.
```
vagrant up
vagrant ssh
```
At this point, you may want to double check that VirtualBox is using nested `vmx` instructions to use hardware emulation inside a virtual machine; otherwise, you won't be able to create VMs inside the VM.  
You can check from within the VM with a `cat /proc/cpuinfo|grep vmx`. If you see stuff, skip this paragraph. If you instead see nothing, then you need to `vagrant halt`, go to VirtualBox, modify your VM under `Settings > System > Processor > Enable Nested VT-x/AMD-V` and restart the VM (if this is greyed out, enable virtualization from your BIOS). Now the check should show stuff.

Inside the box, install git, clone the repo, copy the configuration and start the stack:
```
sudo yum install -y git
git clone https://opendev.org/openstack/devstack
cd devstack/
cp /vagrant/local.conf .
./stack.sh
```

After about 30-40 minutes, DevStack will be up, with Swift (with S3), Nova, Glance, Neutron, Keystone, Horizon *BUT* Horizon will be broken since it relies on Apache2 + WSGI with Python 2.7, while we used Python 3.  
We need to replace WSGI module with Python 3 equivalent.  
Just run the following:
```
sudo yum install -y rh-python36-mod_wsgi.x86_64
echo "LoadModule wsgi_module /opt/rh/httpd24/root/usr/lib64/httpd/modules/mod_rh-python36-wsgi.so" | sudo tee /etc/httpd/conf.modules.d/10-wsgi.conf
sudo systemctl restart httpd
sudo iptables -F
```
The last line also deactivates the firewall on the VM, which is sealing everything from outside (very nasty DevStack preset).

Head your browser to [http://192.168.42.101/dashboard](http://192.168.42.101/dashboard) and you are good to go.

Configuration files are under `/etc` and services are under `/opt/stack`. Everything runs as a SystemD unit with prefix `devstack@<service name>` (`n` is nova, `q` is neutron (it was quantum, remember?)), therefore the logs are all managed by journalctl.

## Troubleshooting
### Horizon still displays a 500 error
Make sure that the module referenced in `/etc/httpd/conf.modules.d/10-wsgi.conf` is not linked to Python 2: ldd on the module and see if you catch traces of Python 2.  
After you install `rh-python36-mod_wsgi.x86_64` package, the file we need should be like under `/opt/rh/httpd24/root/usr/lib64/httpd/modules/mod_rh-python36-wsgi.so`. If you can't find it, look for it with `sudo find / |grep wsgi|grep so$`. 
Doing ldd here will show it's python 3.

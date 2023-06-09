# Ansible Automation Platform Server Setup Notes


Last update: 18 April 2023

### Pre-Reqs

For this example we will be working with RHEL 8.x 

Create a VM for the AAP infrastructure instance and install RHEL 8.7.  The VM was sized with 2 vCPUS, 8GB RAM and 120GB "local" drive.  Note: For this example I have enabled Simple Content Access (SCA) on the Red Hat Customer portal and do not need to attach a subscription to the RHEL or Satellite repositories.  After you have created and started the RHEL 8.7 VM, we will ssh to the RHEL VM and work from the command line.

For this lab environment I chose aap01.example.com for the hostname of the server hosting Satellite. 

Check hostname and local DNS resolution.  Use dig to test forward and reverse lookup of the server hosting Satellite.  If the Satellite hostname is not available from DNS, the initial installation will fail.    
```
$ ping -c3 localhost
$ ping -c3 aap01.example.com
$ dig aap01.example.com +short
$ dig -x 10.1.10.254 +short
```   
We will set the hostname on the server to avoid any future issues.
```
$ hostnamectl set-hostname aap01.example.com
```

Verify the time server with chrony.  I have a local time server that my systems use for synching time.  Type the following command to check the the time synch status.  
```
$ chronyc sources -v
```
Register the dev Server to Red Hat Subscription Management service.
```
$ sudo subscription-manager register --org=<org id> --activationkey=<activation key>
```
You can verify the registration with the following command.
```
$ sudo subscription-manager status
```    
#### Configure and enable repositories  

With SCA, we still need to enable relevant repositories for our RHEL instances.  Following steps will walk you through enabling repos.

Disable all repos.
```    
$ sudo subscription-manager repos --disable "*"
```       
Enable the following repositories.
```    
$ sudo subscription-manager repos --enable=rhel-8-for-x86_64-baseos-rpms \
--enable=rhel-8-for-x86_64-appstream-rpms
```
Verify that repositories are enabled.
```
$ sudo subscription-manager repos --list-enabled
```

Update all packages.  This may take a few minutes to complete.  Let's make sure tthe container-tools module which contains podman is up to date.
```
$ sudo dnf -y update
```
Install SOS package on base OS for initial systems analysis in case you need to collect problem determination for any system related issues.  
```
$ sudo dnf install sos
```
 I would also recommend registering this server to Insights.  
```
$ sudo insights-client --register
```
### Lab Clients

For my lab clients I setup four instances named servera.example.com through serverd.example.com

Server | IP Adresss
-------|------------
servera.example.com | 10.1.10.201

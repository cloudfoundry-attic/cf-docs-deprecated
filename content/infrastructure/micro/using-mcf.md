---
title: Using Micro Cloud Foundry
description: Using the Micro Cloud Foundry Console
tags:
    - overview
    - mcf
    - mongodb
---

**Subtopics**

- [Micro Cloud Foundry Default Configuration](#micro-cloud-foundry-default-configuration)
- [Using the Micro Cloud Foundry Console](#using-the-micro-cloud-foundry-console)
- [Micro Cloud Foundry Resource Limits](#micro-cloud-foundry-resource-limits)
- [Increasing Micro Cloud Foundry Virtual Machine Memory](#increasing-micro-cloud-foundry-virtual-machine-memory)
- [Logging in to Micro Cloud Foundry](#logging-in-to-micro-cloud-foundry)
- [Configuring Micro Cloud Foundry Networking](#configuring-micro-cloud-foundry-networking)
- [Working Offline With Micro Cloud Foundry](#working-offline-with-micro-cloud-foundry)
- [Troubleshooting Micro Cloud Foundry](#troubleshooting-micro-cloud-foundry)


## Micro Cloud Foundry Default Configuration

This section describes the Micro Cloud Foundry version 1.2 default configuration.

### Virtual Machine Configuration

+ RAM: 1GB
+ Disk: 16GB
+ vCPUs: 2

### Service Limits

+ MySQL: 2GB storage, maximum 256MB storage per instance
+ Postgres: 2GB storage, maximum 256MB storage per instance
+ Mongo DB: 256MB per instance
+ Redis: 256MB per instance

### Runtime Versions

+ ruby18: Ruby 1.8, version 1.8.7
+ ruby19: Ruby 1.9, version 1.9.2p180
+ java: Java 6, version 1.6
+ node: Node.js, version 0.4.12
+ node06: Node.js, version 6.0.8

### Frameworks

+ rails3
+ sinatra
+ grails
+ node
+ java_web
+ lift
+ spring

### Service Versions

+ mongodb:  MongoDB NoSQL store, version 2.0
+ mysql: MySQL database service, version 5.1
+ postgresql: vFabric PostgreSQL database service, version 9.0
+ rabbitmq: RabbitMQ messaging service, version 2.4
+ redis: Redis key-value store service, version 2.2

## Using the Micro Cloud Foundry Console

When you power on the Micro Cloud Foundry virtual machine, Linux boots and the
console text menu displays. The console menu is the primary administrative
interface for the micro cloud.

The console displays status information at the top, including the Micro Cloud
Foundry version, the host name (Identity), Cloud Foundry account email address (Admin),
and the IP address assigned to the virtual machine.

Choose options from the menu by entering the number and pressing <Enter>. The
console prompts for any information required to perform the task.

1. **refresh console**. Choose this option to redraw the console display, for
example when messages cause the menu to scroll off the display.

2. **refresh DNS**. Update the Micro Cloud Foundry IP address in the DNS
records.

3. **reconfigure vcap password**. Choose this option to change the password for the `root` and `vcap` users.

4. **reconfigure domain**. Choose this option when you create a new domain or
generate a new token for your Micro Cloud Foundry domain. Log in to the [Micro
Cloud Foundry website](https://my.cloudfoundry.com/micro) to manage domains and
retrieve a token to enter at the prompt.

5. **reconfigure network**. Use this option to choose between DHCP and static
network configurations. If you choose **static**, the console prompts for an IP
address, gateway, network mask, and DNS servers.

6. **enable offline mode**. Choose this option to toggle the virtual machine between online and offline modes.

7. **reconfigure proxy**. If you are on a network that requires a proxy, choose
this option and enter the address and port of the proxy, for example: `192.168.1.128:4000`.

8. **services**. Displays the status of the services on the micro cloud.

9. **restart network**. Restart network services on the virtual machine.

10. **restore defaults**.

11. **expert menu**. Displays the Expert menu, where you can set the debug level, display logs, and perform other advanced configurations on your Micro Cloud Foundry.

12. **Help**. Displays a URL for online installation and setup documentation and
the default configuration limits for the virtual machine and services.

13. **shutdown VM**. Shut down the Micro Cloud Foundry virtual machine.

## Micro Cloud Foundry Resource Limits

Micro Cloud Foundry has the following default resource limits:

+ VM: 1 GB RAM, 16 GB disk
+ MySQL: 2 GB disk, max 256 MB per instance
+ MongoDB: 256 MB per instance
+ Redis: 256 MB per instance

The admin user has the following limits:

+ 1.0 G memory
+ Up to 16 provisioned services
+ Up to 16 applications

## Increasing Micro Cloud Foundry Virtual Machine Memory

The Micro Cloud Foundry virtual machine is initially configured with 1GB memory.
If you need more memory for your applications, follow these steps:

1. Shut down the Micro Cloud Foundry virtual machine.

2. In VMware Workstation for VMware Player, right-click the Micro Cloud Foundry virtual machine, and choose Settings.

3. Click Memory and specify the new memory size in the right panel.

4. Click OK.

5. Start the virtual machine.

6. Select the new highlighted item, **reconfigure memory** from the console menu.

The Micro Cloud Foundry reconfigures the virtual machine and services for the new memory size.



## Switching between Networks

If you switch between networks often and do not need to make your Micro Cloud Foundry VM available to other users, it is easier to configure the VM networking to use NAT instead of the bridged mode. Beginning with version 1.2, NAT is the default mode for the Micro Cloud Foundry VM. If you want to share your cloud with others, you will have to enable bridged mode.

## Logging in to Micro Cloud Foundry

Micro Cloud Foundry is a virtual machine with an Ubuntu Linux operating system
and the Cloud Foundry software layer and application services. There is no
graphical desktop environment installed, but you can log in to the virtual
machine and get a bash shell using `ssh`.

It is not a good idea to customize the Cloud Foundry services in any way, since
this introduces a dependency that may not be satisfied when you move
applications to another Cloud Foundry instance.

Some reasons you might log in to Micro Cloud Foundry are:

+   View server log files
+   Check process status or loads, for example, using `top`
+   Troubleshoot DNS problems on your local network

You can log in as `root` or `vcap` using the password you set when you initially
start and configure the Micro Cloud Foundry virtual machine.

From a computer with `ssh` installed, log in to Micro Cloud foundry using a
command like the following:

        $ssh root@domain.cloudfoundry.me

where *domain* is the domain name you registered for the Micro Cloud Foundry.
You can also use the IP address assigned to the virtual machine, which is
displayed on the console.

## Configuring Micro Cloud Foundry Networking

Micro Cloud Foundry provides a network environment similar to Cloud Foundry.
URLs are resolved using DNS to locate the host computer running
your application, the Micro Cloud Foundry virtual machine. The application
processes the request, including parsing the remainder of the URL and the HTTP
request and returning a response to the client. The client browser and
application interact with the network in exactly the same way they do in
production, except the cloud is running on the same host.

Development and deployment tools - `vmc` and STS - also work with Micro Cloud
Foundry just as they work with CloudFoundry.com or any local or hosted Cloud
Foundry instance.

To provide a production-like network environment, Micro Cloud Foundry associates
the virtual machine's IP address with *domain*.cloudfoundry.me in DNS. This
requires an Internet connection so that Micro Cloud Foundry can update its
address at `cloudfoundry.me` and so that the URL can be resolved when you access
the application with your browser. When the virtual machine is assigned a new IP
address, for example if you move to a different location, it updates the DNS
records.

If your browser uses a proxy and the DNS lookup is not working, you may have to
exclude `.cloudfoundry.me` from the proxy.

The way you configure the network adaptor in the Micro Cloud Foundry virtual
machine determines who can reach your micro cloud:

- If you choose the Bridged network connection option, your micro cloud gets an
address on the LAN from a DHCP server on the LAN. It can be accessed from other
hosts on the LAN.

- If you choose the NAT network connection option, your micro cloud gets an
address on a network that exists only on the host running the virtual machine.
Your cloud can only be accessed from a browser on the host running the virtual
machine.

Unlike the Bridged network option, with the NAT connection option, you do not
get a new address when you change locations. If you are not sharing your micro
cloud with others and you move around frequently, use the NAT option to avoid
the possibility of lagging DNS updates.

## Working Offline With Micro Cloud Foundry

When you install Micro Cloud Foundry using a token from the Cloud Foundry website, it uses dynamic DNS functionality to allow any Internet-connected computer that is on the same network with the VM to connect to it. That is, to get the local network address for the VM requires Internet access, even if you are accessing the VM from the same computer on which it is running. This is called online mode and it requires Internet access.

If you have to work without an Internet connection, you must put Micro Cloud Foundry into offline mode and configure your host to route DNS requests to your Micro Cloud Foundry VM. Also, if you initially configure Micro Cloud Foundry with a domain name instead of a configuration token, you must *always* use offline mode.

Offline mode is only supported with the VM network adapter set to NAT. This means it can only be accessed from the host it is running on. To share your Micro Cloud Foundry with others, you must set the network adapter to Bridged mode and run Micro Cloud Foundry in online mode.

If you use Micro Cloud Foundry in offline mode and still have an active Internet connection, you may experience problems accessing sites on the Internet.

### Configuring Micro Cloud Foundry for Offline Mode

You can configure offline mode manually or use the `vmc micro` command. VMC version 0.3.16.beta4 or higher is required to use the `vmc micro` command. See [Using the VMC micro Command](#using-the-vmc-micro-command) for instructions.

The remainder of this section describes how to configure offline mode manually.

There are three tasks to complete to put Cloud Foundry in offline mode.

**Step 1**. In the VM's Virtual Machine Settings, select Network Adapter and make sure that NAT is selected. If you have to change the setting, restart the virtual machine.

**Step 2**. In the Micro Cloud Foundry console menu, select option 6 to toggle to offline mode.

**Step 3**. Configure your host computer to route DNS requests to the Micro Cloud Foundry VM. This is accomplished in differing ways depending on the OS and whether you use DHCP or a static IP address.  In the instructions that follow, replace the IP number 172.16.52.136 with the IP number shown on the Micro Cloud Foundry console. Replace mydomain.micro with your offline domain name.

### Linux

If you are using DHCP, edit the file `/etc/dhcp3/dhclient.conf` and add this line:

```bash
prepend domain-name-servers 172.16.52.136
```

If the VM is configured with a static IP, edit the file `/etc/resolv.conf` and add the following line before the rest of the name servers:

```bash
nameserver 172.16.52.136
```

### Mac OS X

If you are using DHCP, create the directory `/etc/resolver`, and then create a file with your offline domain name, for example `mydomain.micro`. Add the following line to this file:

```bash
nameserver 172.16.52.136
```

If you configured the Micro Cloud Foundry with a static IP, open the Network Preferences and add 172.16.52.136 first in the list of DNS servers.

### Windows

Follow these steps  whether you configured Micro Cloud Foundry with DHCP or a static IP address:

+ Open the Network and Sharing control panel.
+ Choose Change adapter settings.
+ Right-click VMware Virtual Ethernet Adapter for VMnet8, and choose Properties.
+ Set the preferred DNS server to 172.16.52.136.

## Using the VMC Micro Command

The `vmc micro` command automates the steps described in the previous section. Review that section to understand how the command changes your configuration.

Install the vmc gem, or upgrade it if needed. You need version 0.3.16.beta4 or greater. See [VMC Installation](/tools/vmc/installing-vmc.html) for instructions.

Here is the syntax for the `vmc micro` command:

```bash
Usage: vmc micro [options] command

Options
  --password          VCAP user password
  --vmrun             /path/to/vmrun
  --vmx               /path/to/micro.vmx
  --save              Save VCAP password

Commands
  offline             Run Micro Cloud in offline mode
  online              Run Micro Cloud in online mode
  status              Display current status
```

To reconfigure and control the virtual machine, vmc needs paths to the .vmx file and the vmrun command. It may find the vmrun command on your path, but the first time you run it you must provide the path to the micro.vmx file using the --vmx option. The paths are saved in the .vmc_micro file in your home directory so you do not have to specify the options agai on future runs.

In the following example, the path to the micro.vmx file is specified. vmc discovers that the VM is not running and offers to start it. It reports the status and asks whether to save the password for future runs.

```bash
$ vmc micro --vmx /home/mcf/micro.vmx status
Please enter your Micro Cloud Foundry VM password (vcap user)
Password: ********
Confirmation: ********
Micro Cloud Foundry is not running. Do you want to start it? y
Micro Cloud Foundry currently in online mode
VMX Path: /home/mcf/micro.vmx
Domain: mydomain.cloudfoundry.me
IP Address: 192.168.255.134
Do you want to save your password? n
```

Execute `vmc micro offline` to work offline.

```bash
$ vmc micro offline
```

This puts the VM in offline mode (the same as selecting option 6 from the menu) and sets the DNS on your host to query the VM. For actions that require administrative or root privilege, you may be prompted to authenticate.

Execute `vmc micro online` to work online.

```bash
$ vmc micro online
```

This puts the VM in online mode and reverses the DNS configuration change on your host computer. For actions that require administrative or root privilege, you may be prompted to authenticate.


## Troubleshooting Micro Cloud Foundry

### Gathering Debugging Info

If you encounter problems and need help debugging, please do the following:

1.  In the Micro Cloud Foundry console menu, enter 11 to display the Expert menu.

2.  Enter 1 to set the debug level to DEBUG.

3.  Retrieve the `/var/vcap/sys/log/micro/micro.log` file from the VM and attach it to the support ticket. To retrieve the file, log into the VM as vcap, using the password set when the VM was configured. For example, using scp and the IP address shown on the console:

```bash
$ scp vcap@92.168.1.215:/var/vcap/sys/log/micro/micro.log .
```

### Proxy Problems

If you use a proxy, keep in mind that the proxy may not be able to access your Micro Cloud Foundry VM. For example, if you set the VM's network adaptor to use NAT, there is no way for the proxy to find the VM, so you must exclude your domain system's proxy settings.

Another proxy related problem occurs when the VM's network adaptor uses bridged mode and you have a VPN on your host. The Micro Cloud Foundry VM traffic won't enter the tunnel, and thus cannot reach the proxy.

### Problems Accessing Your Instance

If the DNS entry for you Micro Cloud Foundry VM is not up-to-date, accessing your instance can fail. For example:

```bash
$ vmc target api.martin.cloudfoundry.me
Host is not valid: 'http://api.martin.cloudfoundry.me'
Would you like see the response [yN]? y
HTTP exception: Errno::ETIMEDOUT:Operation timed out - connect(2)
```

Check the Micro Cloud Foundry console. Choose "1" to refresh the screen. If you see a "DNS out of sync" message like the following:

```bash
Current Configuration:
 Identity:   martin.cloudfoundry.me (DNS out of sync)
 Admin:      martin@englund.nu
 IP Address: 10.21.164.29 (network up)
```

choose "2" to force a DNS update.

If the console DNS status is "ok", then the IP address of the VM matches the IP in DNS. Validate that you do not have a cached entry on your local system with the host command:

```bash
$ host api.martin.cloudfoundry.me
api.martin.cloudfoundry.me is an alias for martin.cloudfoundry.me.
martin.cloudfoundry.me has address 10.21.165.53

```

If the two differ, then you need to flush the DNS cache:

**Mac OS X**

```bash
dscacheutil -flushcache
```

**Linux (Ubuntu)**

```bash
sudo /etc/init.d/nscd restart
```

**Windows**

```bash
ipconfig /flushdns
```

### "Cannot connect to cloudfoundry.com" Message When Configuring Micro Cloud VM

When you configure the Micro Cloud VM to use DHCP it is assigned an address from the network's DHCP pool. If you continuously create/destroy VMs, for example in a testing environment, and your DHCP addresses have a lease life of 12 to 24 hours, it is possible to exhaust the DHCP pool. You will see a message during the configuration process that you cannot connect to cloudfoundry.com, although you are able to reach cloudfoundry.com with a browser. Restarting your DHCP server should return unused leases to the pool so they can be reused before the lease life expires.
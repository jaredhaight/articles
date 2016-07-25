## PFSense FTW
In the last [part of the series](https://www.psattack.com/articles/20160718/setting-up-an-active-directory-lab-part-1/), we created a **Host-Only Network** for our domain. This is really nice because it will help to keep the miscellaneous domain traffic from crossing over to other networks we're connected to. The downside is that we won't have internet access. We're going to fix this by creating a simple Firewall VM that will handle routing between the Host-Only Network and our Host Computers network. To do this, we're going to use [PFSense](https://www.pfsense.org/) a simple (but feature packed) firewall that's built off BSD. 

### Building our Firewall VM
Provision a new VM for the Firewall. It doesn't have to be anything fancy, 1 vCPU, 256MB of RAM and 5GB of HDD should cover it. Make sure that when you're creating the VM, you choose **FreeBSD 64-bit** 

The other thing we need to do is add a second network card to the VM. 

1 - Go to properties pane for the VM, select the "Network" section.

2 - On the first network card tab, make sure that it is set to "NAT"

3 - Click the second network card tab, enable the adapter

4 - Assign it to the "Host-Only Network" you created in part 1.

![](https://www.psattack.com/webhook-uploads/1464546425687/pfsense01.png)

### Installing PFSense
1 - [Download the ISO from their website](https://www.pfsense.org/download/) and boot the VM off of it.

2 - When prompted **Press I** to start the Installer. Choose **Accept These Settings** and then **Quick/Easy Install**

![](https://www.psattack.com/webhook-uploads/1464547083919/pfsense03.png)

3 - When prompted, choose **Standard Kernel** and then reboot when prompted. Make sure to **unmount the ISO from the VM** before the machine boots back up.

### Setting up PFSense
When PFSense is booted, you'll be presented with a menu.

![](https://www.psattack.com/webhook-uploads/1464555002865/pfsense05.png)

We need to configure the LAN interface to work properly for our Host-Only network. To do this, from the PFSense menu, press `2` to select **Change IP Addressing** and `2` again to select the **LAN Interface**. You'll then run through a series of prompts to setup the router. Here are the answers:

1 - **New LAN IPv4 Address:** The address we give this interface should be the same address you used as the _gateway address_ when you setup the IP address on the Domain Controler in part 1. In the example, I used 10.10.10.2

2 - **New LAN Subnet Bit Count:** This depends on how you setup your "Host-Only network", but it's probably 24

3 - **Upstream Gateway Address:** Just press enter, we don't need an upstream IP for a LAN interface.

4 - **New LAN IPv6 Address:** Just press enter, we're not using IPv6 for routing.

5 - **Enable DHCP Server on LAN?** "N", we want to disable the DHCP server in PFSense.

6 - **Revert to HTTP?** "N", We do not want to use HTTP for the admin interface.

![](https://www.psattack.com/webhook-uploads/1464554990013/pfsense06.png)

Awesome, our PFSense box is setup. There's a ton more we can do with PFSense, it will definitely be able to grow with you if you start building more complicated labs, for now though this is all we need for our simple lab setup.

## Installing DHCP

Hosting DHCP through a Windows box in Active Directory gives us plenty of benefits, chief among them being that DHCP leases will automatically be added to our DNS servers. It's also incredibly easy to setup. 

For the purposes of our lab, we can just host DHCP on the Domain Controller. This isn't something you'd typically see in production except for maybe in very very tiny networks. In a production environment you typically want your domain controllers dedicated to domain controlling. Adding extra roles to the DCs increases risk, patching overhead and the chance that they're going to crash cause of something stupid.

Let's go back to our DC and start the **Add Roles and Features** wizard again. This time we're going to add the DHCP Server Role. When prompted to add the required features, select **Add Features**.

![](https://www.psattack.com/webhook-uploads/1464556066869/dhcp01.png)

After that, keep clicking "Next" until you get the option to "Install", then click that. 

Once the install has finished, we can configure the DHCP server by clicking on the "Notification" button in Server Manager and selecting **Complete DHCP configuration**.

![](https://www.psattack.com/webhook-uploads/1464556318285/dhcp02.png)

Just click Next > Next > Finish for this one.

### Configure the DHCP Server

1 - In **Server Manager** click on the Tools menu in the upper right and select **DHCP**

![](https://www.psattack.com/webhook-uploads/1464557932690/dhcp03.png)

2 - Expand your domain on the left-hand side, right click **IPv4** and select **Add New Scope**

![](https://www.psattack.com/webhook-uploads/1464557948140/dhcp04.png)

3 - Click Next through the Wizard. When prompted, name your DHCP scope whatever you want. I went with the unimaginative title of "Lab"

4 - When prompted for the scope, create a range of 50 to 100 IPs within your network and set your subnet mask appropriately.

![](https://www.psattack.com/webhook-uploads/1464556886055/dhcp07.png)

5 - Keep clicking next in the wizard you're asked if you'd like set **additional options**, select **yes** and click next.

6 - For the router address, **enter the address that you set for the LAN interface on the PFSense VM** (the same address that you put as the default gateway on the Domain Controller). For this lab, I'm using `10.10.10.2`. Click **Add** then click next.

![](https://www.psattack.com/webhook-uploads/1464557126699/dhcp08.png)

7 - Keep clicking **Next** until you get to the end of the wizard.

## Now what?
We have a pretty functional lab network now. We have a domain, internet works and DHCP is up and running. [In the next article](https://www.psattack.com/articles/20160718/setting-up-an-active-directory-lab-part-3/), we'll cover adding and managing users and computers through Active Directory and some basic Group Policy.
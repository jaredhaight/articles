## Who is this for?
This guide is written for people who are fairly comfortable with Windows (any version) but haven't necessarily worked with labs, virtualization software or Active Directory. You should be comfortable doing basic Windows stuff (renaming a computer, changing IP addresses, etc). Through this series of posts, we'll cover:
* Setting up Virtualbox to virtualize our lab
* Setting up a firewall/router to keep our lab separate from the rest of network but still allow it to access the internet
* Setting up a basic Active Directory environment
* Some AD basics like how to create users, join computers to the domain and create group policy objects.

Throughout this, I'll try and explain what technologies are at work and why we're doing whatever it is that we're doing.

## What is Active Directory
Before we get started with building our lab, let's establish what we're actually building here. Active Directory is a collection of technologies used to manage a group of users and computers. This collection of users and computers falls under a **domain**. A domain is referred to by its name. Older networks (say.. before Windows 2008) typically had a single name, for example **LAB**. Nowadays, domains are named using a more typical domain name structure. Typically, companies will have their Active Directory domain as a subdomain of their main domain for example, **ad.company.com**. For this lab though, our domain will simply be **example.com**.

Another term that you'll see is **Forest**, which is a collection of domains. A domain is always part of a forest, even if there's just one domain.

For this introductory article, all that we need to know is:
* A **domain** is a collection of users and computers managed by Active Directory
* Our domain name is **example.com**

## What Services make an Active Directory domain
As mentioned before AD is a collection of services. For the purpose of our simple lab all of these services are going to be handled by a single server, our **Domain Controller**. This is actually how things are handled in most production networks too, except you'll have multiple domain controllers for resiliency (_very bad things happen if your DCs vanish from your domain_)

**DNS** - DNS is absolutely vital for Active Directory to work. Active Directory relies on a series of DNS records to establish what services are available on the domain and who provides what. For the most part, these records are managed automatically, all we need to worry about right now is making sure DNS is available. 

**LDAP** - Lightweight Directory Access Protocol. This service is responsible for keeping track of what is on the network

**Kerberos** - Kerberos handles Single Sign On throughout the domain. It is what allows you to use one username and password to log into multiple computers throughout the domain.

**SMB** - SMB is used to share files throughout a domain. Domain Controllers use it to  share group policy objects among other things.

All of these services are installed and configured when you install the **Active Directory Domain Services** role in Windows Server. 

## Lets build a Lab

The first thing we need for our lab is a copy of Windows Server. Microsoft offers 180-day trial copies of their server products so just [download a copy](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-server-2012-r2).

The next thing we need is some virtualization software. Use whatever you're comfortable with. At the time of this writing I'm using [Virtualbox](https://www.virtualbox.org/wiki/Downloads) which is free and works great.

### Provisioning a VM for your DC
For lab purposes, we don't need anything beefy. I typically provision my lab DCs with 1 vCPU,  1GB of RAM and 30gb of HDD space.

### The Virtual Network
When considering the networking of your virtual lab, it's important to consider what you're trying to achieve. Virtual Networks (at least, the kind you'll be playing with on through VMWare Workstation or Virtualbox) come in three flavors
* **NAT:** The virtualization software creates a new virtual network that VMs are a part of. The software acts as a router, natting traffic between the virtual network and the real network that a host computer is associated with. This is the default networking setup for most Virtualization software
* **Host-Only:** The virtualization software creates a new virtual network (as above), but in this case, it does NOT act as a router. VMs in a host-only network can talk among themselves, but their traffic can't leave the host device.
* **Bridged:** The virtualization software puts the VM on the same network as the host computer, so the VM acts just like another computer on the network.

Depending on how you're using your lab, you'll need to decide what networking option works best for you. For this walkthrough, we're going to create a Host-Only network for use with our lab. In VirtualBox, do the following:

1 - In VirtualBox Preferences, go to **Network**.

2 - Click on the **Host-Only Network** tab and **add a new network**.

3 - Edit the new Host-Only network and **give it an IP address scheme** that agrees with you (or just use the one in the screenshot below.)

![](https://www.psattack.com/webhook-uploads/1464541351574/virtual_network01.png)

Now, select the VM you've created and edit its settings to use the network you just created.

![](https://www.psattack.com/webhook-uploads/1464541365720/virtual_network02.png)

## Installing Windows
Installing Windows is a pretty straight forward affair. For this example, we'll be using a copy of Server 2012 R2 with the GUI.

![](https://www.psattack.com/webhook-uploads/1464473370338/dc_install01.png)

The only other thing that might be a little confusing, is that we're going to want to choose **Custom Install** to do a clean install.

![](https://www.psattack.com/webhook-uploads/1464473422933/dc_install02.png)

Other than that, just follow the steps in the wizard any you'll be fine.

### Setup Windows
If you've never used Windows Server before, don't worry. For the most part, Windows Server functions exactly like client versions of Windows. It just has some extra bells and whistles. 

Unless otherwise specified, you can assume that however you'd accomplish something in a Windows client is how you can do it on the Server.

To setup our Domain Controller, we're going to have to do three things:

1 - Install our Virtualization Guest Extensions. In VMWare these are called VM Tools, in VirtualBox they're called Guest Additions, whatever they're called in your virtualization software, install them.

![](https://www.psattack.com/webhook-uploads/1464473708980/dc_install03.png)

2 - Give it a static IP address. The actual address doesn't matter, I typically give my DCs the last IP in the subnet. We're also going to set our gateway to **.2** and use Google's DNS servers for name resolution (you can use whatever you want). In [part 2](/articles/20160718/setting-up-an-active-directory-lab-part-2/) of this series, we'll be adding routing to our host-only network so that we'll have some internet access in the lab.

![](https://www.psattack.com/webhook-uploads/1464541828947/ip_addressing.png)

3 - Rename the computer something sensible, or not... whatever. You'll probably want to know what your DC is named though.

![](https://www.psattack.com/webhook-uploads/1464541891889/rename_computer01.png)

4 - Reboot.

## Installing the Domain Controller Role

Yay! The fun part! When your server comes back up, we need to set it up as a domain controller. This consists of two parts, _installing the Active Directory Domain Services role_ and the _promoting the server to a domain controller_.

### Installing the ADDS Role

This is super easy. Open up **Server Manager** from the start menu and from the Quick Start, select **Add Roles and Features**.

![](https://www.psattack.com/webhook-uploads/1464542373568/add_role.png)

Click next until you reach the step to select roles, select **Active Directory Domain Services** and click **Add Features** to the window that pops up.

![](https://www.psattack.com/webhook-uploads/1464542448720/add_role02.png)

Keep clicking Next until the install happens. Wait patiently for the install to finish.

### Promoting the Server to a Domain Controller

Once the role is installed, you should see a little warning sign in Server Manager, click that and then select **Promote this Server to Domain Controller**

![](https://www.psattack.com/webhook-uploads/1464542625184/promotion01.png)

The first thing we're going to have to do is create our forest, so select **New Forest** and enter a domain name. I stick with **"example.com"** [cause I love standards](https://tools.ietf.org/html/rfc6761).

![](https://www.psattack.com/webhook-uploads/1464543078229/promotion02.png)

On the next screen we'll leave the defaults and we're going to create a **recovery password**. You will most likely never use this in a lab, its typically used when very bad things happen to your domain.

![](https://www.psattack.com/webhook-uploads/1464543182522/promotion03.png)

On the next screen, you're going to see this error message. It's cool, it just means that the server couldn't find an existing DNS infrastructure for the domain you specified earlier. Of course it couldn't, we haven't created it yet. Click **Next** and move on with your life.

![](https://www.psattack.com/webhook-uploads/1464543280550/promotion04.png)

For the next series of prompts, we'll just accept the defaults. It's good enough for our lab (in actuality, they're good enough for most production networks too). The wizard will check some pre-requisites and then you'll end up at the install screen. There will be some warnings, that's fine. Click **Install**.

![](https://www.psattack.com/webhook-uploads/1464543428176/promotion05.png)

The Wizard is going to handle installing and configuring Active Directory and DNS (which are kind of inseparable) and then the server will reboot.

## Now What?

When the server comes back up, you'll have a domain controller! Hooray!! You can log in using the same Administrator account as before, only now it's a **Domain Administator**. _An interesting characteristic of Domain Controllers_ is that there are no longer local accounts on the server. Domain Controllers **are** the domain, they're what host everything that makes the domain work, so the concept of "local accounts" doesn't apply here.

![](https://www.psattack.com/webhook-uploads/1464544414515/domain_controller01.png)

In [Part 2](https://www.psattack.com/articles/20160718/setting-up-an-active-directory-lab-part-2), we'll cover how to add DHCP and Internet access to our lab, and finally, in [Part 3](/articles/20160718/setting-up-an-active-directory-lab-part-3/) we'll cover some things we can do with our fancy domain.

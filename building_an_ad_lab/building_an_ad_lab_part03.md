## Where are we at?
So in [Part 1](https://www.psattack.com/articles/20160718/setting-up-an-active-directory-lab-part-1/), we created our Domain Controller and in [Part 2](https://www.psattack.com/articles/20160718/setting-up-an-active-directory-lab-part-2/) we addressed networking in the lab. Now we're ready to actually play with Active Directory. The first thing we're going to do is create a new admin user for ourselves. We'll then add another computer to the domain and then finally we'll create some basic Group Policy settings.

## Creating a Users
For this article, we'll be using the **"Active Directory Administrative Center"** (or **ADAC**) to work with AD. To get started, go to the Start Menu and start **"Active Directory Administrative Center"**.

![](/https://www.psattack.com/webhook-uploads/1468788578422/ad_lab_administrative_center01.png)

ADAC is relatively new, for the longest time Administrators used "Active Directory Users and Computers" (ADUC) to manage users and computers in their domain. ADAC is a little bit more capable and does a good job of surfacing common tasks. Of course in addition to the GUI tools, you can use PowerShell to manage AD. Like a lot of modern Windows management tools, this is actually all ADAC is doing. You can even view the PowerShell commands that it's running by expanding the **"Windows PowerShell History"** bar at the bottom of the window.

![](/https://www.psattack.com/webhook-uploads/1468872759870/ad_lab_adac_powershell.png)

### Getting Organized

Now before we create any users, let's establish some structure. Switch your view to "Tree View" by clicking the "Tree View" icon (it's the one under the "Ac" in Active Directory in the screenshot below) and then select our domain from the sidebar in ADAC. As you can see the domain comes prepopulated with a bunch of folders. These are Containers. We're going create our own containers, except we'll be creating  **Organization Units** or "OUs". The only difference between OUs and Containers that we care about is that you can't apply Group Policy to Containers. 

OUs and Containers are part of the LDAP standards and are used to organize the objects in a domain. Until you get really into AD, you won't have to worry about Containers again. _Just call everything an **"OU"** and you'll sound like you know what you're talking about._

![](/https://www.psattack.com/webhook-uploads/1468789009652/ad_lab_adac_overview01.png)

We're going to create a new OU that we'll use for our environment. In production, it's typical to not touch the default Containers too much and instead put all your objects in new OUs that make sense for the company. 

To create a new OU, right click on "Example (local)" and chose **New > Organization Unit**.

![](/https://www.psattack.com/webhook-uploads/1468789294707/ad_lab_adac_create_ou.png)

We're going to name this new OU **"Lab"**

![](/https://www.psattack.com/webhook-uploads/1468789328842/ad_lab_adac_create_ou02.png)

Next, repeat this process to create a **"Users"** and **"Computers"** OU under the newly created "Lab" OU. You can do this by right-clicking the "Lab" OU and then following the same process to create an OU.

![](/https://www.psattack.com/webhook-uploads/1468789370148/ad_lab_adac_create_ou03.png)

Once you're done, your domain should look like this.

![](/https://www.psattack.com/webhook-uploads/1468789547396/ad_lab_adac_create_ou04.png)

### Actually Creating Users

Ok, so let's create a user. To do this, **right-click the "Users" OU** that you created and choose **New > User**.

Fill in the **Full Name**, **User UPN Logon**, **User SamAccountName**, and **Password**. You'll notice some fields will fill in others as you type. You can also toggle the "Password Options" so that you won't have to change this password when you first log in.

![](/https://www.psattack.com/webhook-uploads/1468789634726/ad_lab_adac_create_user01.png)

Let us also add this user to the "Domain Admins" group. In the new user screen, **select "Member of"** from the sidebar.

1 - Click **"Add"** from the right-hand side

2 - In the "Select Groups" window that opens up type **"Domain Admins"** and **click the "Check Names" button**.

3 - The words "Domain Admins" should become underlined. **Hit "OK".**

![](/https://www.psattack.com/webhook-uploads/1468789797791/ad_lab_adac_create_user02.png)

## Create a Client Computer

Our domain isn't all that interesting with just a single Domain Controller, so let's add a client. Go ahead and [download a copy of Windows 10 from Technet](https://www.microsoft.com/en-us/evalcenter/evaluate-windows-10-enterprise). We're going to create a new VM. For client VMs I typically use 2 vCPUs, 1 GB of RAM and 30 GB of hard drive space. We'll also want to **make sure that this VM is put on the same host-only network that our other VMs were put on**.

Go through the install for Windows and accept the defaults. When the install is finished install the Virtualization Tools for your platform. If you're unsure of how to do any of this reference the instructions from when we did this for our [Domain Controller in part one](https://www.psattack.com/articles/20160718/setting-up-an-active-directory-lab-part-1/#lets-build-a-lab).


### Join the Domain

Once the install has finished and the computer has rebooted we're ready to join this computer to the domain. 

1 - Open up Explorer, **right click on "This PC"** and go to **"Properties"**.

![](/https://www.psattack.com/webhook-uploads/1468801518456/ad_lab_join_domain01.png)

2 - Click on **"Change Settings"** on the right-hand side.

![](/https://www.psattack.com/webhook-uploads/1468801534315/ad_lab_join_domain02.png)

3 - In the window that opens up, click the **"Change"** button and then name the Computer, switch **"Member of"** to **"Domain"** and enter **"example.com"**. Hit OK.

![](/https://www.psattack.com/webhook-uploads/1468801546238/ad_lab_join_domain03.png)

4 - You'll be prompted for credentials. You can use the "Administrator" account or the account that you created earlier in this article.

![](/https://www.psattack.com/webhook-uploads/1468801557818/ad_lab_join_domain04.png)

5 - After entering your credentials, the computer will join the domain and prompt you to reboot. Do that.

![](/https://www.psattack.com/webhook-uploads/1468801568474/ad_lab_join_domain05.png)

While the client is rebooting go to your Domain Controller and **open up ADAC**. Your computer should be in the "Computers" OU at the root of the domain. **Right click on the computer** and choose **"Move.."** and send it to the "Computers" OU under the Lab OU.

![](/https://www.psattack.com/webhook-uploads/1468801587084/ad_lab_join_domain06.png)

## Group Policy

Group Policy is one of the main reasons Active Directory has been so successful. It allows you to granularly configure User and Computer settings throughout a domain. For the purposes of this article we're just going to cover the very surface of Group Policy, but you're strongly encouraged to play with this in your lab. 

For our example, we're going to go ahead and turn off Windows Firewall for Computers in our "Lab\Computers" OU. While this is obviously bad practice, it's a nice example of Group Policy settings and is fairly typical in production Windows environments.

### Creating Group Policy Objects

1 - Go to your start menu and run **Group Policy Managment**

![](/https://www.psattack.com/webhook-uploads/1468802696910/ad_lab_group_policy01.png)

2 - Expand out Forest > Domains > Example.com > Lab. Right click the "Computers" OU and choose "Create a GPO in this domain..."

![](/https://www.psattack.com/webhook-uploads/1468802722394/ad_lab_group_policy02.png)

3 - Name the GPO "Firewall Rules"

![](/https://www.psattack.com/webhook-uploads/1468802831401/ad_lab_group_policy_03a.png)

4 - Right click the newly created GPO and choose "Edit"

![](/https://www.psattack.com/webhook-uploads/1468802859230/ad_lab_group_policy03b.png)

5 - In the "Group Policy Managment Editor" window that opens up, expand **Computer Configuration > Policies > Windows Settings > Security Settings > Windows Firewall..** and select **"Windows Firewall.."**. Click the **"Windows Firewall Properties"** link

![](/https://www.psattack.com/webhook-uploads/1468802906500/ad_lab_group_policy04.png)

6 - This brings up a standard Windows firewall settings window. **Set the Firewall to "Off"** and click OK.

![](/https://www.psattack.com/webhook-uploads/1468803057695/ad_lab_group_policy05.png)


### Things to know about Group Policy settings

Here is a quick rundown of things to know when working with Group Policy.

* Group Policy takes some time to take effect. Computers will check for updates every 45 minutes or so. You can speed this up by running `gpupdate` on the computer that you want to update You can also just reboot the computer, it will pull new updates when it boots.
* You'll notice that Group Policy settings are split between User and Computer and sometimes the same setting exists in both areas (for example, a lot of Internet Explorer settings). A common mistake is setting something in the Computer settings and then applying that GPO to an OU full of users (or vice versa). Those settings won't take effect since they have nothing to act on.

## Wrapping up

You now have a completely functional Active Directory lab! The world is your oyster. My suggestion for next steps would be to try and replicate configurations that you're used to. Here are some ideas: 

* Setup drives to auto mount when people log in.
* Deploy software through Group Policy
* Password Policies
* LAPS
* Startup scripts
* Add another server instance and play with different server roles
* Add a second domain controller
* Add multiple clients
* Play with roaming profiles

The more your lab replicates the sort of configurations that you encounter in your day to day life, the more useful this lab is going to be when you're playing with exploits or pivoting through a network.

I hope that you've found this series useful. If you have any questions or suggestions, hit me up on [twitter](https://www.twitter.com/jaredhaight)
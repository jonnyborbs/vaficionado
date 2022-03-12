---
id: 321
title: 'Configuring vRA 7 for 2 Factor Authentication'
date: '2016-02-22T12:14:32-08:00'
author: jon
layout: post
image: /assets/images/cropped-va-header.jpg
guid: 'http://www.vaficionado.com/?p=321'
permalink: /2016/02/configuring-vra-7-for-2-factor-authentication/
categories:
    - 'Compliance and Security'
    - Integration
tags:
    - 2fa
    - authentication
    - 'google authenticator'
    - mfa
    - 'multi factor'
    - pbis
    - radius
    - vidm
    - vra
---

One of the most exciting new features in vRealize Automation 7 is the addition of the VMware Identity Manager (or vIDM) to act as the identity provider. This brings a whole host of new capabilities, but one of the key among them is the addition of simple and flexible multi-factor authentication. This guide will walk you through the process of configuring vRA 7 for 2 factor authentication, using Google Authenticator as our example token.

In this example scenario, a vRA 7 environment is already set up and fully functional using traditional username and password authentication. The guide also assumes you have a basic CentOS server set up and available for configuration. Both of these steps are outside the scope of the instructions below.

# Part 1: Configuring the Linux Host

First, ensure that you have proper DNS resolution set up for the CentOS host. This host will act as your authentication intermediary, processing both the Active Directory username and passwords as well as the Google Authenticator token. Since AD is involved, DNS and time configuration will be critical.

![Create_DNS_Record](/vaficionado/assets/images/2016/02/Create_DNS_Record.png)

Next, SSH to your Linux host. You’ll need to be a privileged user (i.e. root) for these operations, so that’s what I’ve logged in as here. Your configuration may require you to log in as a standard user and then **su** to root, or use **sudo**.

![Log_In_Host_SSH](/vaficionado/assets/images/2016/02/Log_In_Host_SSH.png)

Next, you must edit the SELinux configuration.

```
vi /etc/selinux/config

SELINUX=disabled
```

Ensure that the SELinux policy is set to either **permissive** or **disabled** as shown below. While it is definitely possible (and probably advisable) to keep SELinux enabled for this configuration, the additional steps to do so are outside the scope of this guide.

![Edit_SELinux_Configuration](/vaficionado/assets/images/2016/02/Edit_SELinux_Configuration.png)

Now, remember what I said about DNS and time being critical when integrating with Active Directory? You’ll need to set up NTP services on your host to ensure that there’s no time drift for authentication to work properly.

```
yum install ntp -y
ntpdate <strong><your_ntp_server_here></strong>
```

This will install the **ntpdate** package on your host, as well as set the time source. Ensure that you set this NTP server to the same one that provides time to your Domain Controllers – this will prevent time related authentication failures.

![Install_Configure_NTP](/vaficionado/assets/images/2016/02/Install_Configure_NTP.png)

Now would also be a great time to confirm your DNS configuration is correct. Check the **/etc/hosts** file and ensure that your hostname is mapped to the correct IP address. Also check your **/etc/resolv.conf** file to ensure that your host is pointing at a DNS server which can properly resolve your Active Directory. In the example shown, the DNS server is set directly to our Domain Controller.

![Confirm_DNS_Config](/vaficionado/assets/images/2016/02/Confirm_DNS_Config.png)

Next, ensure that the **wget** package is installed. Your host may already have this installed.

```
yum install wget -y
```

![Install_wget](/vaficionado/assets/images/2016/02/Install_wget.png)

Great. The basic system is now set up and ready to start loading the software that will do the heavy lifting.

First up will be the PowerBroker Identity Suite, or PBIS. These utilities will enable the simple addition of AD authentication to your Linux host. The commands below will add the PBIS RPM repository so that yum can download and install the packages.

```
rpm --import https://repo.pbis.beyondtrust.com/yum/RPM-GPG-KEY-pbis
wget -O /etc/yum.repos.d/pbiso.repo https://repo.pbis.beyondtrust.com/yum/pbiso.repo
sed -i "s/mirrorlist=https/mirrorlist=http/" /etc/yum.repos.d/epel.repo
yum clean all
yum install pbis-open -y
```

![Install_PBIS](/vaficionado/assets/images/2016/02/Install_PBIS.png)

Now that PBIS is installed, you can join your Linux host to your AD domain.

```
domainjoin-cli join corp.local <strong><your_domain_username></strong>
```

This command will actually join the host to the domain, creating a computer object and all the required Linux PAM configuration. Ensure you use a username with the rights to add machines to the domain – in the example here, we used the default Administrator account.

```
/opt/pbis/bin/config AssumeDefaultDomain true
```

Next, this command will ensure that the domain you joined is always assumed to be the default. This saves you entering DOMAIN\\username notation for everything you do.

![Join_AD_Domain](/vaficionado/assets/images/2016/02/Join_AD_Domain.png)

Now open up your **Active Directory Users and Computers** snap-in. By browsing to the default **Computers** container, you can validate that your Linux host is now added to the Active Directory. In this example, it is named **util-01a** and is listed as a CentOS 6.3 host running PBIS Open 8.3.

![Validate_Domain_Membership](/vaficionado/assets/images/2016/02/Validate_Domain_Membership.png)

And while you’re in the ADUC view, create a domain group called **RADIUS\_Logon\_Disabled** in the **Users** container. You don’t need to add any users to it now – this will be used only if you want to deny any users the ability to authenticate against RADIUS without completely disabling their account. We’ll come back to this group later.

![Create_Denied_Logon_Group](/vaficionado/assets/images/2016/02/Create_Denied_Logon_Group.png)

Now, **reboot** your Linux host. This ensures that all of the PBIS configuration is in full effect.

Once the host is fully restarted, log in as the same privileged account you were using before. We’re not done yet!

```
wget https://packages.sw.be/rpmforge-release/rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
rpm -Uvh rpmforge-release-0.5.2-2.el6.rf.x86_64.rpm
```

This will enable another external repository, so that you can obtain the QR code generator that will be used with Google Authenticator…

![Prepare_RPMforge_Repo](/vaficionado/assets/images/2016/02/Prepare_RPMforge_Repo.png)

```
yum install qrencode qrencode-devel git pam-devel gcc -y
```

…and this will grab and install all the packages and dependencies needed to build the Google Authenticator components.

![Install_Build_Tools](/vaficionado/assets/images/2016/02/Install_Build_Tools.png)

Since the Google Authenticator utilities aren’t delivered as an RPM package, they’ll need to be built from source. To do that, you’ll download the source files from a Git repository and compile them directly on the Linux host. Don’t worry, this sounds a lot harder than it is.

```
cd /root
git clone https://code.google.com/p/google-authenticator
cd /root/google-authenticator/libpam
make && make install
```

This checks out the latest version of the Google Authenticator code, downloads it to your local system, compiles it and installs it. Easy, right?

![Download_Compile_Google_Authenticator](/vaficionado/assets/images/2016/02/Download_Compile_Google_Authenticator.png)

This is the last package installation, honest. Download and install the FreeRADIUS server, using:

```
yum install freeradius -y
```

![Install_FreeRADIUS](/vaficionado/assets/images/2016/02/Install_FreeRADIUS.png)

Now that all the packages are installed, there’s some configuration to be done. This part requires some config file editing, so be sure you’ve got your editor of choice handy and read the steps carefully – small mistakes can have big impact here!

First, the user which FreeRADIUS runs as must be changed. By default, the server executes as the **radiusd** user, but because we will need to read Google Authenticator tokens from every user’s home directory, it is far easier to run the service as **root** instead. There are of course other ways to make this possible without running as root, but they are outside the scope of this guide. In a production environment, you should definitely explore doing so.

```
vi /etc/raddb/radius.conf

Change:
user = radiusd
group = radiusd

To:
user = root
group = root
```

![Change_FreeRADIUS_User_Group](/vaficionado/assets/images/2016/02/Change_FreeRADIUS_User_Group.png)

Next, edit the **radiusd.conf** file to deny access to members of the AD group you created earlier. This is accomplished by adding the text shown here, in the “**Deny access for a group of users**” section of the file.

```
vi /etc/raddb/users

Add:
DEFAULT Group == "RADIUS_Login_Disabled", Auth-Type := Reject
 Reply-Message = "Your account has been disabled."

DEFAULT Auth-Type := PAM
```

![Configure_Denied_RADIUS_Users](/vaficionado/assets/images/2016/02/Configure_Denied_RADIUS_Users.png)

Now, FreeRADIUS must be configured to accept PAM-based authentication. PAM is the Linux Pluggable Authentication Module framework, and is what makes all of this fancy authentication possible.

```
vi /etc/raddb/sites-enabled/default

Uncomment the line shown so it just reads "pam"
```

![Enable_RADIUS_PAM](/vaficionado/assets/images/2016/02/Enable_RADIUS_PAM.png)

Once the FreeRADIUS server is configured to accept PAM authentication, PAM itself must be configured to use the correct mechanisms, in this case combining Active Directory authentication with Google Authenticator tokens. To do this, edit the **/etc/pam.d/radiusd** file and comment out all of the existing lines, then add the configuration below

```
vi /etc/pam.d/radiusd

Comment all existing lines by prefixing with <strong>#
</strong>
Add:
auth requisite pam_google_authenticator.so forward_pass
account required pam_lsass.so use_first_pass
```

![Configure_PAM_Modules](/vaficionado/assets/images/2016/02/Configure_PAM_Modules.png)

Finally, the RADIUS server must be configured to authenticate the vRealize Automation server itself. This is done by pairing a shared secret with the hostname of the system. Edit the **/etc/raddb/clients.conf** file and add the text specified in the section shown. Be sure not to add this new client definition inside the default definition. In the example shown, the client is **vra-01a.corp.local**, the secret is **VMware1!**, and the shortname is **vra-01a**. Fill in your specific details instead.

```
vi /etc/raddb/clients.conf

Add:
client <<strong>your-full-vRA-VA-hostname</strong>> {
 secret = <<strong>your-shared-secret</strong>>
 shortname = <<strong>your-vRA-VA-friendly-name</strong>>
}
```

![Add_RADIUS_Site](/vaficionado/assets/images/2016/02/Add_RADIUS_Site.png)

Now, start the FreeRADIUS server.

```
service radiusd restart
```

![Restart_radiusd](/vaficionado/assets/images/2016/02/Restart_radiusd.png)

# Part 2: Configuring vRealize Automation

Now that the Linux host is configured to process the authentication requests, you’ll need to configure vRealize Automation’s VMware Identity Manager instance to leverage it.

Log in to vRealize Automation as a **Tenant Administrator**.

Click on the **Administration** tab, then the **Directories Management** button on the left. Select the **Connectors** button and you’ll see the screen pictured. This is where you’ll configure the vIDM Directory connection. Click on **first.connector** as shown.

![Configure_vRA_Directory_Connector](/vaficionado/assets/images/2016/02/Configure_vRA_Directory_Connector.png)

You’ll be presented with the screen below. Click on the **Auth Adapters** button. Notice that the **RadiusAuthAdapter** is **Disabled**. Let’s change that – click on **RadiusAuthAdapter**.

![Configure_Auth_Adapters](/vaficionado/assets/images/2016/02/Configure_Auth_Adapters.png)

Here you’ll see the configuration for the vIDM RADIUS adapter. Fill in the fields as shown, substituting the correct **Radius server hostname/address**, **Shared Secret** (remember you entered this in an earlier step – in this example it was **VMware1!**) and **Realm prefix**. The Realm prefix is your domain name, with the trailing slash character. Also, note the **Login page passphrase hint** has been customized – this reminder will display on the login page to help guide users to enter the correct data.

Do not enable the **Secondary Server** at this time – leave all the rest of the fields as-is.

Click the **Save** button at the bottom of the screen, then switch back to the vRealize Automation tab in your browser.

![Configure_RADIUS_Adapter](/vaficionado/assets/images/2016/02/Configure_RADIUS_Adapter.png)

Now, click on the **Network Ranges** button and select **Add Network Range**. This will allow you to specify groups of IP addresses which will use a particular authentication config. It’s a good idea to configure just one or two IPs for testing purposes initially, so that you don’t accidentally lock yourself out of the environment.

![vRA_Network_Ranges](/vaficionado/assets/images/2016/02/vRA_Network_Ranges.png)

The Network Range configuration is pretty straightforward. Just enter a name, description and IP range. Make the starting and ending IP addresses the same to specify only a single host. In this example, we are limiting this range to the local desktop. Click **Save**.

![Add_Network_Range](/vaficionado/assets/images/2016/02/Add_Network_Range.png)

Click on **Policies** to the left and then edit the **default\_access\_policy\_set**. Remember that you can create multiple policies for multiple scenarios.

![vRA_Access_Policies](/vaficionado/assets/images/2016/02/vRA_Access_Policies.png)

Click on the **Green +** sign to add a new policy rule.

![Edit_Default_Policy](/vaficionado/assets/images/2016/02/Edit_Default_Policy.png)

Configure the policy rule as shown:

- If a user’s network range is: &lt;**your new network range here**&gt;
- And the user is trying to access content from: **All device types**
- Then the user must authenticate using: **Radius Only**

Click **Save**.

![Add_Policy_Rule](/vaficionado/assets/images/2016/02/Add_Policy_Rule.png)

Now, grab the icon highlighted in red with your mouse and drag the new rule to the top of the list. Click **Save**.

![Reorder_Policy_Rules](/vaficionado/assets/images/2016/02/Reorder_Policy_Rules.png)

vRealize Automation is now configured to use RADIUS authentication, combining both Active Directory credentials with a Google Authenticator token.

# Part 3: Enabling Users for Google Authenticator

Now that the Linux host has been built and configured and vRA has been set up to take advantage of it, you need to create tokens for your users.

Re-connect to your Linux host from Part 1 using SSH, or if you still have an active session simply switch back to it. You should be authenticated as root at this point, as you will be assuming the identity of your AD users to create their tokens.

In the pictured example, we are becoming a user named **mary**. Mary is an AD user who has never before logged in to this Linux host – yet we were able to assume her identity by authenticating against Active Directory. Pretty cool! You can also check that you are indeed logged in as Mary by running **whoami**.

```
su mary
whoami
```

![Su_to_AD_User](/vaficionado/assets/images/2016/02/Su_to_AD_User.png)

Now, you’ll create the Google Authenticator token. This can be done by running the following command:

```
google-authenticator -tdf -r 3 -R 30 -w 17 -Q UTF8
```

Notice that the command creates a huge QR code, a Secret Key, and 5 emergency scratch codes. These codes can be used in the event that you don’t have your smart device handy, but each can only be used once. Keep those in a safe place. The QR code is a graphical representation of the alphanumeric secret key printed directly beneath it.

![Create_Google_Authenticator_Token](/vaficionado/assets/images/2016/02/Create_Google_Authenticator_Token.png)

Here’s the fun part. Pick up your nearest handy smart device. It could be a smartphone, a tablet, etc. I use an iPhone, so the following images were captured there.

Search for the **Google Authenticator** app in the App Store, or Google Play, etc. Download it – it’s free. Open the app.

![Google_Authenticator_App_Store](/vaficionado/assets/images/2016/02/Google_Authenticator_App_Store.png)

Tap the “**Scan Barcode**” option and grant the application access to your camera. You can also select Manual Entry and type in the alphanumeric secret key – but where’s the fun in that?

![Set_Up_Google_Authenticator](/vaficionado/assets/images/2016/02/Set_Up_Google_Authenticator.png)

Point your phone at the QR code generated on the screen and the app will do the rest!

![Scan_QR_Code](/vaficionado/assets/images/2016/02/Scan_QR_Code.png)

You can see that the Authenticator app has automatically generated a token for Mary, showing her name and the server which she’s authorized for. The little Pac-Man thing to the right is a timer – these tokens are only good for a single use, and only valid for 30 seconds.

![View_User_Token](/vaficionado/assets/images/2016/02/View_User_Token.png)

# Part 4: Testing Your Work

To recap, you’ve just:

- Built a Linux host to handle the task of authenticating against Active Directory and Google Authenticator
- Configured vRealize Automation to leverage that host as an authentication source
- Created a time-based token for one of your Active Directory users

Now it’s time to put it all together and test the configuration.

Open a new browser window. I find that an ‘Incognito’ or ‘Private Browsing’ window works best, since you probably have another window logged in as your Tenant Administrator already. Notice that you are now prompted for the **username** and **AD Password + Google Auth Code** – that was the free text you entered a few steps back to help guide your users.

![Login_vRA](/vaficionado/assets/images/2016/02/Login_vRA.png)

Log in using the new token on your smartphone. Assuming the following parameters:

- Username = Mary
- Password (AD) = VMware1!
- Google Authenticator Code (from smart device) = 098765

You would log in with a username of “**mary**” and a passcode of “**VMware1!098765**”

![Login_Successful](/vaficionado/assets/images/2016/02/Login_Successful.png)

Voila!

This post was brought to you by Breakside Brewery’s Salted Caramel Stout, which was genuinely instrumental in getting me through developing this configuration. Notes of chocolate play on the nose while sea salt and fresh caramel round out the palate in one of the smoothest, most pleasant stouts I’ve ever tried.

![Breakside_Salted_Caramel](/vaficionado/assets/images/2016/02/Breakside_Salted_Caramel.jpg)

Happy automating!

Special thanks to Ed Kaczmarek for contributing to this guide – follow him at <span class="s1">@edkaczmarek!</span>
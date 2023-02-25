---
description: >-
  This tutorial will run you through the steps you need to take to setup a VPN
  server at home, allowing you to securely connect back in and manage your
  staking machine.
---

# Setting up home VPN access

If you're ever out and about and away from home for long periods of time, then this tutorial is for you!

_Question_ - Why should I set up a VPN server? It is easier to simply forward ports and connect in.

_Answer_ - A VPN server introduces another layer of security. To connect to your node via SSH, you first have to connect to your VPN and _then_ SSH into your node. This introduces another barrier and makes intrusions much more difficult.

### Requirements

* You will need a public static IP address or a DNS for your home network.
* This tutorial assumes you have Ubuntu Server 22.04 LTS installed and running. If not, no stress! You can follow the below link for a tutorial on how to do that.

{% content-ref url="../tutorials/installing-linux.md" %}
[installing-linux.md](../tutorials/installing-linux.md)
{% endcontent-ref %}

### Recommendations

* For security purposes, I recommend you set up the VPN server on a separate machine that is not your staking machine. This can even be a VM.

With all of that out of the way - Let's dive right into it!

### **Step 1 ) Run commands to ensure your machine is up to date.**

Please login to the machine and authenticate as superuser using the below command

`sudo -i`

Now execute the below command to ensure the OS and packages are up to date. Please upgrade any that aren't.

`apt-get update && apt-get upgrade`

<figure><img src="../.gitbook/assets/image (40).png" alt=""><figcaption><p>Your output will look like this if there are no upgrades needed.</p></figcaption></figure>

### **Step 2 ) Install OpenVPN dependencies.**

Please execute the below command.

`apt-get install ca-certificates wget net-tools gnupg`

<figure><img src="../.gitbook/assets/image (16).png" alt=""><figcaption><p>With all those installed, we are ready to prepare the machine for the VPN server.</p></figcaption></figure>

### **Step 3) Import OpenVPN GPG key and add the official repository.**

Execute the below three commands

* This command will load the OpenVPN public GPG key.

```
curl -fsSL https://as-repository.openvpn.net/as-repo-public.gpg | gpg --dearmor > /etc/apt/trusted.gpg.d/as-repo-public.gpg
```

* This command will add the OpenVPN access server repository to your machine.

```
echo "deb http://as-repository.openvpn.net/as/debian jammy main">/etc/apt/sources.list.d/openvpn-as-repo.list
```

* This command will check all configured repositories for updates.

```
apt-get update
```

<figure><img src="../.gitbook/assets/image (41).png" alt=""><figcaption><p>You'll now notice a new line for the OpenVPN access server repository after running "apt-get update".</p></figcaption></figure>

### **Step 4) Install OpenVPN access server.**

Now the fun begins! To install, execute the below command

```
apt-get install openvpn-as
```

<figure><img src="../.gitbook/assets/image (33).png" alt=""><figcaption><p>You may need to install a fair few packages for this one...</p></figcaption></figure>

Hooray, you've just installed OpenVPN access server! Pay attention to this part of the debug, it contains valuable information.

<figure><img src="../.gitbook/assets/image (44).png" alt=""><figcaption><p>Note the URL, the username and the password. Very important!</p></figcaption></figure>

The Admin UI is for making changes to the server config and adding users.

The Client UI is for your devices, you'll be able to download user profiles/certificates here. [We will do that part later.](setting-up-home-vpn-access.md#step-6-adding-user-accounts.)

### **Step 5) Configuring the server for remote access.**

Browse to your Admin UI URL. You'll receive a certificate warning, you can safely ignore this and continue. Once completed, you'll see the below UI.

<figure><img src="../.gitbook/assets/image (42).png" alt=""><figcaption><p>Please login here!</p></figcaption></figure>

Please login, read and accept the EULA and we are ready to go!

We need to make a few network changes, for this please navigate to Configuration > Network Settings

<figure><img src="../.gitbook/assets/image (4) (2).png" alt=""><figcaption></figcaption></figure>

#### Step 5.1) - Changing the VPN traffic port

Please find the "Multi-Daemon Mode" section, and edit both ports away from the default ports. This is for security purposes. These ports can be the same number. I picked 9514, but this is an example only, I recommend choosing your own ports.

<figure><img src="../.gitbook/assets/image (8) (2).png" alt=""><figcaption><p>"9514" is an example port only.</p></figcaption></figure>

#### **Step 5.2) - Adding in our public IP**

Please don't navigate away from the "Network Settings" page for now. But you will need to open the below URL in a new tab.

{% embed url="https://www.whatismyip.com/" %}
This website will show you your public IP address.
{% endembed %}

Copy your IP address from this website and paste it in the "Hostname or IP Address" field located at the top of the "Network Settings" page. This will already be populated with your _private_ IP address, you must overwrite it with your _public_ one.

<figure><img src="../.gitbook/assets/image (35).png" alt=""><figcaption><p>If you don't have a static IP, you will need a DNS for your home network.</p></figcaption></figure>

#### Step 5.3) Optional - Configure the admin UI and client UI to run on different ports.

This step is optional, but for security purposes I _heavily_ recommend it.

We are going to configure the admin UI and the client UI to run on different ports because we only need to publicly access the client UI.

On the same page "Network Settings", please scroll down to the bottom and find "Client Web Server" and toggle the "Use a different IP address or port" setting.

<figure><img src="../.gitbook/assets/image (17) (3).png" alt=""><figcaption><p>Please press "No" and turn it into "Yes".</p></figcaption></figure>

Now we can change which port we want the client web server to run on, you can make this any port of your choosing. I chose 9515.

<figure><img src="../.gitbook/assets/image (30).png" alt=""><figcaption><p>"9515" is an example port only.</p></figcaption></figure>

From here, please click "Save Settings" and then "Update Running Server"

<figure><img src="../.gitbook/assets/image (39).png" alt=""><figcaption><p>Please hit this button</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (2) (4).png" alt=""><figcaption><p>And also this button...</p></figcaption></figure>

Once the running server has been updated, you may need to refresh your browser and log back into the admin UI.

#### Step 5.4) Advanced users only - Adding a static route

**If you are an advanced user** - You \*\*\*\* may have setup your OpenVPN server on a _different_ subnet than your Ethereum nodes/validators.

If that is the case, you will need to browse to "Configuration > VPN Settings" and add in a static route for your validator network.

If they are not on separate subnets, please continue onto step 6.

<figure><img src="../.gitbook/assets/image (32).png" alt=""><figcaption><p>Advanced users only.</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (25).png" alt=""><figcaption><p>Advanced users only - Toggle "Yes, using Routing" and add in the subnet your Ethereum nodes/validators are on.</p></figcaption></figure>

### Step 6) Adding user accounts.

#### Step 6.1) - Create a new user and set a password

Please navigate to "User management" > "User Permissions".

From here, you can add a new user. Please type out a username and tick the "Allow Auto-login" box, then select the "More Settings" box.

<figure><img src="../.gitbook/assets/image (34).png" alt=""><figcaption><p>"Allow Auto-Login", and then "More Settings"</p></figcaption></figure>

You can now set a password for the account in the new options that appear when you click "More Settings".

<figure><img src="../.gitbook/assets/image (37).png" alt=""><figcaption><p>Make sure its a complex one!</p></figcaption></figure>

Once done, please "Save Settings" and "Update Running Server" again.

#### Step 6.2) Advanced users only - Allow access to another network

If you are one of the lucky ones that had to do [step 5.4](setting-up-home-vpn-access.md#step-5.4-advanced-users-only-adding-a-static-route), then you may also need to add your Ethereum node/validator subnet to the user account too.

<figure><img src="../.gitbook/assets/image (7) (2) (2).png" alt=""><figcaption></figcaption></figure>

### Step 7) Unblock local ports.

If you are using a local firewall (which you should be), you may need to unblock the local ports depending on how you have it configured.

These are the ports you set in [step 5.1](setting-up-home-vpn-access.md#step-5.1-changing-the-vpn-traffic-port) and [step 5.3](setting-up-home-vpn-access.md#step-5.3-configure-the-admin-ui-and-client-ui-to-run-on-different-ports.) + port 943 (The default admin port). You _can_ change the admin UI port if you wish, but as it doesn't have external access it is not really necessary.

```
sudo ufw allow 943
sudo ufw allow 9514 <Change to your "Multi-Daemon Mode" port>
sudo ufw allow 9515 <Change to your "Client Web Server" port>
sudo ufw enable
sudo ufw status numbered
```

Almost there!

### Step 8) Port forwarding.

For this step you will need to login to your router and forward ports to the machine running OpenVPN access server. The exact workflow is router dependent, so please search online for instructions and include your router make and model in the search.

You will need to forward two ports, both TCP/UDP.

* The port(s) you entered for "Multi-Daemon Mode" (I used 9514)
* The port you entered for "Client Web Server" (I used 9515)

Once completed, please browse to the below website and enter in your ports to check if they are forwarded correctly.

{% embed url="https://www.yougetsignal.com/tools/open-ports/" %}
This website will show you if your ports have been opened correctly.
{% endembed %}

<figure><img src="../.gitbook/assets/image (26).png" alt=""><figcaption><p>(IP Address redacted) - My VPN traffic port is open.</p></figcaption></figure>

<figure><img src="../.gitbook/assets/image (7).png" alt=""><figcaption><p>(IP Address redacted) - My client web UI port is open.</p></figcaption></figure>

### Step 9) - Configure a device for VPN access.

If you've made it this far, then congrats! You will be pleased to know that all the hard stuff is out of the way.

Please complete this step on the device you want to set up remote access.

Open the client web UI - You can use either your public or private IP for this step. In my case, I navigated to [https://192.168.3.111:9515](https://192.168.3.111:9515)

Once in, log in using the user account you created in [step 6](setting-up-home-vpn-access.md#step-6-adding-user-accounts.). If successful, you will see the screen in the below step.

#### Step 9.1 ) - Install OpenVPN client

Please select the OS you are using.

<figure><img src="../.gitbook/assets/image (12).png" alt=""><figcaption></figcaption></figure>

From here you can select a client for your device.

* If on Windows or Mac, it will automatically download the OpenVPN client software and guide you through the rest of the process.
* If on Linux, Android or iOS, it will take you to an external page with further instructions.

#### Step 9.2) Download the client user profile

Please download the "autologin profile".

<figure><img src="../.gitbook/assets/image (31).png" alt=""><figcaption></figcaption></figure>

Once done, you will have to import the profile into the OpenVPN software. The software itself (Windows or Mac) or external pages (Linux, Android or iOS) will show you how to do this.

### Step 10) - Connect to the VPN

Lucky you, I saved the easiest step for last.

* If using a laptop or desktop, please connect to another network such as a mobile hotspot.
* If using a phone, please disconnect from your WiFi and ensure you are connected to your telco's internet.

Now check your IP address at > [https://www.whatismyip.com/](https://www.whatismyip.com/)

Go back to the OpenVPN software and hit connect and you'll be connected in just a matter of seconds.

Now check your IP address again at > [https://www.whatismyip.com/](https://www.whatismyip.com/)

If the connection was successful, you will see it now matches your home IP as you are now connected to your home internet connection. From here you can securely SSH into your Ethereum nodes/validators.

If you can connect to your home network but are unable to SSH into your servers, you may need to tweak the firewall on your Ethereum node to accept incoming SSH connections _from the IP address of your OpenVPN server_.

## FAQ

FAQ to be added later!

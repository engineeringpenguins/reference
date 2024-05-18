# Virtual Networking Enviornments

## Introduction

When studying networking or designing a networking solution, it is essential to have some way of labbing up the technology in a rapid, accurate, and scalable way. This is where network emulators come in as you can build as many abstractions of hardware as you need provided you have powerful enough equipment. Network Simulators/Emulators are basically a front-end with networking focused features that interacts with some kind of hypervisor or a trained simulation model/engine. Note that the terms 'Simulation', 'Emulation', and 'Virtualization' can be used interchangeably by the community but are actually different concepts. Emulation is more about representing hardware in software, where simulation is conceptualizing the behavior of a system and using statistic to predict how real hardware would react and replicating that into the software, and virtualization is a self contained system that truly operates and functions (almost) the same way as physical hardware.  

This article is going to cover some of the more common networking emulators and how to use them at a high level.  

### Table of Contents

<ol>
  <li>
    <details>
      <summary><a href="#gns3">GNS3</a></summary>
      <ul>
        <li><a href="#gns3-introduction">GNS3 Introduction</a></li>
        <li><a href="#install-gns3">Install GNS3</a></li>
        <li><a href="#using-gns3">Using GNS3</a></li>
        <li><a href="#thoughts-on-gns3">Programming</a></li>
      </ul>
    </details>
  </li>
  <li>
    <details>
      <summary><a href="#eve-ng">EVE-NG</a></summary>
      <ul>
        <li><a href="#eve-ng-introduction">EVE-NG Introduction</a></li>
        <li><a href="#install-eve-ng">Install EVE-NG</a></li>
        <li><a href="#using-eve-ng">Using EVE-NG</a></li>
        <li><a href="#thoughts-on-eve-ng">Thoughts on EVE-NG</a></li>
      </ul>
    </details>
  </li>
  <li>
    <details>
      <summary><a href="#pnetlab">PNETLAB</a></summary>
      <ul>
        <li><a href="#pnetlab-introduction">PNETLAB Introduction</a></li>
        <li><a href="#install-pnetlab">Install PNETLAB</a></li>
        <li><a href="#using-pnetlab">Using PNETLAB</a></li>
        <li><a href="#thoughts-on-pnetlab">Thoughts on PNETLAB</a></li>
      </ul>
    </details>
  </li>
  <li>
    <details>
      <summary><a href="#cisco-packet-tracer">Packet Tracer</a></summary>
      <ul>
        <li><a href="#cisco-packet-tracer-introduction">Cisco Packet Tracer Introduction</a></li>
        <li><a href="#install-cisco-packet-tracer">Install Cisco Packet Tracer</a></li>
        <li><a href="#using-cisco-packet-tracer">Using Cisco Packet Tracer</a></li>
        <li><a href="#thoughts-on-cisco-packet-tracer">Thoughts on Cisco Packet Tracer</a></li>
      </ul>
    </details>
  </li>
</ol>

## GNS3

### GNS3 Introduction

Graphical Network Simulator 3 (GNS3) was the first network emulator that I actively used to lab and study. Its more complex to setup and manage than other emulators but its completely open source and has a massive community that is active in supporting its development. Standard deployments use a client/server model and all management and configuration post install is done from the client.   

### Install GNS3

GNS3 uses the KVM underlay for virtualization, installing on Ubuntu is extremely easy and the install can add the server and the client to the same machine. You can install GNS3 on Windows now but it will leverage Hyper-V to create an ubuntu VM for the GNS3 server. Easy isnt fun so for this tutorial I am using Windows but the steps are similar for Linux.  

1. Download GNS3 for Windows from [the download page](https://www.gns3.com/software/download)
   - An account is required to download but you dont have to agree to ALL of the SolarWinds bloatware
2. Run the installer, accept the default options
3. You will be asked to choose a hypervisor for the GNS3 Server VM
   - I used ESXI but it supports most of the popular hypervisors
4. Deploy the VM to your prefered hypervisor
5. Console into the VM from your hypervisor
6. Open the 'Security' menu
7. Enable authentication
8. Provide a username/password

### Using GNS3

After the Windows client has completed installation it should open the application automatically, if not open it manually. The first prompt asks how you want to deploy applications (how do you want to setup the server). You CAN run it on Windows locally now as I said earlier but for this demonstration we are going the traditional route.  

1. Choose to 'Run appliances on a remote server'
2. Provide server information
   - IP address, Port 80, Username/Password

You can now see the resource utilization of the GNS3 VM in the "Server Summary" pane on the far right. If you have multiple servers in a cluster running machines you could interact with them all from here.  

You can hand build your own images to use within GNS3 using KVM and exporting the VM. What is more common is to use GNS3 community templates.  

1. Download a community template from the [GNS3 Marketplace](https://gns3.com/marketplace/appliances)
2. Open the Router device type on the far left navigation
3. Click "new template"
4. Choose to "import an appliance file"
5. Select the downloaded template from step 1
6. Confirm "Install the appliance on the main server"
7. Select your desired image version
   - If your prefered image says 'Missing" for your files you can overide
     - Check the box for 'Allow Custom Images'
     - Click 'Import' and choose whatever files you have
   - Alternatively you can click 'Create a New Version'

![Deploy GNS3 Appliance](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/netsim/gns3-image.png)  

If you built your own device with KVM and want to do this the hard way just click 'Edit' in the top navigation header and click 'Preferences'. From there select the type of image (QEMU VMs for example) and click the '+ New' button. The wizard will guide you through allocating resources and uploading your disk image.  

### Thoughts on GNS3

GNS3 is extremely stable and has enough community involvement that professional support is available just by posting on the GNS3 forum. Typically I see people doing small Cisco labs in GNS3, for more complex and multi vendor setups GNS3 is not typically used. Now that SolarWinds has put marketing/branding everywhere and forces you to download their software I do not reccomend using GNS3.  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

## EVE-NG

### EVE-NG Introduction

Emulated Virtual Enviornment Next Generation (EVE-NG) is **THE** network virtualization platform used by the pros, I have never seen an expert do a networking demo with any other tool. Its a fork of UNETLAB and offers an open source option as well as a paid solution with more features. Its extremely vendor agnostic and thanks to its KVM underlay EVE-NG is capable of virtualizing almost anything. In contrast to GNS3, EVE-NG is managed through a web UI and does not have any client software. For small scale projects and simple designs GNS3 and EVE-NG are comparable and the best choice comes down to preference, however, in large scale designs or complex networks EVE-NG excels and should be used over GNS3.

![Example of an EVE-NG lab](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/netsim/eve-demo.png)  

### Install EVE-NG

The process for installing and configuring the free and paid solutions are nearly identical. I'm using the Community Edition (free) image for this demo but its worth noting that you cannot go from the free image to the pro image, you have to reinstall from scratch if you want to upgrade.

1. [Download the latest ISO](https://www.eve-ng.net/index.php/download/)
   - Grab the 'client side' downloads if you plan to connect to machines with desktop tools like VNC and Putty
2. Deploy the ISO to your hypervisor
   - Depending on hypervisor may need to set to BIOS or EFI
   - 200GB storage and 4CPU (virt and IO/MMU enabled) by 16GB RAM is pretty safe
3. Turn on the VM and console into it
4. Choose English and accept the config
5. Wait for EVE to reboot at least twice
6. Use the default login for initial setup
   - root eve
7. Configure the hostname, ip, dns, etc.
   - if using ESXI, disable security settings on portgroup
8. Connect to port 80 on the management ip and login with default creds
   - admin eve
9.  Use the 'Management' dropdown in the top navigation bar and click 'User Management'
10. Change the password for the admin user and create a new user for you to use

### Adding Images to EVE-NG

You use a special naming convention for the image files and SFTP them to a directory on your EVE instance. You can follow the steps for your specific image [using their howto page](https://www.eve-ng.net/index.php/documentation/howtos/) as it is unique to each device. You can create image files with your own configs or bootstrapping and virtualize a handful of CPU architectures.  

### Using EVE-NG

![Create a new lab](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/netsim/eve-labs.png)  

1. Click on 'Main' in the top navigation bar
2. Create a new lab using the icon that looks like a word document
   - You can also create folders by typing the name in the text field above the lab menu
   - You can Import/Export labs from here as well
3. Provide the name, author, description, and any optional details for your lab
4. Right click anywhere in the workspace and click 'New Node'
5. Use the dropdown to select the device type that you want
6. Provide your configuration for the node(s)
   - Use the image dropdown if you have multiple releases of the same device type
   - Multiple unites with the same name will have -x appended to whatever is used in name/prefix
   - Not every device uses Telnet so use the Console dropdown and change to VNC if things dont work
7. Save the configuration to deploy
8. Right click on the device and power it on
9. Double click on the device to open the console

![Deploy an image](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/netsim/eve-node.png)  

### Thoughts on EVE-NG

EVE-NG is king for a reason and the evidence is reinforced every time I use it. As a product it excells in ease of use and scaling up complex networks. Being browser based is a great feature as you can expose the web portal to the internet and remotely access it from any machine with a web browser (this can also be a negative). As a company they are notorious for poor customer support, out of touch executives, price hikes, and destructive updates so it is not the perfect product but for almost every situation I would reccomend using it over any competitor.  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

## PNETLAB

### PNETLAB Introduction

PNET LAB like EVE-NG is a fork of UNETLAB and provides a simple way to virtualize network architectures. This was one of the more contraversial options for network virtualization when it came out due to the dubious legality of their platform. Originally you would download community created labs from the application and all of the images and licenses used in those labs would come downloaded with the lab. They have since removed that feature but many tools remain that allow the same result. Its often criticized for feeling like a counterfit EVE-NG but people who use it often advocate that it does many things better than EVE-NG. Either way they are both forked from the same opensource project and do not credit the original author so interpret how you will.    

### Install PNETLAB

1. [Signup for an account](https://authen.pnetlab.com/auth/register/view) and verify your email
2. Download the latest OVA from [their downloads page](https://pnetlab.com/pages/download)
3. Deploy the OVA to your Hypervisor 
   - I am using ESXI 8.1 but the install image will work on any VMWare product
     - Currently only VMWare hypervisors are supported
   - Enable CPU virtualization for the guest VM
4. Turn on the VM and console into the it
5. use the default login for initial setup
   - root pnetlab
6. Provide your own root password when prompted
7. Set the hostname and ip assignment when prompted
8. Navigate to the ip of the vm in your browser using https
9. Select 'Online Setup'
   - Online Setup ties the install to your user account and integrates with the lab store
10. Login using the splash page that appears
    - If you have connection issues, ssh to the vm and troubleshoot linux networking

### Using PNETLAB

You can import .unl lab files similar to EVE-NG but a unique feature of PNETLAB is that you can also browse community labs from within the PNETLAB GUI. This makes installation extremely easy, its tied to your account so you can find and redownload your community labs on any PNETLAB install.  

Install a community lab

1. Log into web interface for PNETLAB
   - Change the dropdown from 'Console' to 'HTML'
2. Click 'Download Labs' in the navigation header
3. Select one of the labs that appear or go to the store to browse all labs
4. In the middle of the screen, click 'Get Lab' to download and import it

![Downloading a lab](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/netsim/pnet-dlab.png)  

When downloading a lab you no longer automatically pull all of the dependencies/images uses inside the lab. You will either need to swap the images in the lab to something you have downloaded or get any required images uploaded to the server. As its forked from the same source project you can use the same process described in the <a href="#adding-images-to-eve-ng"> EVE-NG section</a>.  

Use a community lab

1. Click 'Home' in the upper left navigation header
2. Select the lab you downloaded and click 'open' on the right hand menu
3. Mouse over the far left part of the workspace and click 'Setup Nodes'
4. Select the option to 'Configure Nodes'
   - Use the 'Image' column to replace or identify images to upload if needed
5. Confirm that all devices have a live image
6. Mouse over the far left and click 'Setup Nodes'
7. Select the option to 'Start all nodes'

If the author provided a 'Lab Guide' you can use the button in the lower left of the workspace to read through it. Right click anywhere in the workspace to add a new 'node' (image).  

![Workspace & Guide](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/netsim/pnet-space.png)  

### Thoughts on PNETLAB

Earns all of the critques from the community about being sketchy but also lives up to its reputation of having a superior interface and experience to EVE-NG. I love the idea of downloading community labs from within the web-ui and the open source nature of the project itself. While not as feature rich as EVE-NG, the things that it does do it does very well if not better than EVE-NG.  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>

## Cisco Packet Tracer

### Cisco Packet Tracer Introduction

Cisco Packet Tracer (CPT or more commonly called Packet Tracer (PT)) is ironically the most contraversial network simulator in this article. **EVERY** fledgling network engineer downloads packet tracer to start their Cisco journey. Anytime someone brings up CPT online you will see a schism among the network engineers and the community will collapse in on itself. Its a classic application that Cisco put out to help students study for certifications. While extremely polarizing the common sentiment is that CPT has everything you need to get the CCNA. Unlike other tools in this article CPT does not virtualize anything, it simulates behavior of Cisco hardware but you will find that the devices are not running IOS and most commands are not available or functional.   

### Install Cisco Packet Tracer

1. Navigate to the Packet Tracer [Download Page](https://www.netacad.com/portal/resources/packet-tracer)
2. Sign in or create a Cisco Network Academy account if needed
3. Download the executeable
4. Open the downloaded file and sign with your NetAcad creds

### Using Cisco Packet Tracer

![Device inventory](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/netsim/pt-devices.png)  

The lower left panel is where you set the type of object that you want to choose from. The top row represents the main categories and the bottom row represents the sub categories. You should default to the upper left button (Networking Devices) and the lower left button (Routers) selected. Whatever object type is filtered with these buttons will appear in the middle left panel.  

1. Click and drag one of the routers into the workspace
2. Select the 'Switches' button on the lower row in the bottom left panel
3. Click and drag one of the switches into the workspace
4. Select the 'Connections' button on the upper row in the bottom left pannel
5. Click the lightning bolt and connect the router and the switch
6. Use the options dropdown in the top menu header
7. Select 'Preferences' and enable 'always show port labels ...'

The coolest thing about packet tracer is that you can see the network hardware and actually swap out the available modules/expansion cards. 

1. Double click on the router
2. Power down the device by clicking on the power switch
3. Click and drag any expansion card into one of the open slots
4. Power up the device by clicking on the power switch
5. Click on the CLI button
6. Decline the initial setup wizard
7. Configure the router as needed

![Router Modules](https://github.com/engineeringpenguins/reference/blob/main/Linked-Images/netsim/pt-module.png)  

### Thoughts on Cisco Packet Tracer

While I believe that packet tracer is good enough for the CCNA, you have either outgrown packet tracer or you are just about to. Its certainly a great tool for starting out and is almost a rite of passage but it is not viable for professional use.  

<p align="right" style="font-size: 14px; color: #555; margin-top: 20px;">
    <a href="#introduction" style="text-decoration: none; color: #007bff; font-weight: bold;">
        ↑ Back to Top ↑
    </a>
</p>
<br>
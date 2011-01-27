<link href='style.css' media='screen' rel='stylesheet' type='text/css' />

# Simple Rails Cloud Server Setup (Part 1)

Part 1 [Part 2](rails_cloud_server_part_2.html) [Part 3](rails_cloud_server_part_3.html)

In the past setting up a Rails server was a bit more involved that it should have been. Thanks to work done by the guys over at Phusion Passenger (http://phusion.nl/) any rack compatible Ruby Application (Rails  2.2 or greater) can be easily deploy to either a Apache or Nginx Webservers. This article will take you from nothing to a fully functional, production ready Rails stack.
Tools of the trade

## Tools of the trade

* rackspacecloud account
* ssh
* rackspacecloud account
* arch linux
* rvm
* passenger
* iptables
* nginx
* mysql
* denyhosts

# Long Guide

This guide makes a few assumptions, first of which, that you have a good local development environment setup similar to the one described in My Local Ruby on Rails Setup. This guide also assumes basic familiarity with the command line, and that you want to use rackspaces cloud to host your application.

### Step 1: The Server

For most cases hosting a web application from your home connection is a terrible idea. Up time issues, outbound bandwidth issues, not to mention it violates certain ISP’s TOS. Other options are to host the application from a private data center, or co-location service. The upsides to this are usually, if everything goes well the potential cost could be cheaper, data remains on site which my be a requirement for some applications. One downside would be, managing your own hardware is usually a large upfront investment, both financially and time wise. Another, and in my opinion the deal breaking downside for most application is that if anything hardware wise goes wrong, it is more then likely you responsibility and time on the line to fix.

In-order to minimize downtime due to hardware failure, a common pattern is to setup a cluster of Virtualization servers (Xen, VMware etc.) with data stored on redundant network storage devices, if configured correctly this allows for quick fail over and some nice redundancy, the downside is cost and overhead in complexity. Another lesser known downside of hosting your own servers is a drastic increase in your companies insurance rates. Their are some other issues, such as capacity to scale in a timely fassion, security, power and IO redunancy and availability. Again, in some situations this makes plenty of sense, but in the majority of cases this is just pure needless complexity.

The solution this article suggests, is hosting your application with a reputable Cloud hosting provider. Such as [Amazon EC2](http://aws.amazon.com/ec2/), [RackSpace Cloud](http://rackspacecloud.com/), or Rack/Rails Specific [Engineyard Cloud](http://engineyard.com). Each of these provides charge an hourly service rate based on the server instance size.

This article will describe the processes using the Rackspace Cloud, hopefully if time permits, future articles will describe the process on EC2, and the greatly simplified process on the Engineyard Cloud. 

### a. The Signup
Clearly the first step is getting an rackspace cloud account, so head over to [Rackspace Cloud](http://www.rackspacecloud.com/) and signup. Give them your credit card information, but dont worry at approx 0.03 cents an hour, spinning up a new instance for this tutorial will cost you orders of magnitude less then the Grande Americano from Starbucks currently fueling you.

### b. Choosing the Right Image.
Rackspace current supports a large variety of virtualized operating systems, they currently include:

* Arch 
* CentOS 
* Debian
* Fedora
* Gentoo
* Oracle Enterprise Linux
* Red Hat Enterprise Linux
* Ubuntu
* Windows Server 2003 (32bit/64bit)
* Windows Server 2008 (32bit/64bit)

Instance specifications and pricing ranges:

    Linux:
    256   MB   10 GB  $0.015/hour
    to
    15872 MB  620 GB  $0.96/hour

    Windows:
    1,024 MB    40 GB  30 Mbps	$0.08/hour
    to
    15,872 MB  620 GB  70 Mbps	$1.08/hour
    
With set costs for servers, hardware maintenance, and the flexibility to add and remove as many instances as needed, its a clear cut business case win. Other bonus is, free internal IO between all rackspace products.

In this article we will be using the Arch 2010.05 image, Arch team takes an elegant and minimalistic approach to everything. In short this keeps complexity down, and transparency up, two very important attributes. Over the past few years I have deployed and maintain several arch servers, all running flawlessly, with the exception of a 3am Catastrophic Hardware failure on slicehost, which was resolved within 15 minutes (mad props to the slicehost guys).

Moving forward, choose the following image and image size:

    Arch 2010.05
    512 MB    20 GB @ $0.03 slice

Afew minutes later you should recieve a notification email, alerting you to the completion of your new Arch server, and it will include the IP Address, root login and password.

# Step 2 Meet your new app server.

With your IP address, and password in hand, its time to meat your new app server. So fire up the terminal, and issue the following commands.
    
    ssh root@<ip address of the server
    < enter password when prompted > 

Now that you are signed into your new app server, its a good idea to add your own account, as logging in as root is a terrible idea. To accomplish this issue the following commands:
    
    adduser <username of choice> 

Keep hitting enter until you are prompted for a password, fill in the password of your choice. As the administrator, you will most likely want the ability to issue the same privileged commands the root user is capable of, without having to log in as root each time. To accomplish this, we will enable the group "wheel", and add your account to the group. As a member of the wheel account, you will be able to issue the same privileged accounts as root, simply by prefixing the commands with sudo.

The first step will be to enable the wheel usergroup, to do so remain logged in as root and type:

    visudo
    or
    nano /etc/sudoers

Once the config file opens up, scroll down to

    # %wheel    ALL=(ALL) All

and uncomment the line, by removing the leading #.

At this point the wheel group has been set as the group for individuals capable of issuing sudo commands. The next step will be to add your new account to the group wheel. To do this, we will edit the groups file, and simply add our username to the appropriate line.

    vi /etc/group
    or
    nano /etc/group

Now search for the line that looks like
  
    wheel:x:10:root

and simply append your a comma, and your username to the end.

    wheel:x:10:root,<username you choose>


Your account has now been created, and is permitted, via the sudo command, to issue privileged commands. Go ahead, and feel free to logout of the root account, and start a new ssh session from your local machine. 

    ssh <username>@<ip address>
    

It is time to get familiar with Arch linux. First thing we will do is get familiar with the included package manager, pacman. This simple, fast, and easy to use binary package system is one of the main features of arch linux. Its goals, are to deliver a fast, simple, repeatable, and consistent way to manage software packages on arch.

Some useful pacman commands:

to update the system:
    sudo pacman -Syu

to install a new package:
    sudo pacman -Sy <package name>
    

Since you have a fresh install, it would be a great idea to quickly do a full system update. Their is a good chance pacman itself requires an update, if this is the case, simple upgrade pacman, then reissue the bellow command.

    sudo pacman -Syu

When I ran the command, it seemed some of the mirrors where quite out of date, some even (mirror.cs.vt.edu) failed entirely, the update did succeed, but we should fix this. Their where also some warnings thrown:

    warning: /etc/rc.conf installed as /etc/rc.conf.pacnew
    warning: /etc/pacman.d/mirrorlist installed as /etc/pacman.d/mirrorlist.pacnew
    warning: /etc/rc.conf installed as /etc/rc.conf.pacnew

Since we are running a default install these warnings, are easily correct as we are only dealing with defaults anyways. In my case, issue the following commands did the trick.
  
  
    sudo mv /etc/rc.conf.pacnew /etc/rc.conf
    sudo mv /etc/pacman.d/mirrorlist.pacnew /etc/pacman.d/mirrorlist
    sudo mv /etc/rc.conf.pacnew /etc/rc.conf

Now that we also updated the pacman mirror list, we may aswell use the rankmirror utility to optimize package updates even further. The tool works as described, simply ranking and ordering mirrors in /etc/pacman.d/mirrorlist in the order of best performance.

Now to help it out, we will jump into the mirrorlist, and comment all but the mirrors located relatively close to the data center. Since I know my cloud server is a resident of the United States of America, it makes little sense to enable the any of the Ukraine mirrors, and it makes plenty of sense to enable the american ones. 

So we will jump into the mirrorlist, and do some preflighting for the rankmirror tool, by uncommented all the mirrors in the united states, and commenting out all the rest.

    sudo vim /etc/pacman.d/mirrorlist
    

Now that we did the easy work, we will make rank mirror do the hard work for us, actually measuring the performance of each mirror, and ordering them appropriately. 

    sudo su
    mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.org
    rankmirrors -n 10 /etc/pacman.d/mirrorlist.org > /etc/pacman.d/mirrorlist
    exit

iptables

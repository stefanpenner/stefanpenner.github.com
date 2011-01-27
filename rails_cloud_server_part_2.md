<link href='style.css' media='screen' rel='stylesheet' type='text/css' />

# Rails Cloud Server Setup Part 2
## RVM and Nginx

[Part 1](rails_cloud_server_part_1.html) Part 2 [Part 3](rails_cloud_server_part_3.html)

Previously, we setup a Arch Linux 512mb of Ram Rackspace Cloudserver, got some of the basics sorted out. Now that we have the server setup, lets jump right in and get rvm and Nginx rolling.

## Tools of the trade

* tools of part 1
* rvm
* passenger
* Nginx
* mysql


### Dev tools
To get started on this we will install the dev tools we need to retrieve and compile some of the tools we need.

    sudo pacman -Sy gcc make patch git
    sudo pacman -Sy git

Create our deploy user, which will be an intermediate account that controls the web server, has the deployment rvm installation, can checkout the app, and has permission to checkout our app from version control. (Next Article, deployment).

### Deploy Account
    sudo adduser Nginx

when prompted for additional groups add wheel, this is a nice quick shortcut to add a user on user creation to a specific group. If you forgot to, you can always retroactively manually edit the /etc/group file.

Now that we have the Nginx account, lets switch accounts to the Nginx account. And start setting up our rails specific stack. Using su, we can run a shell with substitute user and group ID, essentially allowing us perform fast user switching.

    su Nginx

now we will jump to the Nginx users home directory

    cd ~

 Now as the Nginx, user we will essentially setup a remote environment very similar to the local development environment discussed in the past article(link), but with some key differences. 


### RVM (Ruby Version Manager)
Lets install rvm, with the following command, and then following the on-screen instructions
    bash < <( curl http://rvm.beginrescueend.com/releases/rvm-install-head )

Re-hash utility locations library, this should allow us to access to the new utility without having to start a new session
    hash -r

Test rvm, this should display the rvm usage.
    rvm

If testing rvm failed, quickly start a new session, and rvm should be kicking.
    exit
    su Nginx

with rvm installed we can easily install same exact same version and patch level Ruby as our local development environment
    rvm install ree

Just for consistency, set ree as the default ruby 
    rvm use ree --default

### Phusion Passenger
The easy and robust deployment of ruby on rails applications on apache and Nginx web servers brought to us by the Phusion guys.

In-order to ensure passenger and rvm play nicely, we will let rvm know which ruby we want to use for passenger.  The bellow call will generate wrapper scripts in RVMs bin directory (see Notes below). These wrapper scripts ensure environment variables such as GEM_HOME and GEM_PATH are set correctly for applications run by passenger.
  
    rvm ree --passenger
  
    gem install passenger
    
### Passenger and Nginx

The web server we will use is Nginx, it may not be known by all, but has proven incredibly stable, fast, and most importantly predictable. Current Nginx serves roughly 7% of the worlds domains, including some high profile domains including: WordPress.com and Hulu.com. Compared to its counter part apache, Nginx has a very low, and predictable memory footprint, which scales linearly with the number of concurrent requests, rather then apaches exponentially growing memory footprint. What this means for us, is that Nginx will perform faster, use significantly less memory, and stay out of the way.

sudo passenger-install-Nginx-module

to keep it simple, we will let the passenger gem compile and setup Nginx for us. In some situations one may need to compile custom modules into Nginx, but for us this should be fine.
  
    when prompted choose 1 (Yes: Download and install Nginx for me)

Testing Nginx, lets start by running Nginx.

  sudo /opt/Nginx/sbin/Nginx

By default Nginx should be serving from port 80, the default http port, so lets navigate over to your new appservers IP address in the browser. The resulting page should read.
    Welcome to Nginx!

Lets just kill Nginx for now
    sudo killall -9 Nginx

After this is done, we will need to tweek some of Nginx settings.

Lets get Nginx, setup as an arch daemon. The scripts that control specific daemon actions, such as start|stop|restart are located in.
    /etc/rd.d/*

As we want Nginx to start as a daemon on boot, it is a good idea to have it conform to this standard.

First off all we will need to have access to the Nginx process id, lets open up the Nginx conf file, and simple uncomment the line in question. This will have Nginx drop its pid to file, so we can get access to it from our rc.d script

    sudo vim /opt/Nginx/conf/Nginx.conf
    or
    sudo nano /opt/Nginx/conf/Nginx.conf

Uncomment
    #pid        logs/Nginx.pid;

Next lets drop in the appropriate daemon script for Nginx. I have provided one that should do the trick over at:
    https://gist.github.com/fe07eaac258c57022851

To drop use this simply

    sudo wget https://gist.github.com/raw/fe07eaac258c57022851/be31621ff47da247134606230c209bb1aba3c472/Nginx  /etc/rc.d/Nginx

Lets set the executable bit
    sudo chmod +x /etc/rc.d/Nginx

Now we can start and stop Nginx to our hearts content

    sudo /etc/rc.d/Nginx start

    sudo /etc/rc.d/Nginx stop

Now that we can make Nginx start and stop, it is a good idea to have Nginx start on boot, just in case our server has to be restart this should be automatic. This configuration is done from following file.

    /etc/rc.conf

open the file in your favorite text editor.
    sudo vim /etc/rc.conf 
    or
    sudo nano /etc/rc.conf 
    or
    ...

look for the daemon section, which in my case was at the very bottom
    DAEMONS=(syslog-ng network netfs crond)

and simple add Nginx to the list. Arch automatically looks in the /etc/rc.d/* folder for matching scripts and calls start on them. Scripts are loaded in the order that they appear in the daemons section of the rc.conf file.
    DAEMONS=(syslog-ng network netfs crond Nginx)


### RVM, Passenger, and Nginx

To take full advantage of rvm we should point to passenger the specially wrapped rvm binaries. To do so we will edit the Nginx configuration file.

    sudo vim /opt/Nginx/conf/Nginx.con
    or
    sudo nano /opt/Nginx/conf/Nginx.con

Find this line
    passenger_ruby /home/Nginx/.rvm/rubies/ree-1.8.7-2010.02/bin/ruby;

and replace it with with
    passenger_ruby /home/Nginx/.rvm/bin/passenger_ruby;


### Vhost for your App

Now that we have the server more or less setup, Nginx running, passenger and Nginx playing nicely, and rvm powering the ruby stack. Its time for us to test it all out, and make sure we can boot a simple rails application.

Nginx

For testing purposes we will just have the Nginx default host point to where our test rails application will bes public directory, and then enable passenger. This is nice and simple, and will helps us debug if their are any problems. 

To set the default host, open up the Nginx.conf in your favorite edit
    sudo vim /opt/Nginx/Nginx.conf
    or
    sudo vim /opt/Nginx/Nginx.conf

Now look for the first server block, replace the existing root directive with
    /srv/rack/example/public;

next, we will enable passenger for this block by simply adding the following line anywhere in the block:

    passenger_enabled on;

finally your initial server block should contain

    server {
      ...
      root /srv/rack/example/public;
      passenger_enabled on;
      ...
    }
Now lets start or restart our Nginx processing to make sure everything is loaded correctly

    sudo /etc/rc.d/Nginx restart

### Example Rails App

Now finally its time to test out your new stack with an example Rails application. 
First we will need to setup some simple directories that conform to those which we choose to specify in the Nginx.conf, and set the correct permissions.

    sudo mkdir -p /srv/rack/
    sudo chown Nginx /srv/rack

Now we can jump into the directory, install rails, and generate a nice fresh new rails application.   

    cd /srv/rack

Since we do not yet have rails installed we will need to do so in-order to generate the empty rails applications. Since this is a server install, we really dont need to install the rails documentation, so the --no-ri and --no-rdoc are appended to the gem install. This should speed the install up
    gem install rails --no-ri --no-rdoc
    rails example

Now that we generated rails, we will need to use bundler to install its dependencies, so Nginx can serve it.
    cd exmaple
    bundle install

And ... a full simple rails stack should be running. To verify this, surf over to your web servers IP address in your browser. You should see classy Welcome aboard rails fresh rails app start page.

### Mysql
By default rails uses sqlite3, but for most production cases this will not do the trick. One of the DBMS rails supports out of the box is mysql, so lets quick install and configure mysql.

    sudo pacman -Sy mysql

    sudo /etc/rc.d/mysqld start
    /usr/bin/mysql_secure_installation

Follow the on-screen instructions, set the root password, and answer yes to the rest of the questions. You can always come back to this later on.

More then likely, we will want Mysql to start at boot, if that is the case simple add mysqld to daemons list at /etc/rc.conf

    sudo vim /etc/rc.conf
    or
    sudo nano /etc/rc.conf

Find the line that looks like
    DAEMONS=(syslog-ng network netfs crond nginx)

And Simply add mysqld to that list, the result should look like
    DAEMONS=(syslog-ng network netfs crond mysqld nginx)

### Next article -- Version Control and Deployment.


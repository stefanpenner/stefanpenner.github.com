<link href='style.css' media='screen' rel='stylesheet' type='text/css' />
# Ruby Dev Environment.

Now a days Ruby on Rails seems to be all the rage, but with its popularity comes a fast moving platform, multiple implementations, various incompatibilities, and some strange terminology. This article will describe my setup, and the reasons behind my choices. 

## Tools of the trade

* Terminal
* [Latest Xcode](http://developer.apple.com/technologies/tools/xcode.html)
* [Homebrew](http://github.com/mxcl/homebrew) (OSX only)
* [Ruby Version Manager](http://rvm.beginrescueend.com/)
* [Git](http://git-scm.com/)
* [Gitx](http://gitx.frim.nl/)
* [Bundler](http://gembundler.com/)
* [railsapi.com](railsapi.com)
* [Texteditor](emac,vim,textmate)

# Quick Guide ([Long Guide](#long_guide))

#### 1. The Compiler
  
  Download and install the latest Xcode:
    http://developer.apple.com/technologies/tools/xcode.html 

#### 2. The Package Manager
  
  Install homebrew:
    ruby -e "$(curl -fsS http://gist.github.com/raw/323731/install_homebrew.rb)"

  Commands to know
    brew
    brew update
    brew install <package_name>
    brew uninstal <package_name>

### 3. Version Control
    
    brew install git

### 4. Ruby
  
  Installing RVM:
  
    bash < <( curl http://rvm.beginrescueend.com/releases/rvm-install-head )
    rvm update
    rvm reload

  Installing REE:
    
    rvm install ree

Setting REE as the default ( now in any terminal session, typing ruby will get you the REE ruby)”
rvm use ree --default

### 5. Dependency Management

  Install Bundler:
    gem install bundler --pre

### 6. Install rails
  
  For rails 3:
  
    #remove pre ones the Release candidate phase has finished
    gem install rails --pre 


  For older rails:

    gem install rails --version 2.3.8

<span id="long_guide"></span>
#Long Guide
First of all this article assumes OSX 10.6 is used although older version of OSX might work, homebrew and gitx will not work on Linux or Windows, in addition the Ruby version manager will not work on windows. I will discuss Linux and windows specific setups in later articles.

### Step 1. The Compiler
The default OSX install is without all the useful compilers and other development tools required for our goals. The folks at apple did bundle Xcode with the install cd’s, but this copy may be out of date, so if you have a nice fast connection jump over to http://developer.apple.com/technologies/tools/xcode.html, download the latest Xcode and grab a cup of coffee, unless you have an amazing 100Mbps home internet connection this step might take some time. After the download has completed simply install the dev tools.

### Step 2. The Package Manager
Now that we have the basic development tools, more specifically a c compiler, we are on our way. As a Rails developer one may need many opensource tools and projects, such as ack, mysql, mongodb, memcachd etc. For this we will use the new package manager by mxcl called Homebrew. Homebrew is a simple, powerful and easy to use package manager for OSX, and installing it is as simple as opening the terminal and typing.

    ruby -e "$(curl -fsS http://gist.github.com/raw/323731/install_homebrew.rb)"

  Commands to know

    brew
    brew update 
    brew install <package_name> 
    brew uninstal <package_name> 

Further reading
mxcl.github.com/homebrew

### Step 3. Version Control

As a developer it is paramount that code be strictly controlled, and maintained. This becomes even more important when working or collaborating with many others. To accomplish this I use the version control system Git (http://git-scm.com/). Since git and version control is a topic on to itself, in a later article I will describe some useful aspects of git and git work flows. Thanks to homebrew installing git is as simple as.

    brew install git

### Step 4. Ruby

Now that we have installed Xcode, homebrew, and git we can go ahead and install ruby. You might now be wondering, if Stef has gone crazy. Clearly OSX comes with ruby pre-installed, he even used it for step 2. Long story short, the ruby bundled with OSX is out of date, should not be used in production, and you should develop with the same ruby you plan to deploy on. 

### Step 4a. Choosing the right ruby

Let me introduce to you the rubies:

* MRI/YARV (ruby)
* Rubinius (rbx)
* JRuby (jruby)
* Ruby Enterprise Edition (ree)
* MagLev (maglev)
* IronRuby (ironruby)
* MacRuby (macruby)
* URABE Shyouhei´s Ruby (mput)

Hopefully in a later article I will describe each of the above ruby implementations in more detail, but for this article I will only briefly discuss MRI YARV and REE.

MRI, or Matz&rsquo;s Ruby interpreter is the original ruby interpreter, and the current version is 1.8.7-p302, currently has some memory management and performance issues. More or less fine for development and small short running scripts, but not recommended for production.

YARV, or Yet Another Ruby VM is meant to be the successor to 1.8.7, with major improvements, and the current version is ruby 1.9.2. Unfortunately their are some incompatibilities between 1.9.2 and 1.8.7 which break quite a few of the user contributed libraries, and therefore mainstream adoption has been slow. ( to see if your libraries work on 1.9 checkout http://isitruby19.com/) 

REE, or Ruby Enterprise Edition, simply put is fork of MRI compatible with Ruby 1.8.7 and with drastically improved memory management, it also offers copy on write memory which is ideal when running multiple rails instances in production. Due to the diligence and compatibility of the REE team, this is currently my ruby of choice.

### Step 4b. Installing a Ruby

Considering how many Implementations of Ruby in existence, thanks to RVM, or the Ruby Version Manager it is actually quite simple to install or toggle between any of them. Just as the name claims, RVM managers our ruby versions, and allows us to install and use many isolated ruby environment. Which is great for testing, and great when working on multiple projects, each with different requirements. RVM also leaves your system ruby untouched, and merely overlays new environment variables pointing to the ruby of your choice.

Installing RVM:
    bash < <( curl http://rvm.beginrescueend.com/releases/rvm-install-head )
    rvm update
    rvm reload

Installing REE
    rvm install ree

Setting REE as the default ( now in any terminal session, typing ruby will get you the REE ruby)
    rvm use ree --default

Further Reading:
rvm.beginrescueend.com
www.rubyenterpriseedition.com

### Step 5. Dependency Management

Until recently managing rubygems required by a Rails application could cause you some major headaches, luckily bundler (http://gembundler.com/) does the hard work for us. Bundler manages an application&rsquo;s dependencies through its entire life across many machines systematically and repeatably. Installing bundler is just as simple.

    gem install bundler

useful commands:
    bundle install

further reading:
gembundler.com

### Step 6. Install Rails.
I&rsquo;ll jump to the assumption that the reader knows what Rails is, so I’ll skip an introduction. Merely explain the current state of Rails. Currently the Rails 3 is going through is final release candidate phase, and should be finalized within days. New projects should be built using Rails 3, as it offers a much improved and stable API, performance improvements, awesome extendability. 

Installing Rails 3.
    gem install rails --pre

Obviously their are some older monolithic applications, which have not made the migration yet, and for those one can install a specific version.

    gem install rails --version=2.3.8


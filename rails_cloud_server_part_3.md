<link href='style.css' media='screen' rel='stylesheet' type='text/css' />

# Simple Rails Cloud Server Setup ( Part 3)

[Part 1](rails_cloud_server_part_1.html) [Part 2](rails_cloud_server_part_2.html) Part 3

Now that we have a Rackspace Cloud server with a useful rails stack running. It is about time for us to show you how to easily, and consistantly deploy your code. Although it is possible so use simple tools such as scp, or even ftp to simply transfer the working files to production, and trigger a reload of the environment, it is not good practice. First of all we are all human, and we make mistakes. If we can automate something simply, it will prevent simple or stupid mistakes from occuring. With more complex installations, a deployment could easily be across multiple machines, and be more then just source code changes. This article will describe one of the many methods and workflows used to deploy, manage, and even rollback production Rails environemnts. 

## Tools of the trade
* git
* github
* ssh-keys
* capistrano
* everything covered in [Part 1](rails_cloud_server_part_1.html and [Part 2](rails_cloud_server_part_2.html)

### Getting Started

This article assumes a running server as described in [Part 1](rails_cloud_server_part_1.html and [Part 2](rails_cloud_server_part_2.html), and a local development similar to the [Ruby Environment](ruby_environment) described in the the previous article. Once deployment is setup, it is fairly rock solid, and will ease future releasing, and hopefully unessessary rollbacks.

For this article, we will start by creating a simple local Rails application, and configure it to deploy to our remote server.

First we will need to install rails and bundler.
First lets make sure we are using the ruby we expect, this can be checked simply by typing

    ruby -v 
    // to see the version
  
    or 
    which -a ruby
    // to see the path

You should see ree as the current ruby, if use rvm to switch to the correct ruby and then set it as the default ruby.
    
    rvm use ree --default

Now that we are using the correct ruby, we need to install some dependencies.

    gem install rails bundler capistrano

and lets generate a dummy rails application

    rails new blog

    cd blog

now we should make sure all the gem requirements have been filled.

    bundle install

next, we woant all commands to use the bundled gem environment, to accomplish this we prefix each command with 
    bundle exec <regular command>
this ensures the correct gems are loaded.

To generate our first rails scaffold we simple
    bundle exec rails generate scaffold post name:string body:text

Run the database migrations, which will setup our posts table.
    bundle exec rake db:migrate

    bundle exec rails server

Lets checkout your super simple, and trivial blog, by surfing over to:
    http://127.0.0.1:3000/posts

### ssh-keys
We will be using ssh-keys for authentication... blurp. If you already have a key-pair feel free to skip this section.

    ssh-keygen -t rsa -C "<your@email.com"

Now your public key ( you can share this with others) exists at.
    ~/.ssh/id_rsa.pub

And, your private key ( do not share this with others) exists at.
    ~/.ssh/id_rsa

#### Further reading
*[http://help.github.com/mac-key-setup/](http://help.github.com/mac-key-setup/)

### Git
Git is a source control management system, it is a beautiful simple but incredibly powerfull tool. A highly recommended read is the [Git from the bottom up](http://ftp.newartisans.com/pub/git.from.bottom.up.pdf) paper, which nicely explains and describes whats going on internally.

For deployment, and collaboration (as mentioned in the [Distributed Workflow](distributed_workflow.html)) article) using a centralized git location is very useful. For this we will use [Github](http://github.com/). Github offers free plans, which allow unlimited public repositories, and paid plans, which offer a set number of private repositories, and also unlimited public repositores.

If you dont already have a github account, head over to [Github.com](http://github.com/), signup. Once you have signed up, its time to create a new repository on Github. It is not required that you setup a github repository first, you can always push any repository up to a new github account, if down the road you decide to. Now that you have a new github repository lets push your new simple blog to it, simply follow the instructions on empty github repository page to accomplish this.

At the time of this writing those steps where:
    // inside the root of your blog application
    git init
    git commit -m 'first commit'
    git remote add origin git@github.com:stefanpenner/blog.git
    git push origin master

Now if you refresh your blogs github repository page, the page should show the full directory structure and all the source files of your rails application.

#### More Git Info

*[http://help.github.com/](http://help.github.com/)
*[The Git Book](http://book.git-scm.org)
*[http://git-scm.org](http://git-scm.org)
*[Git from the bottom up](http://ftp.newartisans.com/pub/git.from.bottom.up.pdf)

### Capistrano
Capistrano is a deployment tool for written in ruby, initial use to deploy ruby applications, such as rails. But can be used for quite literaly anything, even remote server admin. By configuring Capistrano simple deploy.rb DSL, it will do all the heavy lifting for you. Such as, deployments, rollbacks, database migrations, starting and stoping daemons, and configuration environments.

    capify .

This generates the following files.
    ./Capfile
    ./config/deploy.rb

The first ./Capfile we can simply ignore, the second confg/deploy.rb is where we configure the magic. 


The first part is straight forward, lets give you application a name, specify the remote repository, and set the scm to git.
    set :application, "set your application name here"
    set :repository,  "git@github.com:<username>/<repo_name>.git"
    set :scm, "git"

Since we have our permissions setup correctly on our server, we should not need to use sudo when deploying
    set :use_sudo, false
    
This is where capistrano truely shines, multiple server deployments. By specificying seperate servers or arrays of servers for  :app, :web, :db, we can let capistrano handle deployment to N servers, database migrations to N servers, unfortunately our simple blog application does not (yet) need this scale of archicture, so we can simply bunch, :app, :web, :db together.

    server "<ip of your rackspace server>", :app, :web, :db, :primary => true

Since we are using RVM on our server, we will have to enable the rvm/capistrano helper. This automatically sets the correct environment variables to use the rvm Ruby associated with your passenger installtion.

    $:.unshift(File.expand_path('./lib', ENV['rvm_path'])) # Add RVM's lib directory to the load path.
    require "rvm/capistrano"  

    set :rvm_ruby_string, "ree@passenger"        # Or whatever env you want it to run in.
    set :rvm_type, :user

In the past, or under other system setups rails stack deployments required restarting various services, since we are using passenger as our webserver. We only need to touch a single file in your applications tmp directory, and on the next request passenger will automatically reboot our application.

    namespace :deploy do
      # passenger stuff
      task :start, :roles => :app do
        run "touch #{release_path}/tmp/restart.txt"
      end
    
      task :stop, :roles => :app do
        run "touch #{current_path}/tmp/restart.txt"
      end
      # end passenger stuff
      desc "Restart Application"
      task :restart, :roles => :app do
        run "touch #{current_path}/tmp/restart.txt"
      end
 
In order to run migrations with Rails using bundler, we need to make capistrano aware that it should use bundler.

      # helpers to migrate with bundler
      desc "Run the migrate rake task."  
      task :migrate, :roles => :app do
        run "cd #{current_path} && bundle exec rake db:migrate RAILS_ENV=#{rails_env}"
      end
    end


Now that all the hard work is done, we simply issue the following command in the local blog directory, and our application is deploy, and remote database configured.
  cap deploy:setup deploy deploy:migrate

#### Further Reading
*[http://www.capify.org/index.php/From_The_Beginning](http://www.capify.org/index.php/From_The_Beginning)




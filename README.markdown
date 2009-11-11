rails-nginx-passenger-ubuntu
============================

My notes on setting up a simple production server with ubuntu, nginx, passenger and mysql for rails.

Check Ubuntu Version with
    cat /etc/issue

Setup ssh public key authentification

    mkdir ~/.ssh
    touch ~/.ssh/authorized_keys
    chmod go-rwx ~/.ssh/authorized_keys

On your local machine:
  	ssh-keygen

Copy public key to remote machine:
    cd $HOME/.ssh
    cat identity.pub | ssh remoteuser@remotehost "cat >> .ssh/authorized_keys"

Now you should be able to log in to your server without a password: 
    ssh remoteuser@remotehost

Aliases
-------

    echo "alias ls='ls -FG'" >> ~/.bash_aliases
	echo "alias ll='ls -l'" >> ~/.bash_aliases
	echo "alias la='ls -a'" >> ~/.bash_aliases
	echo "alias lll='ls -al'" >> ~/.bash_aliases
	echo "alias psgrep='ps aux | grep'" >> ~/.bash_aliases
	echo "alias cdw='cd $WORK_DIR'" >> ~/.bash_aliases
	echo "alias gemi='sudo gem install --no-ri --no-rdoc'" >> ~/.bash_aliases
	echo "alias gemu='sudo gem uninstall'" >> ~/.bash_aliases
	echo "alias gem_search='gem search -r'" >> ~/.bash_aliases
    
edit .bashrc and uncomment the loading of .bash_aliases

If you have trouble with PATH that changes when doing sudo, see http://stackoverflow.com/questions/257616/sudo-changes-path-why then add the following line to the same file

    echo "alias sudo='sudo env PATH=$PATH'" >> ~/.bash_aliases
    

Update and upgrade the system
-------------------------------

    sudo apt-get update
    sudo apt-get upgrade

Configure timezone
-------------------

    sudo dpkg-reconfigure tzdata
    sudo apt-get install ntp
    sudo ntpdate ntp.ubuntu.com # Update time
    
Verify that you have to correct date and time with

    date

Configure hostname
-------------------

    sudo hostname your-hostname

Add 127.0.0.1 your-hostname

    sudo vim /etc/hosts
    
Write your-hostname in 
    
    sudo vim /etc/hostname
    
Verify that hostname is set
    
    hostname

Add the *deploy* user
---------------------


Install mysql
---------------

This should be installed before Ruby Enterprise Edition becouse that will install the mysql gem.

    sudo apt-get install mysql-server libmysqlclient15-dev
    
    
Gemrc
-------

Add the following lines to ~/.gemrc, this will speed up gem installation and prevent rdoc and ri from being generated, this is not nessesary in the production environment.

    ---
    :sources:
    - http://gems.rubyforge.org
    - http://gems.github.com
    gem: --no-ri --no-rdoc


Ruby Enterprise Edition
------------------------

Check for newer version at http://www.rubyenterpriseedition.com/download.html

Install package required by ruby enterprise, C compiler, Zlib development headers, OpenSSL development headers, GNU Readline development headers

    sudo apt-get install build-essential zlib1g-dev libssl-dev libreadline5-dev

Download and install Ruby Enterprise Edition

    wget http://rubyforge.org/frs/download.php/66162/ruby-enterprise-X.X.X-ZZZZ.ZZ.tar.gz
    tar xvfz ruby-enterprise-X.X.X-ZZZZ.ZZ.tar.gz 
    rm ruby-enterprise-X.X.X-ZZZZ.ZZ.tar.gz 
    cd ruby-enterprise-X.X.X-ZZZZ.ZZ/
    sudo ./installer
    
    
Change target folder to /opt/ruby for easier upgrade later on

Add Ruby Enterprise bin to PATH

    echo "export PATH=/opt/ruby/bin:$PATH" >> ~/.profile && . ~/.profile
    
Verify the ruby installation

    ruby -v
    ruby 1.8.7 (2009-06-12 patchlevel 174) [x86_64-linux], MBARI 0x6770, Ruby Enterprise Edition 20090928


Installing git
----------------

    sudo apt-get install git-core

Nginx
-------

    sudo /opt/ruby/bin/passenger-install-nginx-module

Select option 1. Yes: download, compile and install Nginx for me. (recommended)

When finished, verify nginx source code is located under /tmp

    $ ll /tmp/
    drwxr-xr-x 8 deploy deploy    4096 2009-04-18 17:48 nginx-0.6.36
    -rw-r--r-- 1 root   root    528425 2009-04-02 08:49 nginx-0.6.36.tar.gz
    drwxrwxrwx 7   1169   1169    4096 2009-04-18 17:56 pcre-7.8
    -rw-r--r-- 1 root   root   1168513 2009-04-18 17:51 pcre-7.8.tar.gz
    
Run the passenger-install-nginx-module once more if you want to add --with-http_ssl_module 

    $ sudo /opt/ruby/bin/passenger-install-nginx-module
    
Select option 2. No: I want to customize my Nginx installation. (for advanced users)

When installation script ask, "Where is your Nginx source code located?" Enter:

    /tmp/nginx-0.6.36

On, extra arguments to pass to configure script add

     --with-http_ssl_module
     
     
Nginx init script
-------------------

More information on http://wiki.nginx.org/Nginx-init-ubuntu

    cd
    git clone git://github.com/jnstq/rails-nginx-passenger-ubuntu.git
    sudo mv rails-nginx-passenger-ubuntu/nginx/nginx /etc/init.d/nginx
    sudo chown root:root /etc/init.d/nginx
    
Verify that you can start and stop nginx with init script

    sudo /etc/init.d/nginx start
    
      * Starting Nginx Server...
      ...done.
    
    sudo /etc/init.d/nginx status
    
      nginx found running with processes:  11511 11510
    
    sudo /etc/init.d/nginx stop
    
      * Stopping Nginx Server...
      ...done.
    
    sudo /usr/sbin/update-rc.d -f nginx defaults
    
If you want, reboot and see so the webserver is starting as it should.

Installing ffmpeg
-----------------

Uninstall x264, libx264-dev, and ffmpeg if they are already installed. Open a terminal and run the following:

	sudo apt-get remove ffmpeg x264 libx264-dev
	
Next, get all of the packages you will need to install FFmpeg and x264 (you may need to enable the universe and multiverse repositories):

	sudo apt-get update
	sudo apt-get install checkinstall yasm libfaac-dev libfaad-dev libmp3lame-dev libsdl1.2-dev libtheora-dev libx11-dev libxvidcore4-dev
	
Get the most current source files from the official x264 git repository, compile, and install. You can run "./configure --help" to see what features you can enable/disable.
	
	cd
	git clone git://git.videolan.org/x264.git
	cd x264
	./configure
	make
	sudo checkinstall --fstrans=no --install=yes --pkgname=x264 --pkgversion "1:0.svn`date +%Y%m%d`" --default
	
Get the most current source files from the official FFmpeg svn, compile, and install. Run "./configure --help" to see what features you can enable/disable.

	cd
	git clone git://git.ffmpeg.org/ffmpeg/
	cd ffmpeg
	git clone git://git.ffmpeg.org/libswscale/
	./configure --enable-gpl --enable-nonfree --enable-pthreads --enable-libfaac --enable-libfaad --enable-libmp3lame --enable-libtheora --enable-libx264 --enable-libxvid --enable-x11grab
	make
	sudo checkinstall --fstrans=no --install=yes --pkgname=ffmpeg --pkgversion "4:0.5+svn`date +%Y%m%d`" --default
	
That's it for installation. You can keep the ~/x264 and ~/ffmpeg directories if you later want to update the source files to a new revision.

You may eventually want to update to the newest revisions of FFmpeg and x264 if there are any major developments or changes:

	sudo apt-get remove ffmpeg x264 libx264-dev
	cd ~/x264
	make distclean
	git pull
	./configure
	make
	sudo checkinstall --fstrans=no --install=yes --pkgname=x264 --pkgversion "1:0.svn`date +%Y%m%d`" --default
	cd ~/ffmpeg/libswscale
	make distclean
	git pull
	cd ~/ffmpeg
	make distclean
	git pull
	./configure --enable-gpl --enable-nonfree --enable-pthreads --enable-libfaac --enable-libfaad --enable-libmp3lame --enable-libtheora --enable-libx264 --enable-libxvid --enable-x11grab
	make
	sudo checkinstall --fstrans=no --install=yes --pkgname=ffmpeg --pkgversion "4:0.5+svn`date +%Y%m%d`" --default

Installing ImageMagick and RMagick
-----------------------------------

If you want to install the latest version of ImageMagick. I used MiniMagick that shell-out to the mogrify command, worked really well for me.

    # If you already installed imagemagick from apt-get
    sudo apt-get remove imagemagick

    sudo apt-get install libperl-dev gcc libjpeg62-dev libbz2-dev libtiff4-dev libwmf-dev libz-dev libpng12-dev libx11-dev libxt-dev libxext-dev libxml2-dev libfreetype6-dev liblcms1-dev libexif-dev perl libjasper-dev libltdl3-dev graphviz gs-gpl pkg-config

Use wget to grab the source from ImageMagick.org.

Once the source is downloaded, uncompress it:


    tar xvfz ImageMagick.tar.gz


Now configure and make:

    cd ImageMagick-6.5.0-0
    ./configure
    make
    sudo make install

	sudo checkinstall --fstrans=no --install=yes --pkgname=imagemagick --pkgversion "6:5.svn`date +%Y%m%d`" --default

To avoid an error such as:

convert: error while loading shared libraries: libMagickCore.so.2: cannot open shared object file: No such file or directory

    sudo ldconfig

Install RMagick
 
    sudo /opt/ruby/bin/ruby /opt/ruby/bin/gem install rmagick

Test a rails applicaton with nginx
----------------------------------

    rails -d mysql testapp
    cd testapp
    
Enter your mysql password
    
    vim config/database.yml
    rake db:create:all
    ruby script/generate scaffold post title:string body:text
    rake db:migrate RAILS_ENV=production
    
Check so the rails app start as normal
    
    ruby script/server -e production

    sudo vim /opt/nginx/conf/nginx.conf
    
Add a new virutal host

    server {
        listen 80;
        # server_name www.mycook.com;
        root /home/deploy/testapp/public;
        passenger_enabled on;
    }
    
Restart nginx

    sudo /etc/init.d/nginx restart
    
Check you ipaddress and see if you can acess the rails application
        


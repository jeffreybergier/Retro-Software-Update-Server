![Retro Software Update Server](images/001-Title.png)

# Retro-Software-Update-Server
A guide on setting up an Apple Software Update Server Mirror in case Apple 
ever decides to shut down these old update servers. So far software update
still works on Mac OS X 10.4 and above, but Apple already shut down servers
for 10.3, 10.2, 10.1, and 10.0 and those can't be recovered.

If you have an interest in retro Macs, this may be something you would like
to set up in case something bad were to happen.

## Approach

1. Install Git, Homebrew, Python2
1. Choose where Software Updates will be mirrored on your server Mac
1. Configure the Web Server on your server Mac
1. \[TODO\] Configure Reposado on your server Mac
1. \[TODO\] Use Reposado to mirror Apple's existing software updates to your server Mac
1. \[TODO\] Configure the retro Mac to fetch updates from your server Max

## System Requirements

### Server Mac
1. 50GB+ to dedicate to storing updates
   - Even the minimal set for 10.4 Tiger is 20GB
   - Supporting all the way to 10.9 Mavericks takes 200GB
   - ChatGPT says that around 10.11/10.12 Apple started locking down the 
     software update server and it can't be changed
1. A normal home network
   - This guide assumes you connect all your Macs to the internet with ethernet 
     or WiFi and they can find eachother and connect to eachother using Bonjour 
     names.

### Retro Mac
1. Connected to Ethernet or WiFi on the same network as your server Mac
1. Mac OS X 10.4 or higher

Note that the server Mac can really be any system. Even modern Linux \(such as 
Debian and Ubuntu\) automatically identify themselves via Bonjour names. But
this guide gives instructions when using a Mac as the server.

# Guide

This guide pretty much runs 100% from Terminal. 
Any text `in this special format` is a terminal command you can copy and paste.

## 1. Install Git, Homebrew, Python2

- Git is included as part of the Xcode command line utilities AND these utilties
are required for Homebrew anyway.

```xcodeselect --install```

- HomeBrew is a simple package manager for macOS. Its easy to install. 
Just follow the guide on [https://brew.sh](https://brew.sh). Make sure to 
follow the instructions in the Terminal after installing HomeBrew.
They are required to make sure that commands you install via homebrew are
easily launchable from the Terminal.

```/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"```

### Python2

This is the difficult one. Python2 is far out of date and HomeBrew no longer
has an easy installer for it. We have to use PyEnv and this basically seemed
to build Python2 from source, which is never fun.

**Install PyEnv and other tools needed for compiling**

```brew install pyenv zlib bzip2 readline openssl@1.1```

**Configure the environment for python2 build**

```
export LDFLAGS="-L$(brew --prefix zlib)/lib -L$(brew --prefix bzip2)/lib -L$(brew --prefix readline)/lib -L$(brew --prefix openssl@1.1)/lib"
export CPPFLAGS="-I$(brew --prefix zlib)/include -I$(brew --prefix bzip2)/include -I$(brew --prefix readline)/include -I$(brew --prefix openssl@1.1)/include"
export PKG_CONFIG_PATH="$(brew --prefix zlib)/lib/pkgconfig:$(brew --prefix bzip2)/lib/pkgconfig:$(brew --prefix readline)/lib/pkgconfig:$(brew --prefix openssl@1.1)/lib/pkgconfig"
```

**Configure your shell to use PyEnv**

Sorry you may need to look up the commands
for how to use `vi` or use a text editor you know better like `nano`

```vi ~/.zshrc```

Paste in the following text and then save and exit

```
export PATH="$HOME/.pyenv/bin:$PATH"
eval "$(pyenv init --path)"
eval "$(pyenv init -)"
```

**Compile Python2**

```pyenv install 2.7.18```

**Set Python2 as the default**

```pyenv global 2.7.18```

**Test Python**

You should not get an error when running this command

```python -version```

## 2. Choose where Software Updates will be mirrored on your server Mac

The Software Update application on your Mac expects a very specific directory
structure and catalog format. Reposado creates this for you. But you still
need to decide where on your system this will go and you will also need
to configure the web server to use this directory. It is OK to have this on
an external drive as the mirror takes a lot of space.

On my system I put everything in 

`/Volumes/Data/Virtualization/SUS`

So the Web Server Document Root is

`/Volumes/Data/Virtualization/SUS/www`

Reposado git clone is

`/Volumes/Data/Virtualization/SUS/reposado`

Reposado Metadata storage is

`/Volumes/Data/Virtualization/SUS/reposado/code/metadata`

So you can put everything wherever you like, but this guide assumes this structure

## 3. Configure the Web Server on your server Mac

Mac OS X has always included Apache as a web server, its just disabled by 
default. Its nothing fancy, but it will work for our retro Mac.

**Set DocumentRoot to the directory you selected above**

`sudo vi /etc/apache2/httpd.conf`

This file is pretty long so you have to scroll down to find the DocumentRoot
setting. I don't think you need to change other options, but I here are the key
ones.

```
DocumentRoot "/Volumes/Data/Virtualization/SUS/www"
<Directory "/Volumes/Data/Virtualization/SUS/www">
    Options Indexes FollowSymLinks
    AllowOverride All
    Require all granted
</Directory>
```

**Give the webserver full disk access**

This might not be needed if you your directory root is on the boot drive
but I definitely needed it because mine is on an external drive.

1. Open System Settings→Privacy→Full Disk Access
1. Click the plus button and enter your admin password
1. Press ⌘+Shift+G in the file picker and type `/usr/sbin/httpd`

**Turn on the Web Server**

`sudo launchctl load -w /System/Library/LaunchDaemons/org.apache.httpd.plist`

**Test the Web Server**

Put a test file in the document root. It can be any file, but perhaps a small
`.txt` file is easiest.

1. Open Safari on your Server Mac and type `http://localhost/mytestfile.txt`
1. Open Safari on your Retro Mac and type ``http://Server-Bonjour-Name.local/mytestfile.txt`

Note that step 2 is critical. If you can't get the website to load on your
Retro Mac you won't be able to update from it either.

## 4. Configure Reposado on your server Mac

Ok, Reposado is where all the magic happens, so lets get started!

**Clone the [Reposado](https://github.com/wdas/reposado) Repository**

```
cd /Volumes/Data/Virtualization/SUS
git clone https://github.com/wdas/reposado.git
```

**Configure Reposado**

`/Volumes/Data/Virtualization/SUS/reposado/code/repoutil --configure`

Reposado will ask you for your directories. Note if you drag in the directories 
from the Finder, it appends a space after them and the developer of Reposado 
says this will cause errors.

```
Filesystem path to store replicated catalogs and updates [None]: /Volumes/Data/Virtualization/SUS/www
Filesystem path to store Reposado metadata [None]: /Volumes/Data/Virtualization/SUS/reposado/code/metadata 
Base URL for your local Software Update Service
(Example: http://su.your.org -- leave empty if you are not replicating updates) [None]: Server-Bonjou-Name.local:80
```

**To Be Continued**

## 5. Use Reposado to mirror Apple's existing software updates to your server Mac
## 6. Configure the retro Mac to fetch updates from your server Max

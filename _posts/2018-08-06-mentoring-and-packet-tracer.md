---
layout: post
title: "Mentoring and installing Packet Tracer 7.1 in Fedora 28"
tags: Fedora, Mentorship
---

I've been working on IT for the past 7 years. And if there's one thing I've learned for sure is that there will always be an infinity of technology to be learned so I keep on studying all the time. I've recently had the opportunity to try and mentor someone and, even though we don't have as much time for catch ups as I'd like I'm trying to improve that and provide more support. Having said that, I've proposed to him to study for the Cisco CCNA certification as a first step on the operations world - networks are a crucial knowledge for Infrastructure/Ops/SysAdmin Engineers all over and the Cisco Academy training does a good job on filling you in that world without needing a massive background in the area.
I've installed Fedora 28 on his laptop (because I use Fedora) and while teaching bits and pieces on the command line and sending him some articles as brain food we've got into the problem to install Packet Tracer on it. I remember I've succeeded on doing that many years ago, around Fedora 18 but it was also a pain in the arse.
After trying to follow a tutorial with him with not success I decided to install it myself and create my own tutorial so I can benefit my mentee and anyone else struggling with this installation.


I've followed massively the main tutorial you'll find when searching around - [This one installing Packet Tracer 7 to Fedora 27](http://www.bt0.ninja/cisco-packettracer-7-1-on-fedora-27/) so most of the credits go to this person, I've just tweaked what needed tweaking for it to work on Fedora 28.

Step to step
-------

1. As a first step just make sure you update the packages on your system
```bash
sudo dnf update
```

2. The massive list of libraries needed
```bash
sudo dnf install zlib-devel ncurses-devel gtk2 glibc glibc-devel  libpng12 libstdc++ libX11-devel libXrender libXrandr libusb  libXtst nss qt qtwebkit qt5-qtmultimedia qt5-qtwebkit
```

3. I've tried going down the best practice route compiling the openssl package based on the RPM for Fedora 17 (not really the best of best practices, I know...). I've faced a couple of issues doing that when building the package. First, found a dependency with Perl:
```bash
rpmbuild -bb openssl-lib-compat-1.0.0.spec
error: Failed build dependencies:
	perl is needed by openssl-lib-compat-1:1.0.0i-1.fc28.x86_64
```
Which I easily fixed installing Perl:
```bash
dnf install perl
```
But then the build command came to a compiler version issue that didn't have a trivial hack and would end up dragging me into a rabbit hole I'd rather avoid.
```bash
annobin: cryptlib.c: Error: plugin built for compiler version (8.0.1) but run with compiler version (8.1.1)
cc1: error: fail to initialize plugin /usr/lib/gcc/x86_64-redhat-linux/8/plugin/annobin.so
```
So I was lazy and used _BT0 Dot Ninja's_ package to get through with it:
One important point to notice is that the steps on that blog point to a `openssl` package that is not working, after digging around feedback in the blog I've found that these steps will get the package to solve the dependency:
```bash
wget http://bt0.ninja/rpm/openssl-lib-compat-1.0.0i-1.fc25.x86_64.rpm
sudo rpm -ivh openssl-lib-compat-1.0.0i-1.fc25.x86_64.rpm
```
  This will do the trick.

4. Get the Packet Tracer 7.1 package. I've got the one from [here](https://www.itechtics.com/download-cisco-packet-tracer-7-1-free-direct-download-links/), just scroll down and download the 7.1 version for linux, [or just click here](https://www.itechtics.com/?dl_id=24).

5. After downloaded, make sure you're at the same directory as the package you just downloaded, which is by default on Fedora the `~/Downloads/` directory:
```bash
cd ~/Downloads/
```

6. Let's unpack the tarball
```bash
tar -xzf PacketTracer71_64bit_linux.tar.gz
```
Just check if all the files are there
```bash
ls -al
```
Give execution rights to the install script
```bash
chmod +x install
```
Now let's install it
```bash
sudo ./install
```
This step will bring the EULA to be accepted (enter `Y` to accept it) and will ask where to install Packet Tracer, just hit enter to install it at the default path `/opt/pt/`.

7. Let's define the environment variables using Packet Tracer's installed scripts
```bash
sudo chmod +x /opt/pt/set_ptenv.sh
sudo chmod +x /opt/pt/set_qtenv.sh
sudo /opt/pt/set_ptenv.sh
sudo /opt/pt/set_qtenv.sh
```
8. Final dependencies to be met - this is using [robertpro's](https://github.com/robertpro/) tips to install Packet Tracer to Fedora 26 - all credits to him!
    ```bash
    mkdir ~/.lib64
    wget https://github.com/robertpro/tips/raw/59d14e7b148ebd10698ad3621b4c8a0bad38844b/packet_tracer_fedora26/libicudata.so.52 -O ~/.lib64/libicudata.so.52
    wget https://github.com/robertpro/tips/raw/59d14e7b148ebd10698ad3621b4c8a0bad38844b/packet_tracer_fedora26/libicui18n.so.52 -O ~/.lib64/libicui18n.so.52
    wget https://github.com/robertpro/tips/raw/59d14e7b148ebd10698ad3621b4c8a0bad38844b/packet_tracer_fedora26/libicuuc.so.52 -O ~/.lib64/libicuuc.so.52

    sudo sed -i "s|lib|lib:$HOME/.lib64|g" /opt/pt/packettracer
    ```

9. Now to set the graphical launcher, let's edit the file `/usr/share/applications/pt7.desktop`.
```bash
sudo vim /usr/share/applications/pt7.desktop
```
Delete all the current content in the file by hitting `dG` _(d + shift + g)_
Enter _INSERT_ mode by hitting `i` and paste the following config with _ctrl + shift + v_
```bash
#!/usr/bin/env xdg-open
[Desktop Entry]
Name=PacketTracer 7.1
Comment=Networking Cisco
GenericName=PacketTracer 7.1
Type=Application
Exec=/opt/pt/packettracer
Icon=pt7
StartupNotify=true
```
Hit _esc_ after pasting and _:wq_ + enter to save and quit the editor.
Make sure the changes were save by checking the contents of the file
```bash
cat /usr/share/applications/pt7.desktop
```
Which should match what you just added to it on this step.

Finally, Packet Tracer 7.1 is installed and ready to use on your Fedora 28. I agree it is quite an involved procedure and it took me many hours of my spare time to do it but fortunately it is the exception when it comes to applications for linux distros - most of the applications I need I have no trouble finding and installing.

Hopefully my mentee appreciates the effort and continues to study hard networking and having fun discovering things on the opensource world.

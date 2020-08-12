---
layout: post
title:  "Old laptop as lapdock"
categories: android retro
permalink: "/retro-lapdock/"
---
Running a modern IDE on a Pentium III laptop using a smartphone.

# Background
I bought a nearly two decade old laptop, originally to have a cheap interface with a raspberry pi.  
A brief summery of the specs:
- Model: Dell latitude C600
- Processor: single core Pentium III 750.0 MHz.  
- Memory: 128.0 MB PC133.  
- Display: 14.1 in TFT (1024 x 768).  
- IO: A lot, even IR and a single USB 1.0 port.
- It makes nostalgic sounds.
<audio src="/assets/c600_sound.mp3" controls> Unable to load song. </audio>



# Installing Linux on the laptop
This was a bit more of a hassle than I expected.
No matter what distro I tried it kept erroring out at some point.
I suspected the cd-rom drive might be faulty, but but even with [Plop](https://www.plop.at/en/bootmanager/intro.html) the laptop would not boot from a USB stick. Network booting was the solution so a house mate left it overnight in a closet with a raspberry pi that served the image.  
![image could not be loaded](/assets/c600_instl.jpg){: style="padding-top:16px; padding-bottom:16px"}  
In the morning I was greeted with Ubuntu!

# Installing Linux on the smartphone
There are many ways to install a chroot environment with linux on android, I was already running [Kali NetHunter](https://www.kali.org/docs/nethunter/) in [Termux](https://wiki.termux.com).

Installing NetHunter in Termux only takes a few commands and a lot of time depending on your phone and network connection.

{% highlight bash %}
pkg install wget
wget -O install-nethunter-termux https://offs.ec/2MceZWr
chmod +x install-nethunter-termux
./install-nethunter-termux  
{% endhighlight %}

# Finishing up
My first idea was using VNC but because of the USB 1.0 port this did not work well.
Luckily x forwarding works with minimal lag.
Connect the devices with a USB cable and enable USB tethering on the phone.
Run these commands on the phone:

{% highlight bash %}
#start NetHunter
nh
#install and setup the openssh server
sudo apt-get install openssh-server
sudo nano /etc/ssh/sshd_config
{% endhighlight %}
Edit the following settings:  
> Port 8023   
> X11Forwarding yes  

{% highlight bash %}
#start the openssh server
sudo /usr/sbin/sshd
#start the xfce4 session
xfce4-session
{% endhighlight %}
Run these commands on the laptop:
{% highlight bash %}
#restart network manager after starting USB tethering
sudo service network-manager restart
#start ssh connection with X Forwarding
ssh -L 5901:localhost:5901 kali@192.168.42.129 -p8023
#start x
xinit
{% endhighlight %}
  And with that we can now use xfce4 on the laptop!  
![image could not be loaded](/assets/c600_kali.jpg){: style="padding-top:16px; padding-bottom:16px"}  
I promised a modern IDE, fortunately java works great on arm so PyCharm and InteliJ work fine!
![image could not be loaded](/assets/c600_pycharm.jpg){: style="padding-top:16px; padding-bottom:16px"}  
Even though it's functional and I could definitely have learned C or Python on it, I wouldn't recommend this project for anything serious.
You could loose data when the USB disconnects and Samsung even broke USB tethering for a few months after I finished this project.

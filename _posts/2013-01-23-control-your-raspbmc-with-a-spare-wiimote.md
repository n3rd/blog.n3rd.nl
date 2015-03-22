---
layout: post
title: Control your Raspbmc with a spare Wiimote
tags: [Raspberry pi, Raspbmc]
description:
---

![logo](https://cloud.githubusercontent.com/assets/3578344/6769908/7c3e6a02-d0ae-11e4-8444-e04f222cbe0b.png){: .callout}

My Raspberry pi is used almost daily with Raspbmc, and I mostly control it with my android phone. I had some Wiimotes laying around and an old bluetooth device so i thought it would be nice to be able to use the those as an alternative.

First I started with a program called wminput, but I couldn't get it to work in XBMC, only in the console (what is pretty much useless). But then I read somewhere that I sould have used WiiUse_WiiRemote of which the source is in the XMBC repository. So I decided to try to build it and guess what? It works!

Here are the steps I executed to make the Wiimote work with Raspbmc:

First of all we need a USB bluetooth device and get it to work. Once you plugged in the device the output of the **lsusb** command should contain a bluetooth device.

Now we have to make the device work, run:

{% highlight bash %}
sudo apt-get update  
sudo apt-get install bluez  
{% endhighlight %}

If this gives you an error you can fix this by running:

{% highlight bash %}
sudo update-rc.d -f dbus defaults  
{% endhighlight %}

Now we have to get the XBMC source to compile WiiRemote, run:

{% highlight bash %}
sudo apt-get install libbluetooth-dev g++ libcwiid1 xbmc-eventclients-common make git-core  
mkdir /usr/local/src/xbmc  
git clone git://github.com/xbmc/xbmc.git /usr/local/src/xbmc  
cd /usr/local/src/xbmc/tools/EventClients/Clients/WiiRemote  
{% endhighlight %}

Now we have to do a small modification to the Makefile:

{% highlight bash %}
modify MakeFile to add "-l bluetooth" on the line "$(OBJS) -o $(BIN)"  
$(OBJS) -o $(BIN) -l bluetooth 
{% endhighlight %}

Now we can build it by running:

{% highlight bash %}
sudo make
{% endhighlight %}

We are ready to test if our setup works, to do this we stop XBMC and restart it when the Wiimote is connected run:

{% highlight bash %}
sudo initctl stop xbmc  
/etc/init.d/bluetooth restart  
cd /usr/local/src/xbmc/tools/EventClients/Clients/WiiRemote  
./WiiUse_WiiRemote  
{% endhighlight %}

Now press button 1 and 2 on the Wiimote to connect and when only the first led is on run:

{% highlight bash %}
sudo initctl start xbmc  
{% endhighlight %}

Congratulations, you should now be able to control Raspbmc with your Wiimote! The only thing left to do is make it start when you power on your Raspberry pi and optionally, only allow certain Wiimotes to operate Raspbmc.

Therefore we have to find the bluetooth address of the Wiimote, we can do this by running:

{% highlight bash %}
hcitool scan  
{% endhighlight %}

We create a file that will be run at startup:

{% highlight bash %}
nano /home/pi/wiimote.sh  
{% endhighlight %}

The contents will be:

{% highlight bash %}
#!/sbin/sh  
/etc/init.d/bluetooth restart  
sleep 2  
cd /usr/local/src/xbmc/tools/EventClients/Clients/WiiRemote  
./WiiUse_WiiRemote --btaddr XX:XX:XX:XX:XX:XX  
{% endhighlight %}

Where XX:XX:XX:XX:XX:XX is your Wiimotes bluetooth address.

Give the file execute permissions:

{% highlight bash %}
chmod +x /home/pi/wiimote.sh  
{% endhighlight %}

And edit rc.local to run it at start up:

{% highlight bash %}
sudo nano /etc/rc.local  
{% endhighlight %}

Just before the last line that reads "exit 0" add:

{% highlight bash %}
sh /home/pi/wiimote.sh &  
{% endhighlight %}
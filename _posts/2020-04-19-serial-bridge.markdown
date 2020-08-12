---
layout: post
title:  "USB Serial to HID bridge"
categories: password bad-usb
permalink: "/serial-bridge/"
show_excerpts: True
---
This device provides a straightforward and cheap way to inject keystrokes using your phone.

# Background
When you use a custom kernel it is possible to use USB-gadget mode on android phones.
This allows it to pose as usb devises such as a keyboard, this is also what oneplus phones use to provide a virtual cd drive with drivers.
I do have a phone running kali nethunter that runs a custom kernel that supports usb gadget mode but I wanted a less intrusive way to simulate a keyboard.

# Prototyping
To make this work I connected two Attiny85 with some resistors and a wire and taped it to an old gift card. \
![image could not be loaded](/assets/step0.jpg){: style="padding-top:16px; padding-bottom:16px"}    
This worked fine for testing the code, and learning that you can't use pin5 as input  on this board because it is connected to reset.
However it looked terrible so I rewired it and taped it to a smaller piece of plastic. \
![image could not be loaded](/assets/step1.jpg){: style="padding-top:16px; padding-bottom:16px"}    

# The result
To make the device smaller, I soldered two boards together using the power pin header.
This connects the grounds which gets rid of the wire but also connects the vin and 5v of the two boards together. To prevent a short circuit I cut the traces going to the 5v pins.
The power led is tied to the 5v pin so you can use this to check if the connection is cut.  
![image could not be loaded](/assets/step2.jpg){: style="padding-top:16px; padding-bottom:16px"}    
I also edited a USB serial app to streamline the experience(it opens automatically when it recognizes the PID of the cable), but the device should work fine with any USB serial app.

# Code
This is where I would put my code, if I still had it. (Make backups!)
I will rewrite it in the future.
{% highlight C %}
void setup() {
Serial.begin(9600);
Serial.println("Hello World!");
}

void loop() {
}
{% endhighlight %}

# Final notes
My main motivation for this project was not having to manually copy passwords from my phone to computers at university, however this might not be a safe practice.
People can't look over your shoulder when you are typing in your password but a shady app might be monitoring your clipboard.

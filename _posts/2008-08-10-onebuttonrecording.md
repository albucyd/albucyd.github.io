---
layout: post
permalink: /2008/08/10/onebuttonrecording/
title: OneButtonRecording
description: A simple GUI to record using Jack Audio and Ardous
date: 2008-08-10 13:50:58 -0000
last_modified_at: 2008-08-24 14:06:56 -0000
publish: true
pin: false
categories:
- Ausland
- Hacking
tags:
- ardour
- hammerfall
- jack
- streaming
---
Its taken a long while. At [ausland](https://ausland-berlin.de) we wanted to be able to easily record our shows. In addition we wanted to be able to stream any show through our webserver. Early on we had dubbed this project 'One Button Recording'. The idea was that it should very easy to use so that even people with little or no knowledge about the involved technologies could do it. The hardware used is:

* a donated pentium 4 2.8Ghz computer
* two condenser oktava microphones
* a microphone preamp build from a kit
* a donated hammerfall dsp soundcard

Since I am the 'IT departement' at ausland and since I had stopped using Windows some time ago I decided to use Linux for this project. I've used debian on servers, but on my desktop machines I use Ubuntu exclusively. So I decided to put this project together using [Ubuntu Studio](https://ubuntustudio.org/ "Ubuntu Studio") 8.04. Basic setup was simple: Ubuntu recognised the Hammerfall DSP soundcard. The Hammerfall DSP card works best with [Jack](https://jackaudio.org/ "Jack Audio") and [Ardour](https://www.ardour.org/ "Ardour Digital Audio Workstation"). But since this soundcard is multichannel, being able to record anything meant that all users had to have at least some idea about how to route channels. Our 'Audio departement' Elle quickly created a template for Ardour, but being able to record anything still involved something like 12 steps. So the first item on my list was to open ardour with a new session that used the template. Sadly this wasn't possible from the commandline. The solution was to to first build the new session files from an empty session created using the template. Then ardour could be started using these session files. This would leave the user with an ardour window where simply clicking on the record button would start recording what come in from the microphones. Of course this didn't cover streaming our recorded sounds. For streaming I decided to use icecast2 on the server and darkice an the audio workstation. Darkice works with Jack, but it didn't connect the right sound card inputs with the darkice outputs. I looked around and found [python bindings for Jack](https://sourceforge.net/projects/py-jack/ "pyjack") that offered a way to manipulate the connections within jack. Since those were python bindings I decided to write my little application in python. I knew that python bindings existed for gtk+, allowing me to easily create GUI for my software. I had not worked with python before but that actually added to the challenge. 
{% include image.html url="/assets/wp-content/uploads/2008/08/recording-und-streaming-gui-300x217.png" description="recording-und-streaming-gui" full_url="/assets/wp-content/uploads/2008/08/recording-und-streaming-gui.png" alt="recording-und-streaming-gui" %} 
The GUI was to be as simple as possible: Two buttons: one that started and stopped darkice and one that started ardour. I used [Micah Carricks pygtk tutorial](https://www.micahcarrick.com/12-24-2007/gtk-glade-tutorial-part-1.html "Pygtk Tutorial") to get me up and running for the GUI. This worked fairly well once I understood the basic layout principles of GTK, even though I did run into some problems using the gtk-builder. I think I had messed up my glade file doing some copy-paste in Glade and builder did not create a gui from my xml. I suspect that some of the widget ids were not unique. However I didn't really check that and instead simply rebuild my GUI in glade this time being careful not to duplicate any ids. After finishing the GUI I created two new python classes, one to create the session files and start ardour, the other one to start darkice and manipulate the jack connections. All of this was pretty straight forward, the only bigger problem I ran into was that I needed my software to wait for some time after starting darkice before I could create the necessary jack connections. I solved this using the [pexpect python module](https://www.noah.org/wiki/Pexpect "Pexpect"), which also gave me functions to monitor health of the started subprocesses. The one thing I did not manage was to pipe the output of the darkice subprocess into a gtk textview. I found some [fairly simple example](https://python.net/pipermail/python-de/2005q2/006611.html "\[Python-de\] Pipes ") on how to do this, however this did not work for me. Maybe I'll investigate this further some other time. I'll be cleaning up the sources and move the hardcoded path and file informations into a config file and then I'll be posting it on this blog, maybe it will be of use to somebody with a similar setup. **Update:** Here is the [recordinggui](/assets/wp-content/uploads/2008/08/recordinggui.zip). I didn't do much cleanup. But I'm sure you can figure it out if needed.

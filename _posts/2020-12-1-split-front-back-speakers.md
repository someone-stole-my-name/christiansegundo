---
layout: post
title: Split front and back speakers
category: Gnu-Linux
tags: 
- pulseaudio
---

I recently ditched USB headphones altogether and switched to regular single jack headphones+microphone like the ones that come with most phones and most people have dozens lying around.
My setup is very simple, headphones and mic plugged into front panel and speakers plugged into rear jack. 

One big issue I found with this setup, is that front and back jacks are not different outputs,
it may be an issue with my audio controller (Intel Corporation 7 Series/C216 Chipset Family High Definition Audio Controller (rev 04)) 
because I couldn't find any documentation on how to split front and back output jacks into different 'devices' to route audio to or what I found didn't work. Here is what did work for me...

## Pulseaudio magic

### Identify your card

```
$ pacmd list-cards
...
    index: 0
        name: <alsa_card.pci-0000_00_1b.0>
        ...
        active profile: <output:analog-surround-40>
...
```

In my case the card name is `alsa_card.pci-0000_00_1b.0` and it is using the `output:analog-surround-40` profile, make sure you are using the same profile. It can be set using `pactl`:

```
pactl set-card-profile <cardindex> <profilename>
```

### New sinks

Create two new sinks by editing either your system-wide `default.pa` file or you local user's (`~/.config/pulse/default.pa`). If using local user settings, remember to include system-wide settings at the top (`.include /etc/pulse/default.pa`).
You have to modify the `master=` setting, so it matches the name of your card.

```
load-module module-remap-sink sink_name=speakers sink_properties="device.description='Speakers'" remix=no master=alsa_output.pci-0000_00_1b.0.analog-surround-40 channels=2 master_channel_map=front-left,front-right channel_map=front-left,front-right
load-module module-remap-sink sink_name=headphones sink_properties="device.description='Headphones'" remix=no master=alsa_output.pci-0000_00_1b.0.analog-surround-40 channels=2 master_channel_map=rear-left,rear-right channel_map=front-left,front-right
```

### Retask jacks

Using `hdajackretask` override `"Green Hadphone, Front Side"` and select `Line out (Back)`:

<center><img src="/images/hdaretask.png"></center>

Click `Install boot override`, reboot and enjoy simultaneous separate audio streams between front and back panel.

----

---
title: "Privacy first fridge camera empowered with machine learning so I know what to get during grocery shopping"
date: 2021-12-09T21:29:01+08:00
lastmod: 2021-12-19T21:29:01+08:00
description: "The result of this privacy first smart home article will explain how to get the feature to see what is in your fridge while you are doing your grocery shopping with open source solution."
resources:
- name: "featured-image"
  src: "featured-image.jpg"
  
  
page:
    theme: "wide"

tags: ["home assistant", "esphome", "doods", "tensorflow"]
categories: ["documentation"]

lightgallery: true

toc:
  auto: false
 
draft: true
---

The result of this privacy first smart home article will explain how to get the feature to see what is in your fridge while you are doing your grocery shopping with an open source solution.
<!--more-->
# Timelapse of the result {#timelapse-id}
{{< image src="fridge-snapshots.gif" title="Fridge timelaps" alt="fridge snapshots"  width="50%" height="50%" >}}
## Pre-requirements {#pre-requierments-id}

### Hardware {#hardware-id}
I used a m5cam as I already had one, to my surprise when I was using it for another project I bumped into heading issues and found out that it is a known issue when running it 24/7. So for me, it is a perfect use case as I get cooling out of the box ;-) If you have a powerful cpu (and not a raspberry pi) you would not need a google coral usb hardware if you don't have the need to ensure to 100% that you get a snapshot every time you open your fridge.

### Software {#software-id}
You need to setup [doods](https://github.com/snowzach/doods/), [esphome](https://www.esphome.io/), [Home Assistant](https://www.home-assistant.io/installation) and some remote/vpn capabilities to home assistant. My setup is built with privacy in mind, there for I use solutions I have full control of so I use my own vpn. I would not recommend exposing your Home Assistant directly to the internet if you are not good at ensuring you always have the latest version due to security patches. If you don't want to build your own vpn I would recommend the cloud function of Home Assistant, [Nabucasa](https://www.nabucasa.com/). It is supposedly built with [privacy](https://www.nabucasa.com/privacy/) in mind and would support the company behind Home Assistant.

## The story {#the-story-id}
During the pandemic I had a lot more time for obvious reasons so I decided to build something with my m5cam, basically a esp32 hardware with a camera. I managed to get esphome to with with it and easily get it connected to Home Assistant, I had an idea on monitor on who does the dishes the most but identify who's hand it is above the dishes was a lot harder than I thought (please help me if you know how!) and I did run into heating issues when I was running it for 24/7. So I shifted focus to see if I could put it somewhere that would automatically cool it without me needing to add fans and such, so the fridge was the natural place to test.

I started to wire it and put it with the angel to capture the opening of the fridge when it is open so I mounted it to the door part of the fridge. It was fun to see the stream when I was opening and closing the fridge but I didn't really want to see a dark picture when I was grocery shopping. So step two was to ensure I always had a image of latest opening of the fridge, I started to test with some switches and motion detection functionality to capture the picture when it is open but I did not get it to work that good as it was almost always blurry, the door was also not in the "perfect state" to capture the view of the fridge and I would block the view when I was taking grocery in or out. 

To solve that, I found that there is a tensorflow models that supports the object fridge so I configured [doods](https://github.com/snowzach/doods/) and tested to post a example image from the fridge when the door is open through curl and one of the objects detected was fridge. I started to setup the [integration between home assistant and doods](https://www.home-assistant.io/integrations/doods/)

The last step was to show the latest captured image in Home Assistant, the doods integration has some options for the file_out, setting dynamic parameters to the filename. So I have one with the time/date in the filename and a second one that is named latest. With that I can use Home Assistant to emulate a camera by reading the local file and then show it in the interface that would also nicely show up in the app (you can see the screenshot in the feature image on top of this page).


## The solution {#the-solution-id}

I use the latest esphome docker image to be able to use the latest features instead installing via python pip,
```
docker pull esphome/esphome
docker run --rm -v "${PWD}":/config -it esphome/esphome run espcam-fridge-1.yaml
```

Follow the official home assistant documentation to add a esphome device to home assistant, https://www.home-assistant.io/integrations/esphome/

My Home Assistant configuration.yaml
```
image_processing:
  - platform: doods
    scan_interval: 100
    url: "http://<ip-to-doods>:<port-to-doods>"
    #detector: edgetpu
    detector: default
    file_out:
      - "/tmp/{{ camera_entity.split('.')[1] }}_latest.jpg"
      - "/camera-storage/fridge/{{ camera_entity.split('.')[1] }}_{{ now().strftime('%Y%m%d_%H%M%S') }}.jpg"
    source:
      - entity_id: camera.kitchen_m5cam_fridge
    confidence: 40
    labels:
      - refrigerator
```

To be able to always see the latest image captured in the home assistant (through the app)
```
camera:
  - platform: local_file
    name: doods_kitchen_m5cam_fridge
    file_path: /tmp/kitchen_m5cam_fridge_latest.jpg
```

# The directors cut {#timelapse-directors-cut-id}
Here are the images I removed from the top gif. It is not perfect as it will pull an image when it detects a fridge regardless of what is in the picture besides the fridge ;-)
{{< image src="fridge-fails.gif" title="Fridge fail timelaps" alt="fridge fails captures"  width="50%" height="50%" >}}

# Creds {#timelapse-id}
Creds goes to the home assistant, doods, esphome team and airijia.com for the esp32cam esphome example. The platform, and frameworks they have built is fantastic and I hope I could drive more users to focus more on a privacy first setup. A special high five to [OttoWinter](https://github.com/OttoWinter) for the [pull request](https://github.com/home-assistant/core/pull/56216) to esphome that added lightweight transport encryption to the api, now I am waiting for the same for mqtt ;-)
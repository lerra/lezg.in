---
title: "Privacy first fridge camera empowered with machine learning so I know what to get during grocery shopping"
date: 2021-12-19T16:29:01+08:00
lastmod: 2021-12-19T21:29:01+08:00
description: "The result of this privacy first smart home article will explain how to get the feature to see what is in your fridge while you are doing your grocery shopping with open source solution. Most important, with a high Wife Acceptance Factor."
resources:
- name: "featured-image"
  src: "featured-image.jpg"
  
  
page:
    theme: "wide"


categories: ["Documentation"]

lightgallery: true

toc:
  auto: true
 
draft: false
---

This privacy first smart home article will explain how to get the open source feature to see what is in your fridge while you are doing your grocery shopping. Most important, with a high Wife Acceptance Factor.
<!--more-->
# Timelapse of the result {#timelapse-id}
Here is a timelapse over two months on my fridge from the solution.
{{< figure src="fridge-snapshots.gif" title="Fridge timelaps" alt="fridge snapshots"  width="50%" height="50%" >}}
## Pre-requirements {#pre-requierments-id}

### Hardware {#hardware-id}
I used an M5Cam (an ESP32Cam hardware) as I already had one. To my surprise when I was using it for another project I bumped into heating issues and found out that it is a known issue when running it 24/7. So for me, it is a perfect use case as I get cooling out of the box ;-) If you have a powerful cpu (and not a Raspberry Pi) you would not need a Google Coral USB hardware to offload the image processing (TPU). If you have the need of to be 100% sure you get a snapshot every time you open the fridge, then I recommend to use the Google Coral USB hardware or a GPU to offload the image processing.

### Software {#software-id}
You need to setup [DOODS](https://github.com/snowzach/doods/), [ESPHome](https://www.esphome.io/), [Home Assistant](https://www.home-assistant.io/installation) and some remote/VPN capabilities to Home Assistant. My setup is built with privacy in mind, therefore I use solutions I have full control of so I use my own VPN. I would not recommend exposing your Home Assistant directly to the Internet, if you are not good at ensuring you always have the latest version due to security patches. If you don't want to build your own VPN I would recommend the cloud function of Home Assistant: [Nabucasa](https://www.nabucasa.com/). It is supposedly built with [privacy](https://www.nabucasa.com/privacy/) in mind and is the supporting company behind Home Assistant.

## The story {#the-story-id}
During the pandemic I had a lot more time, for obvious reasons. So I decided to build something with my M5Cam. Basically an ESP32 hardware with a camera. I managed to get ESPHome to work with it and easily get it connected to Home Assistant. I had an idea to monitor who does the dishes (and cooking) the most, but identifying who's hand it is above the dishes was a lot harder than I thought (please help me if you know how!) and I did run into heating issues when I was running it for 24/7. So I shifted focus to see if I could put it somewhere that would automatically cool it without me needing to add fans and such, so the fridge was the natural place to test.

I started to wire it and mounted the camera to the fridge door with the angle to capture the fridge when the door is open. It was fun to see the stream when I was opening and closing the fridge but I didn't really want to see a dark picture when I was grocery shopping. So step two was to ensure I always had a image of the latest opening of the fridge. I started to test with some switches and motion detection functionality to capture the picture when it is open but I did not get it to work that good as it was almost always blurry, the door was also not in the "perfect state" to capture the view of the fridge and I would block the view when I was taking grocery in or out. 

To solve that, I found that there is a tensorflow model that supports the object fridge so I configured [DOODS](https://github.com/snowzach/doods/) and tested to post an example image from the fridge when the door is open through curl and one of the objects detected was "fridge", I started to setup the [integration between home assistant and DOODS](https://www.home-assistant.io/integrations/doods/).

The last step was to show the latest captured image in Home Assistant. The DOODS integration have some options for the file_out, setting dynamic parameters to the filename. So I have one with the time/date in the filename and a second one that is named latest. With that I can use Home Assistant to emulate a camera by reading the local file and then show it in the interface that would also nicely show up in the app (you can see the screenshot in the feature image on top of this page).


## The solution {#the-solution-id}

I use the latest esphome docker image to be able to use the latest features of installing via "python pip", just make sure you have ESPHome version 2021.9. [Get the example esphome yaml file from my guthub](https://github.com/lerra/home-automation/blob/main/fridge-camera/espcam-fridge-1.yaml) for the esp32 cam, modify it with your needs.
```
docker pull esphome/esphome
docker run --rm -v "${PWD}":/config -it esphome/esphome run espcam-fridge-1.yaml
```

[Follow the official home assistant documentation to add a ESPHome device to Home Assistant](https://www.home-assistant.io/integrations/esphome/) and [download DOODS and follow the instructions to get it up and running](https://github.com/snowzach/doods/).

After that you can configure Home Assistant to use DOODS for processing the images from the ESP32 camera and store images when the object "fridge" is detected. Here is the relevant section from my Home Assistant configuration.yaml:
```
image_processing:
  - platform: doods
    scan_interval: 100
    url: "http://<ip-to-doods>:<port-to-doods>"
    #detector: edgetpu
    detector: default
    file_out:
      - "/tmp/{{ camera_entity.split('.')[1] }}\\
      _latest.jpg"
      - "/camera-storage/fridge/\\
        {{ camera_entity.split('.')[1] }}_\\
        {{ now().strftime('%Y%m%d_%H%M%S') }}.jpg"
    source:
      - entity_id: camera.kitchen_m5cam_fridge
    confidence: 40
    labels:
      - refrigerator
```

To be able to always see the latest image captured in the Home Assistant (through the app or web) you will need to emulate a camera from the local file
```
camera:
  - platform: local_file
    name: doods_kitchen_m5cam_fridge
    file_path: /tmp/kitchen_m5cam\\
    _fridge_latest.jpg
```

Don't forget to add the path to Home Assistant configuration.yaml so you don't get block from use it
```
homeassistant:
  allowlist_external_dirs:
    - /tmp
    - /camera-storage

```
If everything works you can now add the camera in the Home Assistant frontend [lovelace with the picture entity card](https://www.home-assistant.io/lovelace/picture-entity/).

# The directors cut {#timelapse-directors-cut-id}
Here are the images I removed from the top gif. It is not perfect as it will pull an image when it detects a fridge regardless of what is in the picture besides the fridge ;-)
{{< figure  src="fridge-fails.gif" title="Fridge fail timelaps" alt="fridge fails captures"  width="50%" height="50%" >}}

# Credits {#timelapse-id}
Credits goes to the Home Assistant-, DOODS-, ESPHome- team and airijia.com for the ESP32Cam ESPHome example. The platform, and frameworks they have built is fantastic and I hope I could drive more users to focus more on a privacy first setup. A special high five to [OttoWinter](https://github.com/OttoWinter) for the [pull request](https://github.com/home-assistant/core/pull/56216) to ESPHome that added lightweight transport encryption to the API, now I am waiting for the same for MQTT ;-) Btw, feel free to send a [pull request to the article](https://github.com/lerra/lezg.in/blob/master/content/posts/fridge-cam-with-home-assistant-and-machine-learning/index.en.md) or reach out on twitter if you find any errors.
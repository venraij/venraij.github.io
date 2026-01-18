---
title: Setting up an AI assistant using Home Assistant and Ollama
lead: Setting up an AI assistant isn't as hard as you might think. Although you might hit some roadblocks and you need the right hardware, it's definitely doable.
published: 2026-01-18
tags: [Home Assistant, AI, Ollama]
authors:
    - name: "Nick van Raaij"
      gitHubUserName: "VenRaij"
---

### Smartification
I've been in smartification mood recently and this has resulted in something pretty cool in my opinion. 
I found out that it's quite easy to setup an AI assistant at home using Home Assistant and Ollama. As long as you have
the proper requirements. And I did hit a couple of roadblocks, so I'll share my experience here. The final goal is to be
able to trigger the assistant using "Hey MurderBot" and confuse the hell out of my guests.

![ha assistant](media/ha-assistant.png)

## Step 1: What do I need to get started?
I already had the basic requirements to get started. I've got Home Assistant running on a Raspberry Pi 5 and I have a 
home server with a NVIDIA GPU GTX 970 (any decently modern dedicated or integrated GPU should work though) that can run Ollama. 
A CPU could also be used technically, but I would probably die of old age before I ever find out what the weather will be today.
If you don't have these things yet, you need to get them first. It would be handy if the server has docker installed, 
you could do this without docker but unless you've got some kind of fetish for pain and suffering I wouldn't recommend it.

## Step 2: What am I even trying to achieve?
So at some point I realized that I needed to figure out what I wanted to do with this setup. Of course then I thought, 
"Who cares?! Just do it because it's fun!". Well my experience has taught me it's better to have a plan. 
So I decided that I wanted to be able to dot the following things:
- Ask the assistant questions like "What's the weather like today?" or "Is the heating on?"
- Control smart devices like "Turn on the living room lights" or "Set the thermostat to 22 degrees".
- Ask questions about locational data like "Who is at home?".
- Be able to talk instead of typing.
- The assistant should be able to talk as well.

Eventually I'd like to be able to trigger the AI assistant using wake words, but I haven't gotten that far yet. 
It would also be cool if it could greet me when I come home, but again, haven't gotten that far yet.

## Step 3: Setting up Ollama
Setting up Ollama with docker is quite easy, it doesn't require a lot of configuration and is basically ready to got.
I used docker compose to set it up with the following configuration:

```
services:
  ollama:
    image: ollama/ollama
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: 1
              capabilities: [gpu]
    container_name: ollama
    volumes:
      - /var/lib/ollama:/root/.ollama
    ports:
      - 11434:11434
```
I've set it up for an NVIDIA GPU as you can see in the "deploy" property. This part differs each kind of gpu you'd like 
to use. The deploy part can be left out when using the CPU for running the model.

When running ``docker compose up`` this results in a running Ollama server on port 11434. This server doesn't have any 
model setup yet but that turned out not to be an issue..

## Step 4: Setting up Whisper in Home Assistant



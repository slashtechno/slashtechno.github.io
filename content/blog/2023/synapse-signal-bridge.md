---
title: Simple Signal-Matrix Bridge in Docker
date: 2023-12-18
# summary: 
draft: true
# tags: ["signal", "matrix"]
description: Setting up a (simple) bridge between Matrix and Signal
---  
In September of 2023, I wrote a [post](/blog/2023/setting-up-synapse/) regarding how to set up a simple Matrix server. Personally, I think that Matrix is a cool protocol, but I don't use it much (at the time of writing). However, I love the ability to bridge chat services. Beeper, the company which has recently been in the news for developing a native Android iMessage client, originally started out bridging chat services. Whilst their service itself is proprietary (at the time of writing), the bridges they use are open source. They have a [guide](https://github.com/beeper/self-host) on how to self-host various bridges and Matrix with Ansible. I have not gotten around to learning Ansible, so I manually set up the Docker containers.  

## Setup
I have a Synapse server running in Docker on port 8007 locally. Thus, the following instructions assume that Synapse is running on port 8007. If you're running Synapse on a different port, change the port in the following commands.
In addition, I will be using `matrix.example.com` as the server hostname.  
1. Create a new directory for the bridge.  
    ```
    mkdir signal-matrix-bridge
    cd signal-matrix-bridge
    ```  
2. Use your preferred text editor to create a file named `docker-compose.yml` with the following contents. Most of this configuration is taken from the [Mautrix guide](https://docs.mau.fi/bridges/general/docker-setup.html?search=&bridge=signal#docker-compose) and as detailed, ideally, the bridge and Synapse should be on the same network. As this is just a simple server that I'm not using for mission-crucial purposes, I'm not too concerned about this. I've left the comments in the file for reference.
    ```
    version: "3.7"

    services:
    mautrix-signal:
        container_name: mautrix-signal
        image: dock.mau.dev/mautrix/signal:latest
        restart: unless-stopped
        volumes:
            - .:/data
        # If you put the service above in the same docker-compose as the homeserver,
        # ignore the parts below. Otherwise, see below for configuring networking.

        # If synapse is running outside of docker, you'll need to expose the port.
        # Note that in most cases you should either run everything inside docker
        # or everything outside docker, rather than mixing docker things with
        # non-docker things.
        ports:
            - "29328:29328"
        # You'll also probably want this so the bridge can reach Synapse directly
        # using something like `http://host.docker.internal:8008` as the address:
        #extra_hosts:
        #- "host.docker.internal:host-gateway"

        # If synapse is in a different network, then add this container to that network.
        #networks:
        #- synapsenet
    # This is also a part of the networks thing above
    #networks:
    #  synapsenet:
    #    external:
    #      name: synapsenet
    ```
3. 
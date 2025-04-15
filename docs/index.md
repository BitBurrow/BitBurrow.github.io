---
title: BitBurrow overview
---

## NOTE: THIS SOFTWARE DOES NOT EXIST YET

*Everything below is proposed draft documentation. The software to do what is described is being developed and is not at all usable yet.*

## Introduction

BitBurrow is a set of tools to help you set up and use a VPN server anywhere--at your parents' house, an office, or a friend's apartment. And you don't have to be good with computers. A BitBurrow VPN server will allow you to securely use the internet from anywhere in the world as if you were at your "VPN home".

If you just want to set up your own BitBurrow VPN server as described above, skip the rest of this page and go to [How to set up a BitBurrow base](base.md).

## Technical overview

BitBurrow includes four main components:

* BitBurrow **base** is a router at your VPN home that functions as a VPN server. It accepts VPN connections from user devices and forwards them to the internet. See [How to set up a BitBurrow base](base.md).
* BitBurrow **app** runs on Android or iOS and, together with the hub, is used to configure a BitBurrow base.
* BitBurrow **hub** runs on a server on the public internet. It serves to configure a BitBurrow base and also DNS requests from user devices. See [How to set up a BitBurrow hub](hub.md).
* BitBurrow **outpost** is a router configured as a VPN client, allowing users at a secondary location to use the internet as if they were at the "VPN home" without any additional software.

Here is a visual representation.

``` mermaid
%% live-view editor: https://mermaid.live/
%% theme docs: https://mermaid.js.org/config/theming.html

%% FIXME: when Material for MkDocs updates to Mermaid 10.1:
%% * add Markdown titles, e.g.: base["`/fa:fa-network-wired BitBurrow **base**`"\]
%% * add invisible links to attempt to order subgraphs, e.g.:: secondHome ~~~ vpnHome

flowchart BT
    %% flowchart docs: https://mermaid.js.org/syntax/flowchart.html
    %% available icons: https://fontawesome.com/v5/search?o=r&m=free
    subgraph secondHome["<b>second home</b>"]
        outpost[/"<img src='fa/network-wired.svg' width=16 height=16/> BitBurrow <b>outpost</b>"\]
    end
    subgraph coffeeShop["<b>coffee shop</b>"]
        user1("<img src='fa/laptop.svg' width=16 height=16/> user device")
        user2("<img src='fa/mobile-screen-button.svg' width=16 height=16/> user device")
    end
    subgraph vpnHome["<b>VPN home</b>"]
        base[/"<img src='fa/network-wired.svg' width=16 height=16/> BitBurrow <b>base</b>"\]
        %%click base href "https://www.github.com" "BitBurrow base is ..." _blank
        app("<img src='fa/mobile-screen-button.svg' width=16 height=16/> BitBurrow <b>app</b>")
    end
    subgraph internet["<b>internet</b>"]
        hub["<img src='fa/server.svg' width=16 height=16/> BitBurrow <b>hub</b>"]
        resolver(<img src='fa/server.svg' width=16 height=16/> DNS resolver)
        website["<img src='fa/server.svg' width=16 height=16/> website"]
    end
    subgraph key["<b>key</b>"]
        direction LR
        A("<img src='fa/laptop.svg' width=16 height=16/>")
        B("<img src='fa/server.svg' width=16 height=16/>")
        C("<img src='fa/laptop.svg' width=16 height=16/>")
        D("<img src='fa/server.svg' width=16 height=16/>")
        E("<img src='fa/laptop.svg' width=16 height=16/>")
        F("<img src='fa/server.svg' width=16 height=16/>")
        G("<img src='fa/laptop.svg' width=16 height=16/>")
        H("<img src='fa/server.svg' width=16 height=16/>")
    end
    %% http or https
    A -->|"<span style='background-color:#ededed'>http/https</span>"| B
    base ---> website
    base --> hub
    app <--> hub
    linkStyle 0,1,2,3 stroke:purple
    %% ssh or https
    C -->|"<span style='background-color:#ededed'>ssh</span>"| D
    app --> base
    linkStyle 4,5 stroke:red
    %% WireGuard connections (add '=' for longer connections)
    E ==>|"<span style='background-color:#ededed'>WireGuard</span>"| F
    outpost ===> base
    user1 ===> base
    user2 ===> base
    base ===> hub
    %% DNS connections (add '.' for longer connections)
    G -.->|"<span style='background-color:#ededed'>vxm.example.org DNS</span>"| H
    outpost -..-> resolver
    user1 -..-> resolver
    user2 -..-> resolver
    resolver -.-> hub
    %% styles
    %%linkStyle default stroke:green
    classDef highlight fill:#fff411,stroke-width:4px
    %%class base highlight
```

## Links

* [Source code for the BitBurrow hub](https://github.com/BitBurrow/BitBurrow)
* [Source code for the BitBurrow app](https://github.com/BitBurrow/BitBurrow/tree/main/app)
* [Source code for *this* website](https://github.com/BitBurrow/BitBurrow.github.io)
* [BitBurrow website](https://bitburrow.com)


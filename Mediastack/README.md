# Media Stack
This docker compose contains a media stack consisting of several services for self-hosted media/streaming/etc.

## Services
- Gluetun for (selective) routing of traffic through a VPN tunnel
- qBittorrent for Torrent exchange (⚠ I do not endorse piracy or other illegal activities)
- qBittorrent Port Manager to enable automatic updates to the inbount port given by the VPN provider
- Jellyfin as the Media Server
- Radarr for movies 🎥
- Sonarr for shows 📺
- Bazarr for subtitles
- Prowlarr for indexing and searching

## Notes on my setup
I run docker in a VM on my Proxmox server. All media files as well as (most) configuration is stored on my NAS (TrueNAS Scale) and provided to my docker host using mounted shares. 
If you do this in a similar way, please be aware that this might be a bit tricky in terms of permissions and ownership. This is also the reason some of my service data is not stored on shares (e.g. because of databases which cannot handle this).

In my case, that is not an issue as my VMs are also backed up daily to my NAS and weekly off-site. But make sure that you have a solid backup strategy in place in case shit hits the fan.

## Underlying filesystem structure
It is advices to have the docker volumes/mounts refer to the same root such that e.g. hardlinks can function and unnecessary copying of files is avoided.
My setup assumes that the data directory **`FOLDER_FOR_MEDIA`** has the following structure:

``` { .text .no-copy }
    ├── /FOLDER_FOR_MEDIA   ⠀
    ⠀⠀⠀⠀⠀├── backup			← This will be configured as the backup location within the arr-services
    ⠀⠀⠀⠀⠀├── media			← Contains the media files
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── anime             
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── movies      
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── music	← Not used by any services, but you could use it for Lidarr 
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── pictures	← Not used by any services   
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀└── tv                
    ⠀⠀⠀⠀⠀├── downloads		← Used by qBittorrent  
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── anime             
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── complete       
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── incomplete  
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── movies             
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀├── prowlarr       
    ⠀⠀⠀⠀⠀│⠀⠀⠀⠀└── tv                        
    ⠀⠀⠀⠀⠀└── watch			← Folder to put manual .torrent files in for download           
```

## VPN Port Forwarding
If you want to make sure that you actually share torrents, it is required to have port forwarding to qBittorrent. That also means your VPN provider needs to support this, which is why I use ProtonVPN. 
Unfortunately, this gives you a different port whenever the connection is established which you would need to update manually then.

Luckily, there is the [qBittorrent Gluetun port update]([https://github.com/SnoringDragon/gluetun-qbittorrent-port-manager](https://github.com/joshua-klassen/qbittorrent-gluetun-port-update)) which automates this for you. It will do this via the qBittorrent WebUI, so the credentials for that are provided to it.

I set the stack up in a way that this service waits for qBittorrent to be up and running before updating the port. This way, it will always work properly in my experience.

## Arr configuration
Once the stack is up and running, some configuration of the arr-services is required. I won't write that down here because there is a pretty good video on the basics [here](https://www.youtube.com/watch?v=3k_MwE0Z3CE).


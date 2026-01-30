# Media Stack Services

## Plex
Media server for streaming movies, TV shows, and music to any device.  
Access: [10.0.10.203:32400/web](http://10.0.10.203:32400/web)

## Gluetun
VPN client that routes traffic through NordVPN for privacy and geo-unblocking.  
Access: No web interface (background service)

## qBittorrent
BitTorrent client for downloading torrents through the VPN connection.  
Access: [10.0.10.203:8080](http://10.0.10.203:8080)

## SABnzbd
Usenet downloader for downloading from newsgroups through the VPN connection.  
Access: [10.0.10.203:8081](http://10.0.10.203:8081)

## Sonarr
TV show collection manager that automatically downloads and organizes episodes.  
Access: [10.0.10.203:8989](http://10.0.10.203:8989)

## Radarr
Movie collection manager that automatically downloads and organizes films.  
Access: [10.0.10.203:7878](http://10.0.10.203:7878)

## Watchtower
Automatically updates Docker containers when new versions are available.  
Access: No web interface (background service)

## Prowlarr
Indexer manager that handles torrent and usenet search providers for Sonarr/Radarr.  
Access: [10.0.10.203:9696](http://10.0.10.203:9696)

## Overseerr
Request management system for users to request movies and TV shows.  
Access: [10.0.10.203:5055](http://10.0.10.203:5055)

## Bookshelf
Digital book library manager for organizing and reading ebooks.  
Access: [10.0.10.203:8083](http://10.0.10.203:8083)

## Bazarr
Subtitle management tool that automatically downloads subtitles for movies and TV shows.  
Access: [10.0.10.203:6767](http://10.0.10.203:6767)
# Media Stack Services

## Plex
Media server for streaming movies, TV shows, and music to any device.  
Access: `<HOST_IP_PLACEHOLDER>:32400/web`

## Gluetun
VPN client that routes traffic through NordVPN for privacy and geo-unblocking.  
Access: No web interface (background service)

## qBittorrent
BitTorrent client for downloading torrents through the VPN connection.  
Access: `<HOST_IP_PLACEHOLDER>:8080`

## Sonarr
TV show collection manager that automatically downloads and organizes episodes.  
Access: `<HOST_IP_PLACEHOLDER>:8989`

## Radarr
Movie collection manager that automatically downloads and organizes films.  
Access: `<HOST_IP_PLACEHOLDER>:7878`

## Watchtower
Automatically updates Docker containers when new versions are available.  
Access: No web interface (background service)

## Prowlarr
Indexer manager that handles torrent and usenet search providers for Sonarr/Radarr.  
Access: `<HOST_IP_PLACEHOLDER>:9696`

## Overseerr
Request management system for users to request movies and TV shows.  
Access: `<HOST_IP_PLACEHOLDER>:5055`

## Bookshelf
Digital book library manager for organizing and reading ebooks.  
Access: `<HOST_IP_PLACEHOLDER>:8083`

## Bazarr
Subtitle management tool that automatically downloads subtitles for movies and TV shows.  
Access: `<HOST_IP_PLACEHOLDER>:6767`
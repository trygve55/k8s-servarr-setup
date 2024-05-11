# helm-charts

```shell
helm repo add k8s-at-home https://k8s-at-home.com/charts/
helm install qbittorrent k8s-at-home/qbittorrent --version 13.4.2
```

```shell
sudo mkdir /mount/hdd0/data
sudo chown trygve:trygve /mount/hdd0/data
mkdir -p /mount/hdd0/data/media/Movies
mkdir /mount/hdd0/data/media/Tv-Series
mkdir /mount/hdd0/data/media/Music
mkdir /mount/hdd0/data/downloads

mkdir -p /home/trygve/k8s-data/config/qbittorrent
helm install qbittorrent arch-qbittorrentvpn/ -n media --create-namespace

mkdir -p /home/trygve/k8s-data/config/sonarr
helm install sonarr sonarr -n media

mkdir -p /home/trygve/k8s-data/config/prowlarr
helm install prowlarr prowlarr -n media

mkdir -p /home/trygve/k8s-data/config/jellyfin
helm install jellyfin jellyfin -n media
```

### Sonarr setup:
Add download client: 
url: qbittorrent-arch-qbittorrentvpn.media.svc

Settings -> Media Management -> Root Folders
Add /data/media/Tv-Series/

### Prowlarr setup
Add indexers

Settings -> Apps
Add sonarr:
Prowlarr Server: http://prowlarr:9696
Sonarr Server: http://sonarr:8989
API Key: Copy API key from Sonarr interface

### Jellyfin

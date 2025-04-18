# helm-charts
```shell
kubectl create namespace media
```

### Add common authentication
This will enable us to login to all of our services once with the same username and password.
```shell
sudo apt install apache2-utils  
htpasswd -c ./basic-auth <your-admin-username>

kubectl create secret generic basic-auth --from-file=auth=basic-auth --namespace media
```

```shell
sudo mkdir -p /mount/hdd0/data
sudo chown trygve:trygve /mount/hdd0/data
mkdir -p /mount/hdd0/data/media/Movies
mkdir /mount/hdd0/data/media/Tv-Series
mkdir /mount/hdd0/data/media/Music
mkdir /mount/hdd0/data/Downloads

mkdir -p /home/trygve/k8s-data/config/qbittorrent
helm install qbittorrent trygve55/arch-qbittorrentvpn/ \
  --namespace media \
  --set baseUrl="/" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="qbittorrent.local"

mkdir -p /home/trygve/k8s-data/config/sonarr
helm install sonarr trygve55/sonarr -n media \
  --namespace media \
  --set baseUrl="/" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="sonarr.local" \
  --values ingress-basic-auth-values.yaml

mkdir -p /home/trygve/k8s-data/config/radarr
helm install radarr trygve55/radarr -n media \
  --namespace media \
  --set baseUrl="/" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="radarr.local" \
  --values ingress-basic-auth-values.yaml

helm repo add k8s-home-lab https://k8s-home-lab.github.io/helm-charts/
helm repo update
helm install flaresolverr k8s-home-lab/flaresolverr -n media

mkdir -p /home/trygve/k8s-data/config/prowlarr
helm install prowlarr trygve55/prowlarr -n media \
  --namespace media \
  --set baseUrl="/" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="prowlarr.local" \
  --values ingress-basic-auth-values.yaml

mkdir -p /home/trygve/k8s-data/config/bazarr
helm install bazarr trygve55/bazarr -n media \
  --namespace media \
  --set baseUrl="/" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="bazarr.local" \
  --values ingress-basic-auth-values.yaml

mkdir -p /home/trygve/k8s-data/config/jellyfin
helm install jellyfin trygve55/jellyfin -n media \
  --namespace media \
  --set baseUrl="/" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="jellyfin.local"

mkdir -p /home/trygve/k8s-data/config/jellyseerr
helm install jellyseerr trygve55/jellyseerr -n media \
  --namespace media \
  --set baseUrl="/" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="jellyseerr.local"

helm repo add byjg https://opensource.byjg.com/helm
helm repo update byjg    
helm upgrade --install staticpage byjg/static-httpserver \
  --namespace media \
  --values static-page-values.yaml \
  --values ingress-basic-auth-values.yaml
```

### Sonarr setup:
Settings -> Download Clients <br>
Add download client: <br>
Name: `qBittorent` <br>
Host: `qbittorrent-arch-qbittorrentvpn` <br>

Settings -> Media Management -> Episode Naming <br>
Series Folder Format: `{Series TitleYear}` <br>
Season Folder Format: `{Series TitleYear} Season {season}` <br>

Settings -> Media Management -> Root Folders <br>
Add `/data/media/Tv-Series/`

### Radarr setup:
Settings -> Download Clients <br>
Add download client: <br>
Name: `qBittorent` <br>
Host: `qbittorrent-arch-qbittorrentvpn`

Settings -> Media Management -> Root Folders <br>
Add `/data/media/Movies/`

### Prowlarr setup
Add FlareSolverr <br>
Settings -> Indexer Proxies -> Add <br>
Name: `FlareSolverr` <br>
Tags: `flare` <br>
Host: `http://flaresolverr:8191/`

Add indexers <br>
Some indexers will indicate that they need FlareSolverr, add the `flare` tag to those indexers.

Settings -> Apps
Add sonarr: <br>
Prowlarr Server: `http://prowlarr:9696` <br>
Sonarr Server: `http://sonarr:8989` <br>
API Key: Copy API key from Sonarr interface

Add radarr: <br>
Prowlarr Server: `http://prowlarr:9696` <br>
Radarr Server: `http://radarr:7878` <br>
API Key: Copy API key from Radarr interface

### Bazarr setup:
Settings -> Languages -> Subtitles Language <br>
Languages Filter: Add your preferred languages

Settings -> Languages -> Languages Profile <br>
Add New Profile <br>
Name: `default` <br>
Add your languages <br>
Save

Settings -> Languages -> Default Language Profiles For Newly Added Shows <br>
Enable for Series and Movies. <br>
Select the profile `default` for both

Settings -> Providers <br>
Add your subtitle providers.

Settings -> Sonarr <br>
Address: `sonarr` <br>
API Key: Copy API key from Sonarr interface

Settings -> Radarr <br>
Address: `radarr` <br>
API Key: Copy API key from Radarr interface

### Jellyfin
Create admin user.

Add Movies, Tv-series and Music folder.

### Jellyseerr
Url: http://media.local/jellyseerr/setup <br>

#### Choose server type:
Choose Jellyfin as server type.

#### Sign In
Jellyfin URL: `jellyfin` <br>
URL Base: `/jellyfin` <br>
Sign in with the username and password created for Jellyfin. <br>

#### Configure Media Server
Press the `Sync Libraries` button and enable your Jellyfin libraries.
Press the `Start Scan` button.
External URL: `http://media.local/jellyfin` <br>
Press the `Save Changes` button.

#### Configure services:
###### Add Radarr <br>
Default Server: `Yes` <br>
Server Name: `radarr` <br>
Hostname or IP Address: `radarr` <br>
API Key: Copy API key from Radarr interface <br>
URL Base: `/radarr` <br>
Quality profile: Select your preferred quality <br>
Root folder: `/data/media/Movies` <br>
External URL: `http://media.local/radarr` <br>
Enable scan: `yes` <br>

###### Add Sonarr <br>
Default Server: `Yes` <br>
Server Name: `sonarr` <br>
Hostname or IP Address: `sonarr` <br>
API Key: Copy API key from Sonarr interface <br>
URL Base: `/sonarr` <br>
Quality profile: Select your preferred quality <br>
Root folder: `/data/media/Tv-Series` <br>
Season Folders: `yes` <br>
External URL: `http://media.local/sonarr` <br>
Enable scan: `yes` <br>

Finish setup!
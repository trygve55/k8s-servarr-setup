# k8s-servarr-setup
This guide will help you set up your own Servarr setup using Kubernetes.
## Requirements
* A Kubernetes cluster (you can follow [this setup](https://github.com/trygve55/kubernets-cluster-setup))
* An ingress controller
* (only for HTTPS)
  * cert-manager
  * Port forwarded port 80 and 443.


## Setup 
#### Add common authentication
This will enable us to login to all of our services once with the same username and password. Replace `<your-admin-username>` with your desired username and use a strong password.
```shell
sudo apt install apache2-utils  
htpasswd -c ./basic-auth <your-admin-username>

kubectl create secret generic basic-auth --from-file=auth=basic-auth --namespace media
```
#### Notes
Remove `--set clusterIssuer="letsencrypt-prod"` from the installation command if you do not wish to enable HTTPS.

Remove `--values ingress-basic-auth-values.yaml` if you do not want to require authentication.

Replace `--set ingress.hosts[0].host="<hostname>"` with the hostname you wish to use.

You can set a baseUrl for each service using `--set baseUrl="/<your base url>"`.

#### Creating directories
Lets create the directories where our media would be stored. Replace `/mount/hdd0` with where you want to store your media.
```shell
sudo mkdir -p /mount/hdd0/data
sudo chown trygve:trygve /mount/hdd0/data
mkdir -p /mount/hdd0/data/media/Movies
mkdir /mount/hdd0/data/media/Tv-Series
mkdir /mount/hdd0/data/media/Music
```
#### Creating namespace
```shell
kubectl create namespace media
```
#### Add helm-charts repo
```shell
helm repo add trygve55 https://trygve55.github.io/helm-charts --force-update
```

### qBittorrent
```shell
mkdir /mount/hdd0/data/Downloads
mkdir -p /home/trygve/k8s-data/config/qbittorrent
helm install qbittorrent trygve55/arch-qbittorrentvpn \
  --namespace media \
  --set baseUrl="/" \
  --set clusterIssuer="letsencrypt-prod" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="qbittorrent.local" \
  --values ingress-basic-auth-values.yaml
```

#### VPN (OpenVPN)
First locate and download the OpenVPN configurations files (*.ovpn) from your VPN provider.

Create a secret containing your VPN credentials and configuration.
```shell
kubectl create secret generic openvpn-credentials \
    --namespace media \
    --from-literal=username="<your vpn username>" \
    --from-literal=password="<your vpn password>" \
    --from-file=openvpn-config.ovpn="<your *.ovpn configuration file>"
```
Add the following to the qbittorrent helm install command:
```shell
  --set vpn.enabled=true \
  --set vpn.protocol=openvpn \
  --set vpn.secret=openvpn-credentials \
```
### Sonarr
```shell
mkdir -p /home/trygve/k8s-data/config/sonarr
helm install sonarr trygve55/sonarr \
  --namespace media \
  --set baseUrl="/" \
  --set clusterIssuer="letsencrypt-prod" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="sonarr.local" \
  --set clusterIssuer="letsencrypt-prod" \
  --values ingress-basic-auth-values.yaml
```
#### Sonarr setup:
Settings -> Download Clients <br>
Add download client: <br>
Name: `qBittorent` <br>
Host: `qbittorrent-arch-qbittorrentvpn` <br>

Settings -> Media Management -> Episode Naming <br>
Series Folder Format: `{Series TitleYear}` <br>
Season Folder Format: `{Series TitleYear} Season {season}` <br>

Settings -> Media Management -> Root Folders <br>
Add `/data/media/Tv-Series/`

### Radarr
```shell
mkdir -p /home/trygve/k8s-data/config/radarr
helm install radarr trygve55/radarr \
  --namespace media \
  --set baseUrl="/" \
  --set clusterIssuer="letsencrypt-prod" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="radarr.local" \
  --values ingress-basic-auth-values.yaml
```

#### Radarr setup:
Settings -> Download Clients <br>
Add download client: <br>
Name: `qBittorent` <br>
Host: `qbittorrent-arch-qbittorrentvpn`

Settings -> Media Management -> Root Folders <br>
Add `/data/media/Movies/`

### Mylar3
```shell
mkdir -p /mount/hdd0/data/media/Books/Comics
mkdir -p /home/trygve/k8s-data/config/mylar3
helm install mylar3 trygve55/mylar3 \
--namespace media \
--set baseUrl="/" \
--set ingress.enabled=true \
--set ingress.hosts[0].host="mylar3.local" \
--values ingress-basic-auth-values.yaml
```

#### Mylar3 setup
Create a user at https://comicvine.gamespot.com/ and login. <br>
Go to https://comicvine.gamespot.com/api/ and copy your Comic Vine API key.

Settings -> Web Interface <br>
ComicVine API Key: `The value from above` <br>
Comic Location Path: `/data/media/Books/Comics` <br>
Mylar API key: Press Generate <br>
Save settings

Settings -> Download settings -> Torrents <br>
Use Torrents: Yes <br>
Select qBittorrent <br>
qBittorrent Host:Port: `http://qbittorrent-arch-qbittorrentvpn:8080` <br>
qBittorrent Label: `mylar3` <br>
qBittorrent Folder: `/data/downloads`

Settings -> Search providers <br>
Torrents: enabled <br>
Enable Torznab: enabled

### Readarr
```shell
mkdir -p /mount/hdd0/data/media/Books/Books
mkdir -p /home/trygve/k8s-data/config/readarr
helm install readarr trygve55/readarr \
--namespace media \
--set baseUrl="/" \
--set ingress.enabled=true \
--set ingress.hosts[0].host="readarr.local" \
--values ingress-basic-auth-values.yaml
```

#### Readarr setup
Settings -> Download Clients <br>
Add download client: <br>
Name: `qBittorent` <br>
Host: `qbittorrent-arch-qbittorrentvpn`

Settings -> Media Management -> Root Folders <br>
Add `/data/media/Books/Books`

#### FlareSolverr
Flaresolverr is only required for specific indexers.
```shell
helm repo add k8s-home-lab https://k8s-home-lab.github.io/helm-charts/ --force-update
helm install flaresolverr k8s-home-lab/flaresolverr -n media
```
#### Prowlarr
```shell
mkdir -p /home/trygve/k8s-data/config/prowlarr
helm install prowlarr trygve55/prowlarr \
  --namespace media \
  --set baseUrl="/" \
  --set clusterIssuer="letsencrypt-prod" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="prowlarr.local" \
  --values ingress-basic-auth-values.yaml
```

#### Prowlarr setup
Add FlareSolverr <br>
Settings -> Indexer Proxies -> Add <br>
Name: `FlareSolverr` <br>
Tags: `flare` <br>
Host: `http://flaresolverr:8191/`

Add indexers <br>
Some indexers will indicate that they need FlareSolverr, add the `flare` tag to those indexers.

Settings -> Apps
Add Sonarr: <br>
Prowlarr Server: `http://prowlarr:9696` <br>
Sonarr Server: `http://sonarr:8989` <br>
API Key: Copy API key from Sonarr interface

Add Radarr: <br>
Prowlarr Server: `http://prowlarr:9696` <br>
Radarr Server: `http://radarr:7878` <br>
API Key: Copy API key from Radarr interface

Add Readarr: <br>
Prowlarr Server: `http://prowlarr:9696` <br>
Readarr Server: `http://readarr:8787` <br>
API Key: Copy API key from Readarr interface

Add Mylar3: <br>
Prowlarr Server: `http://prowlarr:9696` <br>
Mylar3 Server: `http://mylar3:8090` <br>
API Key: Copy API key from Readarr interface

### Bazarr
```shell
mkdir -p /home/trygve/k8s-data/config/bazarr
helm install bazarr trygve55/bazarr \
  --namespace media \
  --set baseUrl="/" \
  --set clusterIssuer="letsencrypt-prod" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="bazarr.local" \
  --values ingress-basic-auth-values.yaml
```

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
```shell
mkdir -p /home/trygve/k8s-data/config/jellyfin
helm install jellyfin trygve55/jellyfin -n media \
  --namespace media \
  --set baseUrl="/" \
  --set clusterIssuer="letsencrypt-prod" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="jellyfin.local"
```

#### Hardware transcoding
If you have an Intel GPU or a iGPU you can enable hardware transcoding to speed up the process. Add the following to your command to enable this.

Make sure at least one renderD* device exists in /dev/dri. Otherwise upgrade your kernel or enable the iGPU in the BIOS.
```shell
ls -l /dev/dri
```

Then we need to query the id of the render group on the host system:
```shell  
getent group render | cut -d: -f3
```
Replace `105` with the id returned from the previous command.
```shell
--set hardwareAcceleration.intel.enabled=true \ 
--set hardwareAcceleration.intel.renderGroupId=105 \
```
You also need to enable this in the Jellyfin settings after the setup is done. <br>
https://jellyfin.org/docs/general/post-install/transcoding/hardware-acceleration/intel

##### Multi GPU systems
If you have multiple GPUs you need to select the correct rendering device. Replace `renderD128` with your rendering device.
```shell
--set hardwareAcceleration.intel.renderDevicePath="/dev/dri/renderD128" \
```

#### Jellyfin setup
Create admin user.

Add Movies, Tv-series and Music folder.
### Jellyseerr
```shell
mkdir -p /home/trygve/k8s-data/config/jellyseerr
helm install jellyseerr trygve55/jellyseerr -n media \
  --namespace media \
  --set baseUrl="/" \
  --set clusterIssuer="letsencrypt-prod" \
  --set ingress.enabled=true \
  --set ingress.hosts[0].host="jellyseerr.local"
```

#### Jellyseerr setup
##### Choose server type:
Choose Jellyfin as server type.

##### Sign In
Jellyfin URL: `jellyfin` <br>
URL Base: `/` <br>
Sign in with the username and password created for Jellyfin. <br>

##### Configure Media Server
Press the `Sync Libraries` button and enable your Jellyfin libraries.
Press the `Start Scan` button.
External URL: `http://jellyfin.local` <br>
Press the `Save Changes` button.

##### Configure services:
###### Add Radarr <br>
Default Server: `Yes` <br>
Server Name: `radarr` <br>
Hostname or IP Address: `radarr` <br>
API Key: Copy API key from Radarr interface <br>
URL Base: `/` <br>
Quality profile: Select your preferred quality <br>
Root folder: `/data/media/Movies` <br>
External URL: `http://radarr.local` <br>
Enable scan: `yes` <br>

###### Add Sonarr <br>
Default Server: `Yes` <br>
Server Name: `sonarr` <br>
Hostname or IP Address: `sonarr` <br>
API Key: Copy API key from Sonarr interface <br>
URL Base: `/` <br>
Quality profile: Select your preferred quality <br>
Root folder: `/data/media/Tv-Series` <br>
Season Folders: `yes` <br>
External URL: `http://sonarr.local` <br>
Enable scan: `yes` <br>

#### Overview page
```shell
helm repo add byjg https://opensource.byjg.com/helm
helm repo update byjg    
helm upgrade --install staticpage byjg/static-httpserver \
  --namespace media \
  --values static-page-values.yaml \
  --values ingress-basic-auth-values.yaml
```

Finish setup!

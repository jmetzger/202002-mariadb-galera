# Installation 

## Repo besorgen und installieren

```
# https://downloads.mariadb.org/mariadb/repositories/#distro=Ubuntu&distro_release=focal--ubuntu_focal&mirror=23Media&version=10.4
sudo apt-get install software-properties-common
sudo apt-key adv --fetch-keys 'https://mariadb.org/mariadb_release_signing_key.asc'
sudo add-apt-repository 'deb [arch=amd64,arm64,ppc64el] http://mirror.23media.de/mariadb/repo/10.4/ubuntu focal main'

sudo apt install mariadb-server 
hostnamectl set-hostname galera1.training.local 
```

## Im  Training 

  * Maschinen 2x in virtualbox clonen 

## Achtung machine_id löschen 

  * Sonst erfolgt die gleiche IP auf allen maschinen 

```
sudo rm -f /etc/machine-id 
sudo systemd-machine-id-setup
sudo hostnamectl set-hostname galera2.training.local
sudo reboot 
```

  

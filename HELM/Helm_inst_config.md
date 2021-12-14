# Installing Helm
Helm can be installed from
1. **From binary release**
2. **From script**
3. **Throguh package managers**

## From binary release
1. Download your [desired version](https://github.com/helm/helm/releases)
2. Unpack it
	```
	tar -zxvf helm-v3.0.0-linux-amd64.tar.gz
	```
3. Find the helm binary in the unpacked directory, and move it to its desired destination.
	```sh
	mv linux-amd64/helm /usr/local/bin/helm
	```
## From script
```sh
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
```
## Throguh package managers
> These are not supported by the Helm project and are not considered trusted 3rd parties.
### Debian/Ubuntu
```sh
curl https://baltocdn.com/helm/signing.asc | sudo apt-key add -
sudo apt-get install apt-transport-https --yes
echo "deb https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```

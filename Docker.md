## Docker
### WEB-L WEB-R
add docker.iso
```
apt install docker*
systemctl enable docker
mkdir /mnt/docker
mount /dev/sr0 /mnt/docker
docker image load -i /mnt/docker/app.tar
docker run -d -p 80:80 --restart=always app:latest
```

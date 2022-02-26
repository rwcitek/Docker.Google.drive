# Google Drive mount in Docker


https://www.linuxbabe.com/cloud-storage/install-google-drive-ocamlfuse-ubuntu-linux-mint

https://hub.docker.com/r/maltokyo/docker-google-drive-ocamlfuse

https://github.com/Capgemini-AIE/ethereum-docker



## Create Image
```bash
docker run --rm \
  --name gdrive \
  ubuntu:20.04 sleep inf & sleep 1

docker exec -i gdrive /bin/bash << 'eof'
  export DEBIAN_FRONTEND=noninteractive
  apt-get update
  apt-get install -y software-properties-common tree elinks rsync vim jq
  add-apt-repository -y ppa:alessandro-strada/ppa
  apt-get update && apt-get install -y google-drive-ocamlfuse
eof

docker container commit gdrive google-drive-ocamlfuse:latest

docker stop gdrive
```


## Get tokens for Google Drive, sync to host, and connect
```bash
docker run --rm \
  --name gdrive \
  -v "${PWD}":/host/data/ \
  --device /dev/fuse \
  --cap-add SYS_ADMIN \
  google-drive-ocamlfuse sleep inf & sleep 1

docker exec -i gdrive /bin/bash << 'eof'
  echo -e '\n\n\nOpen this link in a browser: \n'
  google-drive-ocamlfuse 2>&1 |
    tee url.txt |
    grep -o 'https.*' |
    tr -d '")'
  echo -e '\n\n'
eof

docker exec -i gdrive /bin/bash << 'eof'
  google-drive-ocamlfuse -browser elinks
  rsync -vaR ~/./.gdfuse/default/ /host/data/z.gdrive/
  mkdir ~/google-drive
  google-drive-ocamlfuse ~/google-drive/
eof

docker exec -it gdrive /bin/bash

docker stop gdrive
```


## One you have the tokens ...
```bash
docker run --rm \
  --name gdrive \
  -v "${PWD}":/host/data/ \
  --device /dev/fuse \
  --cap-add SYS_ADMIN \
  google-drive-ocamlfuse sleep inf & sleep 1

docker exec -i gdrive /bin/bash << 'eof'
  rsync -vaR /host/data/z.gdrive/./.gdfuse/ ~/
  mkdir ~/google-drive
  google-drive-ocamlfuse ~/google-drive/
eof

docker exec -i gdrive /bin/bash << 'eof'
  ls -la ~/google-drive/
eof

docker exec -it gdrive /bin/bash
```



## Trying OAuth 2 ... not much luck
```bash
docker run --rm \
  -it \
  --name gdrive \
  -v "${PWD}":/host/data/ \
  --device /dev/fuse \
  --cap-add SYS_ADMIN \
  google-drive-ocamlfuse /bin/bash
```


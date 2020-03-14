
# Cloudflare-DNS-Updater

Most ISP provide a dynamic public IP address, once your ISP changes the public IP address, this service will pickup on this changes and update your DNS records on cloudflare, to maintain
the IP address statically (P.S this is a FREE solution, due note you could still purchase from some ISP a static IP).

## Usage

```sh
CLOUDFLARE_ACCESS_TOKEN=[ACCESS_TOKEN] \
CLOUDFLARE_ZONE_ID=[ZONE_ID] \
cloudflare update \ 
    --dns [DNS_LIST..] \
    --intervals 5
```

Run this ideally controlled by systemd, example service would like like this:

```sh
# path: /etc/systemd/system/cloudflare.service
[Unit]
Description=Cloudflare DNS Updater
Documentation=https://github.com/edenreich/cloudflare-dns-updater
Wants=network-online.target

[Install]
WantedBy=multi-user.target

[Service]
Type=notify
KillMode=process
Delegate=yes
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TasksMax=infinity
TimeoutStartSec=0
Restart=always
RestartSec=5s
ExecStartPre=-/sbin/modprobe br_netfilter
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/bin/cloudflare update \ 
    --token=[ACCESS_TOKEN] \
    --zone=[ZONE_ID] \
    --dns=[DNS_LIST..] \
    --intervals=5
```

Finally run: 
```sh
sudo systemctl enable cloudflare
sudo systemctl start cloudflare
```

## Build

Note: compilation is done using statically linking, to make sure everything comes with the binary as is.

Build for linux:

```sh
docker build -t cloudflare/linux --target normal-build -f build/Dockerfile .
```

Build on and for ARM:

```sh
docker build -t cloudflare/linux-arm7 --target arm7-build -f build/Dockerfile .
```

Copy the binaries from the containers, for example:

```sh
id=$(docker create --name cloudflare_linux cloudflare/linux) && \
docker cp cloudflare_linux:/home/rust/app/bin/cloudflare bin/cloudflare && \
docker rm $id
```

For ARM run:

```sh 
id=$(docker create --name cloudflare_linux-arm7 cloudflare/linux-arm7) && \
docker cp cloudflare_linux-arm7:/home/rust/src/bin/cloudflare bin/cloudflare_arm7 && \
docker rm $id
```

## Tests

After building the binary a simple test to check it works with standard installation of ubuntu:
```sh
docker build -t cloudflare/ubuntu-test -f tests/ubuntu/Dockerfile .
docker run --rm -it cloudflare/ubuntu-test
```

## Download

You may also download the released binary and simply use it:

```sh
sudo curl -sSL "https://github.com/edenreich/cloudflare-dns-updater/releases/download/v1.0.1/cloudflare" -o /usr/local/bin/cloudflare
# or for ARM:
# sudo curl -sSL "https://github.com/edenreich/cloudflare-dns-updater/releases/download/v1.0.1/cloudflare_arm" -o /usr/local/bin/cloudflare

sudo chmod +x /usr/local/bin/cloudflare
sudo ln -s /usr/local/bin/cloudflare /usr/bin/cloudflare
```

## Target

This project targets linux, works on ARM as well.

## Motivation

I have a raspberry pi k3s cluster at home, and I often find it annoying that the internet provider changes the IP address.
So I thought it could be nice to have a webserver without needing to worry about the IP, that would automatically update my DNS
records on cloudflare to point to that newly dynamic IP provided by my ISP.

## Bugs Reporting

If you find any bug please submit an issue ;)
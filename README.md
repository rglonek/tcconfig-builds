# tcconfig-builds
Builds of tcconfig from thombashi/tcconfig. Using critical bug fix in rglonek/tcconfig.

## TODO

Builds for centos 7+

## Build process
```bash
cat <<'EOF' > maker.sh
set -e
apt update && apt -y install git python3 python3-pip curl rename
cd /opt
git clone https://github.com/rglonek/tcconfig.git
cd tcconfig/scripts
./build_deb_package.sh
cd /opt/tcconfig/dist
mv tcconfig_*.deb tcconfig.deb
EOF

cat <<'EOF' > buildall.sh
for i in 22.04 20.04 18.04
do
docker run -itd --rm --name tcconfig ubuntu:${i}
docker cp maker.sh tcconfig:/opt/maker.sh
docker exec tcconfig /bin/bash /opt/maker.sh
docker cp tcconfig:/opt/tcconfig/dist/tcconfig.deb tcconfig-${i}-${PLATFORM}.deb
docker stop tcconfig
done
EOF
```

### build for x86_64
```bash
export PLATFORM=amd64
bash buildall.sh
```

### build for arm64
```bash
export PLATFORM=arm64
bash buildall.sh
```

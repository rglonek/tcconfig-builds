# tcconfig-builds
Builds of tcconfig from thombashi/tcconfig. Using critical bug fix in rglonek/tcconfig.

## Build process

### ubuntu script
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
```

### centos script
```bash
cat <<'EOF' > maker-centos.sh
set -e
yum -y install git python3 python3-pip curl binutils --skip-broken
cd /opt
git clone https://github.com/rglonek/tcconfig.git
cd tcconfig/scripts
PYTHON=python3
DIST_DIR_NAME="dist"
INSTALL_DIR_PATH="/usr/local/bin"
DIST_DIR_PATH="./${DIST_DIR_NAME}/${INSTALL_DIR_PATH}"
PKG_NAME="tcconfig"
cd "$(git rev-parse --show-toplevel)"
rm -rf $DIST_DIR_NAME
mkdir -p "${DIST_DIR_NAME}/DEBIAN"
$PYTHON -m pip install --upgrade "pip>=21.1"
$PYTHON -m pip install --upgrade .[buildexe,color]
PKG_VERSION=$($PYTHON -c "import ${PKG_NAME}; print(${PKG_NAME}.__version__)")
echo "$PKG_NAME $PKG_VERSION"
pip3 install --upgrade chardet==4.0.0
pyinstaller cli_tcset.py --clean --onefile --strip \
    --collect-all pyroute2 \
    --distpath $DIST_DIR_PATH \
    --name tcset
${DIST_DIR_PATH}/tcset --help

pyinstaller cli_tcdel.py --clean --onefile --strip \
    --collect-all pyroute2 \
    --distpath $DIST_DIR_PATH \
    --name tcdel
${DIST_DIR_PATH}/tcdel --help

pyinstaller cli_tcshow.py --clean --onefile --strip \
    --collect-all pyroute2 \
    --distpath $DIST_DIR_PATH \
    --name tcshow
${DIST_DIR_PATH}/tcshow --help
echo ${DIST_DIR_PATH}
cd ${DIST_DIR_PATH}
tar -zcvf ../../../tcconfig.tgz *
EOF
```

### ubuntu main script
```bash
cat <<'EOF' > buildubuntu.sh
set -e
for i in 22.04 20.04 18.04
do
docker run -itd --rm --name tcconfig ubuntu:${i}
docker cp maker.sh tcconfig:/opt/maker.sh
docker exec tcconfig /bin/bash /opt/maker.sh
docker cp tcconfig:/opt/tcconfig/dist/tcconfig.deb tcconfig-${i}-${PLATFORM}.deb
docker stop tcconfig
sleep 5
done
EOF
```

### centos main script
```bash
cat <<'EOF' > buildcentos.sh
set -e
for i in 7 stream8 stream9
do
docker run -itd --rm --name tcconfig quay.io/centos/centos:$i
docker cp maker-centos.sh tcconfig:/opt/maker.sh
docker exec tcconfig /bin/bash /opt/maker.sh
docker cp tcconfig:/opt/tcconfig/dist/tcconfig.tgz tcconfig-centos-${i}-${PLATFORM}.tgz
docker stop tcconfig
sleep 5
done
EOF
```

### runall script
```bash
cat <<'EOF' > buildall.sh
set -e
if [ "${PLATFORM}" = "" ]
then
  echo "Set PLATFORM=amd64|arm64"
  exit 1
fi
bash buildubuntu.sh
bash buildcentos.sh
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

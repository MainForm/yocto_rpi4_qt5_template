# Raspberry Pi 4 Qt6 Yocto Template

Raspberry Pi 4 64-bit target을 위한 Yocto Project 기반 Qt 6 이미지 템플릿입니다.

이 저장소는 Yocto build configuration과 필요한 layer들을 submodule로 묶어두고,
커스텀 layer인 `meta-rpi4-qt6-template`을 통해 Qt 6 런타임 이미지와 예제
애플리케이션을 제공합니다.

## Repository Layout

```text
.
├── build-rpi4/
│   └── conf/
│       ├── bblayers.conf
│       ├── local.conf
│       └── templateconf.cfg
└── sources/
    ├── poky/
    ├── meta-openembedded/
    ├── meta-raspberrypi/
    ├── meta-qt6/
    └── meta-rpi4-qt6-template/
```

## Included Layers

The repository uses the following submodules:

```text
sources/poky                  Yocto Project / Poky, scarthgap
sources/meta-openembedded     OpenEmbedded extra layers, scarthgap
sources/meta-raspberrypi      Raspberry Pi BSP layer, scarthgap
sources/meta-qt6              Qt 6 Yocto layer, 6.8
sources/meta-rpi4-qt6-template Custom Raspberry Pi 4 Qt6 layer
```

## Initialize Submodules

처음 클론한 뒤 submodule을 받아옵니다.

```bash
git submodule update --init --recursive
```

이미 받아온 submodule을 기록된 커밋으로 다시 맞추려면:

```bash
git submodule update --init --recursive
```

원격 branch 최신 커밋으로 submodule을 갱신하려면:

```bash
git submodule update --remote --recursive
```

## Install Host Packages

Ubuntu 22.04/24.04 계열에서 Yocto 빌드에 필요한 기본 패키지입니다.

```bash
sudo apt update
sudo apt install build-essential chrpath cpio debianutils diffstat file gawk gcc git iputils-ping libacl1 liblz4-tool locales python3 python3-git python3-jinja2 python3-pexpect python3-pip python3-subunit socat texinfo unzip wget xz-utils zstd
```

## Build Configuration

BitBake 환경을 활성화합니다.

```bash
source sources/poky/oe-init-build-env build-rpi4
```

명령 실행 후 현재 디렉터리는 `build-rpi4/`로 이동합니다.

현재 설정은 Raspberry Pi 4 64-bit target을 사용합니다.

```conf
MACHINE ??= "raspberrypi4-64"
```

`local.conf`에는 다음 보드 설정이 포함되어 있습니다.

```conf
LICENSE_FLAGS_ACCEPTED = "synaptics-killswitch"
ENABLE_UART = "1"
DISABLE_OVERSCAN = "1"
INIT_MANAGER = "systemd"
```

`DISABLE_OVERSCAN = "1"`은 Raspberry Pi firmware가 TV-style overscan margin을
적용해서 framebuffer 화면이 흐리거나 잘려 보이는 것을 막기 위한 설정입니다.

## Layers

`build-rpi4/conf/bblayers.conf`는 다른 checkout 경로에서도 쓸 수 있도록
`${TOPDIR}/../sources/...` 형태의 상대 경로를 사용합니다.

현재 포함된 layer:

```conf
${TOPDIR}/../sources/poky/meta
${TOPDIR}/../sources/poky/meta-poky
${TOPDIR}/../sources/poky/meta-yocto-bsp
${TOPDIR}/../sources/meta-openembedded/meta-oe
${TOPDIR}/../sources/meta-openembedded/meta-python
${TOPDIR}/../sources/meta-openembedded/meta-networking
${TOPDIR}/../sources/meta-raspberrypi
${TOPDIR}/../sources/meta-qt6
${TOPDIR}/../sources/meta-rpi4-qt6-template
```

`bitbake-layers add-layer`를 사용하면 절대 경로가 들어갈 수 있습니다. 저장소에
공유할 설정은 가능하면 상대 경로 형태를 유지합니다.

## Build Image

Qt 6 Raspberry Pi 4 이미지를 빌드합니다.

```bash
source sources/poky/oe-init-build-env build-rpi4
bitbake rpi4-qt6-image
```

빌드 결과물은 아래 경로에 생성됩니다.

```text
build-rpi4/tmp/deploy/images/raspberrypi4-64/
```

주로 사용할 이미지 파일은 `.wic.bz2`입니다.

```text
rpi4-qt6-image-raspberrypi4-64.rootfs.wic.bz2
```

## Build SDK

Qt 6 애플리케이션 개발용 SDK를 생성합니다.

```bash
source sources/poky/oe-init-build-env build-rpi4
bitbake rpi4-qt6-image -c populate_sdk
```

SDK installer는 아래 경로에 생성됩니다.

```text
build-rpi4/tmp/deploy/sdk/
```

## Provided Image

`rpi4-qt6-image`는 `meta-rpi4-qt6-template` layer에서 제공하는 이미지입니다.

포함 내용:

```text
packagegroup-core-boot
packagegroup-qt6-essentials
qt6-hello-world
OpenSSH server
NetworkManager / nmcli / Wi-Fi support
fontconfig
DejaVu fonts
Source Han Sans Korean fonts
```

## HelloWorld Demo

`qt6-hello-world`는 Qt Creator/CMake 기반 Qt Widgets 예제 애플리케이션입니다.

타겟 설치 경로:

```text
/usr/bin/qt6-hello-world
```

이미지에는 systemd service가 포함되어 부팅 후 자동 실행됩니다.

```bash
systemctl status qt6-hello-world
systemctl restart qt6-hello-world
```

서비스는 framebuffer 환경에서 실행되도록 다음 Qt 환경변수를 사용합니다.

```text
QT_QPA_PLATFORM=linuxfb
QT_QPA_GENERIC_PLUGINS=libinput
```

수동으로 실행할 때는:

```bash
QT_QPA_PLATFORM=linuxfb QT_QPA_GENERIC_PLUGINS=libinput /usr/bin/qt6-hello-world
```

## Flash Image

빌드된 `.wic.bz2` 이미지는 `bmaptool` 또는 Raspberry Pi Imager 등으로 SD card에
기록할 수 있습니다.

예:

```bash
sudo bmaptool copy build-rpi4/tmp/deploy/images/raspberrypi4-64/rpi4-qt6-image-raspberrypi4-64.rootfs.wic.bz2 /dev/sdX
```

`/dev/sdX`는 실제 SD card block device로 바꿔야 합니다.

## Useful Target Checks

보드 부팅 후 HDMI/framebuffer 확인:

```bash
cat /proc/cmdline
cat /sys/class/graphics/fb0/virtual_size
cat /sys/class/drm/card*-HDMI-A-*/modes
```

폰트 확인:

```bash
fc-match
fc-match "Source Han Sans KR"
```

서비스 확인:

```bash
systemctl status qt6-hello-world
journalctl -u qt6-hello-world
```

## Notes

`meta-rpi4-qt6-template` 안의 HelloWorld 소스는 Qt Creator 프로젝트로 교체할 수
있습니다. 다만 다음 파일들은 커밋하지 않습니다.

```text
CMakeLists.txt.user
build/
build-*/
```

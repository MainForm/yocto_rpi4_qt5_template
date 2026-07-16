# Raspberry Pi 4 with Yocto Project

Github로 배포하기 위한 Raspberry Pi 4용 Yocto 빌드 환경입니다.

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
    └── meta-raspberrypi/
```

## Initialize Submodules

처음 클론한 뒤 Yocto 레이어 submodule을 받아옵니다.

```bash
git submodule update --init --recursive
```

이미 받아온 submodule을 최신 커밋으로 갱신하려면:

```bash
git submodule update --remote --recursive
```

## Install Host Packages

Ubuntu 22.04에서 Yocto 빌드에 필요한 패키지를 설치합니다.

```bash
sudo apt update
sudo apt install build-essential chrpath cpio debianutils diffstat file gawk gcc git iputils-ping libacl1 liblz4-tool locales python3 python3-git python3-jinja2 python3-pexpect python3-pip python3-subunit socat texinfo unzip wget xz-utils zstd
```

## Activate BitBake Environment

BitBake 명령을 사용하려면 Poky의 환경 스크립트를 source 해야 합니다.

```bash
source sources/poky/oe-init-build-env build-rpi4
```

명령 실행 후 현재 디렉터리는 `build-rpi4/`로 이동합니다.

정상적으로 활성화되었는지 확인하려면:

```bash
bitbake --version
```

## Configure Layers

이 저장소의 `build-rpi4/conf/bblayers.conf`는 GitHub로 공유해도 다른 경로에서
그대로 사용할 수 있도록 `${TOPDIR}/../sources/...` 형태의 상대 경로를 사용합니다.

`bitbake-layers add-layer` 명령으로 레이어를 추가하면 현재 머신의 절대 경로가
`bblayers.conf`에 기록됩니다. 절대 경로가 들어가면 다른 환경에서 그대로 사용할 수
없으므로, 새 레이어를 추가할 때는 `build-rpi4/conf/bblayers.conf`를 직접 수정해서
상대 경로로 추가합니다.

예:

```conf
BBLAYERS ?= " \
  ${TOPDIR}/../sources/poky/meta \
  ${TOPDIR}/../sources/poky/meta-poky \
  ${TOPDIR}/../sources/poky/meta-yocto-bsp \
  ${TOPDIR}/../sources/meta-openembedded/meta-oe \
  ${TOPDIR}/../sources/meta-openembedded/meta-python \
  ${TOPDIR}/../sources/meta-raspberrypi \
  "
```

## Configure Machine

Raspberry Pi 4 이미지를 빌드하려면 `build-rpi4/conf/local.conf`에서
`MACHINE` 값을 확인합니다.

```conf
MACHINE ??= "raspberrypi4-64"
```

32-bit 이미지를 빌드하려면 다음 값을 사용할 수 있습니다.

```conf
MACHINE ??= "raspberrypi4"
```

## Build Image

BitBake 환경을 활성화한 상태에서 이미지를 빌드합니다.

```bash
bitbake core-image-minimal
```

GUI 또는 더 많은 패키지가 포함된 기본 이미지를 빌드하려면:

```bash
bitbake core-image-base
```

## Build Output

빌드 결과물은 아래 경로에 생성됩니다.

```text
build-rpi4/tmp/deploy/images/<machine>/
```

예를 들어 `MACHINE = "raspberrypi4-64"`인 경우:

```text
build-rpi4/tmp/deploy/images/raspberrypi4-64/
```

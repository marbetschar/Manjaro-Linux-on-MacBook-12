# Manjaro-Linux-on-MacBook-12
Online repository with a customized Manjaro .iso which runs on MacBook 12".

## Downloads
- [.iso Manjaro Deepin for MacBook 12"](???)

## How the .iso was built

### Preconditions

The following steps were taken within a Manjaro Linux VirtualBox environment, running on a macOS Mojave host machine. So I assume you are running the following commands within an existing Manjaro Linux.

#### Install Manjaro Tools

```
sudo pacman -Syu manjaro-tools
```

#### Copy iso-profile template

```
ISO_PROFILE=community/deepin
mkdir -p ~/iso-profiles
cp -r /usr/share/manjaro-tools/iso-profiles/shared ~/iso-profiles/shared
mkdir -p ~/iso-profiles/$ISO_PROFILE
cp -r /usr/share/manjaro-tools/iso-profiles/$ISO_PROFILE ~/iso-profiles/$ISO_PROFILE
```

### Build package and prepare user repository

#### Create the macbook12-repo

```
mkdir -p ~/macbook12-repo/x86_64
```

#### Build the macbook12-spi-driver-dkms package

```
mkdir ~/pkgbuild
cd ~/pkgbuild
git clone https://aur.archlinux.org/macbook12-spi-driver-dkms
buildpkg -b stable -a x86_64 -p macbook12-spi-driver-dkms
```

#### Copy macbook12-spi-driver-dkms package to macbook12-repo

```
cp /var/cache/manjaro-tools/pkg/stable/x86_64/macbook12-spi-driver-dkms*.pkg.tar.xz ~/macbook12-repo/x86_64/
```

#### Build repo .db file

```
cd ~/macbook12-repo/x86_64/
repo-add macbook12-repo.db.tar.gz *.pkg.tar.*
```

### Add macbook12 repository to iso profile

Create `user-repos.conf` in your iso profile:

```
ISO_PROFILE=community/deepin
cd ~/iso-profiles/$ISO_PROFILE
vi user-repos.conf
```

and add the following content:

```
[macbook12-repo]
SigLevel = Optional TrustAll
Server = https://manjaro.marco.betschart.name/$repo/$arch
```

#### Add macbook12 package to iso profile

Add the built macbook12 package name at the end to the list of packages in `Packages-Desktop` and `Packages-Live` (stored in `~/iso-profiles/$ISO_PROFILE`):

```
#macbook12-repo
macbook12-spi-driver-dkms
```

### Build iso

```
ISO_PROFILE=community/deepin
cd ~/iso-profiles
cd $(dirname $ISO_PROFILE)
# `buildiso` needs to be run as root!
sudo buildiso -p $(basename $ISO_PROFILE)
```

## Further reading
- [Build Manjaro ISOs with buildiso](https://wiki.manjaro.org/Build_Manjaro_ISOs_with_buildiso)
- [Buildiso with AUR packages: Using buildpkg](https://wiki.manjaro.org/Buildiso_with_AUR_packages:_Using_buildpkg)

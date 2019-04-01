# Manjaro Linux on MacBook 12"
Online repository with a customized Manjaro .iso which runs on MacBook 12".

## Downloads
- [Manjaro Deepin for MacBook 12"](https://github.com/marbetschar/Manjaro-Linux-on-MacBook-12/releases)

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
# macbook12-repo
macbook12-spi-driver-dkms
```

#### Ignore KERNEL-rt3562sta

Add a hash-sign in front of `KERNEL-rt3562sta` in the `~/iso-profiles/shared/Packages-Mhwd` file to avoid installing this package if you are building for `linux419` - otherwise you'll likely encounter a `error: target not found: linux419-rt3562sta` during the `buildiso` command:

```
#KERNEL-rt3562sta
```

#### Make sure the macbook12 drivers are loaded at boot

Create a file called `macbook12.conf` with the following content:

```
# macbook12 apple spi
applespi
spi_pxa2xx_platform
intel_lpss_pci
```

... and copy it to the following two locations:

- `~/iso-profiles/$ISO_PROFILE/desktop-overlay/etc/modules-load.d/macbook12.conf`
- `~/iso-profiles/$ISO_PROFILE/live-overlay/etc/modules-load.d/macbook12.conf`

### Build iso

```
export ISO_PROFILE=community/deepin
cd ~/iso-profiles
cd $(dirname $ISO_PROFILE)
# `buildiso` does not need to be run as root!
buildiso -f -p $(basename $ISO_PROFILE) -k linux419 -b stable -a x86_64
```

#### If you encounter an error saying...

>
> *`Failed to convert to ACE; could not convert string to UTF-8`*
>
> ... then make sure you ...
>
> *double check your encoding and make sure your `user-repos.conf` is UTF-8 and
> does not contain any strange hidden characters! If in doubt,
> use a coding editor and create a new one from scratch!*
>

>
> *`error: target not found: yaourt`*
>
> ... then make sure you ...
>
> replace `yaourt` in `Packages-Desktop` with `yay`.
>

### Create a bootable media with the iso

Now you are finished creating the iso. Create a bootable media from it and you should be all set.

## Further reading
- [Build Manjaro ISOs with buildiso](https://wiki.manjaro.org/Build_Manjaro_ISOs_with_buildiso)
- [Buildiso with AUR packages: Using buildpkg](https://wiki.manjaro.org/Buildiso_with_AUR_packages:_Using_buildpkg)

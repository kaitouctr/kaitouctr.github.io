---
title: "Creating AppImages for PokeFinder and PokeGlitzer"
date: 2024-08-04
tags: ["pokemon", "linux"]
draft: True
---

**This stuff is simply documenting my experience in trying to create AppImages for the aforementioned applications**

## Background

I have been running a set up based around Fedora Atomic Desktops for a while now. These operating systems use OS images as the main method of delivering updates and expect a container-based workflow. I also like to play around with Pokémon, particularly the technical side of it such as pseudorandom number generation (PRNG) manipulation and glitching around (particularly in Generation III). However the tools used for these activities were definitely not distributed with this kind of system. 

PokeFinder (a tool that simulates the PRNG of various Pokémon games) is just distributed as a binary executable file. This format does not include any dependencies that the software may rely on, which on atomic distributions necessitates a container (in PokeFinder’s case is really only the case for GNOME-based distributions as KDE-based ones have Qt already). I don’t particularly like having to create a brand new container just to be able to run an application as I feel it creates quite a bit of bloat.

PokeGlitzer is not much better. It is shipped as an artifact from a `dotnet build` command without any kind of self-containment. As `dotnet` is usually not installed on most Linux distributions, this means that has to be downloaded first before being able to use the program. On atomic distributions it inherits the same issues as binary executables, and thus also needs a container.

However, for myself anyway, there is a format that allows me to contain the binary and all of its dependencies into one file. It is the AppImage format, largely because of how simple it is to get an application up and running with all of its dependencies bundled in.

## Building PokeFinder

This one caused me the least amount of headaches so I will start with this. First I the PokeFinder binary from the [GitHub](https://github.com/Admiral-Fish/PokeFinder) I simply followed most of [this](https://docs.appimage.org/packaging-guide/from-source/native-binaries.html). I placed `PokeFinder` in `AppDir/usr/bin`, created a `.desktop` file at `AppDir/usr/share/applications`, got `PokeFinder.ico` from `Source/Form/Images` folder in the GitHub repository (converted with ImageMagick, chose the largest size) then ran the `appimagetool` found at the [probonopd/go-appimage](https://github.com/probonopd/go-appimage) repository. Below is an approximation of the commands I have used:

```
./appimagetool*.AppImage -s deploy AppDir/usr/share/applications/PokeFinder.desktop
./appimagetool*.AppImage AppDir
```

## Building PokeGlitzer

To make the process of building the appimage simpler, I did have to install `dotnet` and clone the [PokeGlitzer repository](https://github.com/E-Sh4rk/PokeGlitzer) from GitHub. The `dotnet` was in a container, also make sure the container’s base distribution is Ubuntu 20.04 LTS (this will save quite a bit of annoyance in later parts). I cd’d into the cloned repository and ran the below command:

```
dotnet publish -r linux-x64 -o ../AppDir/usr/bin --sc true
```

This produces a folder that contains a lot of files, but the most important thing about it is that it bundles `dotnet` within it eliminating the need to download it separately. In theory the job is already done by that folder full of files but it does not feel elegant having to zip and unzip that all the time and needing to search through for the binary produced by `dotnet`.

Then using [this guide](https://github.com/AppImage/AppImageKit/wiki/Bundling-.NET-Core-apps#bundling-within-a-docker-container), I created the `AppRun` with the following content (with `chmod 755` applied onto it):

```
#!/bin/sh
HERE="$(dirname "$(readlink -f "${0}")")"
export PATH="${HERE}"/usr/bin/:"${PATH}"
EXEC=$(grep -e '^Exec=.*' "${HERE}"/*.desktop | head -n 1 | cut -d "=" -f 2 | cut -d " " -f 1)
exec "${EXEC}" $@
```

And the `PokeGlitzer.desktop` :
```
# Desktop Entry Specification: https://standards.freedesktop.org/desktop-entry-spec/desktop-entry-spec-latest.html
[Desktop Entry]
Type=Application
Name=PokeGlitzer
Comment=Gen3 Save Editor for Glitchers
Icon=pokeglitzer
Exec=PokeGlitzer
Path=~
Terminal=false
Categories=Utility;
```

Also I have copied the icon from the `PokeGlitzer/Resources` folder within the PokeGlitzer GitHub repository to serve as the icon for the AppImage. There the folder structure all prepared! Then I had to install the below packages in the container to make sure that the AppImage building works properly.

```
sudo apt install file squashfs-tools libsquash-dev liblttng-ust0
```

Note that `liblttng-ust0` is an old version of the lttng library which means that newer distributions most likely do not have it available, thus the requirement that the container must run Ubuntu 20.04. Also I did not know for quite a long time that I needed to install `file`, as the error message that `appimagetool` printed out was confusing:

```
Required helper tool file is not found
```

I honestly thought that there was a helper tool file somewhere that AppImage needed but then I slowly realised that it was the helper tool `file` not a helper tool file that I needed. I think wasted so much time on that.

After all that, I ran the same command as I did with PokeFinder to package it all into an AppImage and then I was done.

```
./appimagetool*.AppImage AppDir
```

## Conclusion

Yes, I literally just regurgitated the official AppImage guides on how to make AppImages. This was just detailing my experience in trying to create an AppImage for these two apps just so I can run them for my specific Linux environment.
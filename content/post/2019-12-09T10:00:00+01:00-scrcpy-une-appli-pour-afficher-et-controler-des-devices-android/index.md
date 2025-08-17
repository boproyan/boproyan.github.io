+++
title = 'scrcpy, une appli pour afficher et contrôler des devices Android'
date = 2019-12-09T10:00:00+01:00 00:00:00 +0100
categories = android
+++
Bonjour nal,

Je viens te présenter une application que j'ai développée, qui permet d'afficher et de contrôler des _devices_ Android connectés en USB.

[![scrcpy](https://raw.githubusercontent.com/Genymobile/scrcpy/master/assets/screenshot-debian-600.jpg)]

    [github]: https://github.com/Genymobile/scrcpy

Elle se concentre sur :

- la **légèreté** (native, affiche uniquement l'écran)
- les **performances** (30~60fps)
- la **qualité** (1920×1080 ou plus)
- la **faible latence** (70~100ms)
- un **démarrage rapide** (~1 seconde pour afficher la première image)
- la **non-intrusivité** (rien ne reste installé sur le device)

Je l'ai appelée [scrcpy][github].

Il fallait un nom aussi [imprononçable](https://github.com/Genymobile/scrcpy#why-scrcpy) que mon précédent projet, [gnirehtet](https://linuxfr.org/news/du-reverse-tethering-sur-android-sans-root) (tu te souviens peut-être, je t'avais parlé de sa [réécriture en Rust](https://linuxfr.org/users/rom1v/journaux/du-reverse-tethering-en-rust)).

Cette fois-ci, c'est une application en C qui utilise [SDL](https://www.libsdl.org/) et [libav/FFmpeg](https://www.libav.org/).


## Compiler et installer

Pour la compiler et l'installer, tout est expliqué dans le [README](https://github.com/Genymobile/scrcpy/blob/master/README.md).

Le plus simple, c'est de prendre la partie serveur [déjà compilée](https://github.com/Genymobile/scrcpy/blob/master/README.md#prebuilt-server) (ça t'évitera d'installer Java et le SDK Android).

Ensuite (pour _Debian_/_Ubuntu_) :

```bash
sudo apt install android-tools-adb ffmpeg libsdl2-2.0.0 \
                 make gcc pkg-config meson \
                 libavcodec-dev libavformat-dev libavutil-dev \
                 libsdl2-dev

# replace by the path where you downloaded scrcpy-server.jar
meson x --buildtype release --strip -Db_lto=true \
    -Dprebuilt_server=/path/to/scrcpy-server.jar
cd x
ninja
sudo ninja install
```

Quelqu'un a aussi fait un paquet [AUR](https://aur.archlinux.org/packages/scrcpy/) pour Arch.

## Exécuter

C'est assez simple :

```bash
scrcpy
```

Il est possible de passer des [options](https://github.com/Genymobile/scrcpy#run), décrites dans l'aide :

```bash
scrcpy --help
```

Une fois l'écran du _device_ affiché, des [raccourcis](https://github.com/Genymobile/scrcpy#shortcuts) permettent d'effectuer des actions spéciales.

J'espère que cette application pourra t'être utile à l'occasion ;-)


## Liens (en anglais)

- [Présentation du projet sur mon blog](https://blog.rom1v.com/2018/03/introducing-scrcpy/)
- [Projet sur github][github]
- [Page développeurs](https://github.com/Genymobile/scrcpy/blob/master/DEVELOP.md)
- [Hacker News](https://news.ycombinator.com/item?id=16544977)
- [Reddit](https://www.reddit.com/r/Android/comments/834zmr/introducing_scrcpy_an_app_to_display_and_control/)

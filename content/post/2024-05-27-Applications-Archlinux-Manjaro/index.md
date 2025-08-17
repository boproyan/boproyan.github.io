+++
title = 'Applications Archlinux/Manjaro'
date = 2024-05-27 00:00:00 +0100
categories = ['archlinux', 'manjaro']
+++
# Applications Archlinux/Manjaro

>Les paquets s'installent par la commande `sudo pacman -S NomDuPaquet`  
Pour les paquets dans les dépôts AUR, `yaourt -S NomDuPaquet` ou `yay -S NomDuPaquet`

**INFO : Les éléments <s>rayés</s> ne sont pas pris en compte**

* XFCE , ne rien afficher sur les bureaux : clic droit sur écran , Bureau et dévalider les icônes par défaut
*  **nfs-utils** , **imagemagick** , **ntp** , **rsync** ,**vlc** *(Installés par défaut avec manjaro capella 15.12 et +)*
* [Pacman et Yaourt - Manjaro Linux](http://wiki.manjaro.org/index.php?title=Pacman_et_Yaourt)
    * Mise à jour de la base, des paquets des dépôts plus ceux de AUR : `yaourt -Syua`
    * [Pacman : Accélérer la compilation](http://la-vache-libre.org/accelerer-la-compilation-si-jose-dire-des-paquets-avec-pacman-sous-arch-linux-manjaro-et-derives/) 
    * [pacman/Tips and tricks - ArchWiki](https://wiki.archlinux.org/index.php/pacman_tips)
    * [Astuces Pacman — ArchwikiFR](https://wiki.archlinux.fr/Astuces_Pacman)
    * [yaourt paginé et en couleur - Vintherine : le blog](http://blog.vintherine.org/post/2014/11/16/yaourt-pagine-et-en-couleur)
* [Haveged - ArchWiki](https://wiki.archlinux.org/index.php/Haveged)
* *Format exfat (SDcard >32Go) : <s>yay -S exfat-utils fuse-exfat , format: mkfs.exfat ,montage :mount -t exfat</s>
*  `Restauration du dossier` **.ssh** : cd ~ ; cp -r /home/yannick_sav/.ssh/ .
* **Gestion des mots de passe**
    * **keepassXC** ,   `sudo pacman -S keepassxc` , ouverture avec clé + mot de passe
        *  Base de données dans le dossier **~/.keepassx**
        *  Clé  keepassxc dans dossier  **~/.ssh/**
   * **[KeePassDX](#keepassdx)** , version amékiorée `sudo pacman -S keepassdx`
* **Multimédia**
    * **Musique** Strawberry : `yay -S strawberry`
    * [Calibre](#calibre) `sudo pacman -S calibre`
    * **MKV** ([Matroska Video](v)) est un format vidéo entièrement libre. Plus exactement il s'agit d'un conteneur permettant de contenir de la vidéo (DivX, Xvid,RV9, etc.), du son... et les outils mkvtoolnix
        * `yay -S makemkv mkvtoolnix-cli mkvtoolnix-gui`
    *  **Kodi** (ex XBMC) 
    *  Ripper CD [Asunder – Un CD ripper/encoder audio qui roxe du poney](https://la-vache-libre.org/asunder-un-cd-ripperencoder-audio-qui-roxe-du-poney/) : `yay -S asunder`
    * **YouTube**
        * [FreeTube](https://holory.fr/freetube/) : Le Client YouTube Open Source `yay -S freetube-bin`
        * **[youtube-dl](https://youtube-dl.org/)** To install it right away for all UNIX users (Linux, OS X, etc.), type:  
        `sudo curl -L https://yt-dl.org/downloads/latest/youtube-dl -o /usr/local/bin/youtube-dl`  
        `sudo chmod a+rx /usr/local/bin/youtube-dl`
        * En cas de problème avec youtube-dl, installer `yay -S yt-dlp`
        * **mps-youtube** recherche écoute musique et vidéo en ligne de commande : `yay -S mps-youtube mpv`
    * **Radios**
        *  [Goodvibes](https://github.com/elboulangero/goodvibes) est un lecteur de radios Internet pour GNU/Linux, sous licence GPL v3+. `yay -S goodvibes`  
        * [Tuner](https://github.com/louis77/tuner) : `yay -S geocode-glib tuner-git`
        *  **Radiotray** ,lecteur simplifié des radios sur internet  `yay -S radiotray`
* [VirtualBox](/posts/VirtualBox/)
* VPN
    * [Mullvad](/posts/Mullvad-VPN-IPV4-IPV6/)
    * [Wireguard (serveur vpn)](/files/VPN-Wireguard.pdf)
*  Cartographie
    *  **Viking** GTK+2 application to manage GPS data : `yay -S viking`
    *  **GPXSee** :(pour remplacer viking) `yay -S gpxsee`
* **Edition, développement, conversion** 
    *  **gedit** , `sudo pacman -S gedit` #qui n'est plus installé par défaut
    *  **Remarkable** Editeur + Visuel markdown <https://github.com/jonschlinkert/remarkable>  avec Node.js installé , `npm install remarkable --save`
    *  **Retext** Editeur + Visuel markdown : `sudo pacman -S retext qt5-webkit` ou **Ghostwriter** `yay -S ghostwriter` 
        * il faut créer un lien pour le dossier "images" dans le répertoire qui contient les fichiers markdown : `sudo ln -s $HOME/Images /images`
    *  **Bluefish** éditeur : `sudo pacman -S bluefish`
    * Editeur **nano** , syntaxe "highlight" : `yay nano-syntax-highlight` <https://github.com/scopatz/nanorc>  
    *  **Pandoc** Conversion between markup formats : `yay -S pandoc-bin`
    *  **[QOwnNotes](http://www.qownnotes.org/)** : Plain-text file notepad with markdown support and nextcloud integration  `yay -S qownnotes`
    *  **Inkscape** est un logiciel de dessin vectoriel professionnel ,`yay -S inkscape`
    *  [Wing Personal Python IDE](https://wingware.com/downloads/wing-personal) 
        * Télécharger et décompresser la version linux (ex wing-personal-7.1.2.0-linux-x64.tar.bz2)
        * En mode su , `cd wing-personal-7.1.2.0-linux-x64` , `./wing-install.py --winghome /opt/wing-personal --bin-dir /usr/local/bin`
    * **Arduino** : `yay -S arduino` et ajouter votre utilisateur aux groupes uucp et lock
        * `sudo usermod -a -G uucp $USER`
        * `sudo usermod -a -G lock $USER`
        * `sudo modprobe cdc_acm` # charger le module cdc_acm
    * **KiCad** est une suite logicielle libre de conception pour l'électronique pour le dessin de schémas électroniques et la conception de circuits imprimés. KiCad est publié sous licence GNU GPL: `sudo pacman -S kicad`
    * **Atom** est un éditeur de texte libre pour macOS, GNU/Linux et Windows développé par GitHub `yay -S atom-editor-git`   
Avec flatpak : `sudo flatpak install flathub io.atom.Atom -y`
* **Torrent**
    *  **qbittorent**  `sudo pacman -S qbittorrent`
    *  [Téléchargez des torrents depuis votre terminal avec Torrench](https://homputersecurity.com/2017/08/27/telechargez-des-torrents-depuis-votre-terminal-avec-torrench/)  
* **Outils**
    * **Geeqie** is a free open software image viewer and organiser program for Linux, FreeBSD and other Unix-like operating systems
    * [Domain Expiration Check Shell Script](https://www.cyberciti.biz/tips/domain-check-script/)  
Prérequis "whois" : sudo pacman -S whois
    * **Double Commander** est un gestionnaire de fichiers qui offre deux fenêtres simultanées : `yay -S doublecmd-qt5`
    *  **Vfat système de fichier** `sudo pacman -S dosfstools` , **mkfs.vfat -F 32 /dev/sdx1**
    *  **hardinfo**, Afficher le matériel `yay -S hardinfo`
    *  Outil disque **smartmontools** <https://wiki.archlinux.org/index.php/S.M.A.R.T.> , `yay -S smartmontools`
    *  **OpenJDK** Java 8 full runtime environment : `sudo pacman -S jre8-openjdk`
    * **réseaux** et autres : `sudo pacman -S bind-tools net-tools lsof qrencode`
    * Diagrammes réseaux et autres...
        *  **DIA** : Un remarquable logiciel de réalisation de diagrammes (réseau, circuit électrique, programme informatique, etc.) `sudo pacman -S dia`
        * **draw.io** ([diagrams.net](https://www.diagrams.net/)) est une application de création de diagrammes et schémas sous licence Apache disponible sous Windows, MacOs, Linux, sous forme d'application web et intégrée à des services cloud tels [NextCloud](https://apps.nextcloud.com/apps/drawio) ou Google Drive. La version web <https://www.draw.io/> et bureau ([télécharger](http://get.diagrams.net/))
    *  [DBeaver](https://dbeaver.io/) : `yay -S dbeaver` *Free multi-platform database tool for developers, SQL programmers, database administrators and analysts. Supports all popular databases: MySQL, PostgreSQL, MariaDB, SQLite, Oracle, DB2, SQL Server, Sybase, MS Access, Teradata, Firebird, Derby, etc...*
    *  [tout savoir sur son système Linux avec la commande inxi](https://memo-linux.com/tout-savoir-sur-son-systeme-linux-grace-a-la-commande-inxi/) : `yay -S inxi`
    * **Nmap** : `yay -S nmap`
    * **KeepassDX** gestion des mots de passe 
    * **Menulibre** pour éditer les .desktop xfce : `yay -S menulibre`
    * **FileZilla** : `yay -S filezilla`
    * **Git** -> `sudo pacman -S git` 
    * **nextcloud** :Client nextcloud  `sudo pacman -S nextcloud-client` puis paramétrer
        *  1 ière connexion , saisir le mot de passe du trousseau de clés "Par défaut"
        *  2 ième connexion , saisir valider le fait qu'il mémorise le mot de passe du trousseau
        *  3 ième connexion , nextcloud-client démarre sans demander le mot de passe
    * **eg** est un utilitaire qui fournit ce qui manque aux Man Pages, c ‘est à dire des exemples corrects et clairs de ce qu’on peut faire avec telle ou telle commande: `yay -S eg`
    * **Grub Customiser** ,paramétrer grub avec un utilitaire graphique: `yay -S grub-customizer` 
    * **VNC Viewer Client **[linux standalone x64](https://www.realvnc.com/fr/connect/download/viewer/linux/)(*vnc connect by RealVNC*),télécharger l'appimage et vérifier le checksum   
SHA-256: SHA-256: 4ac20464566dc6756325bb2476f82f495d7479eb7587f2dd65b3d3f1164d8648  
Déplacer le fichier dans **/usr/local/bin** avec le nom souhaité (effacer l'ancien avant)   
`sudo rm -f /usr/local/bin/realvnc && sudo mv VNC-Viewer-* /usr/local/bin/realvnc && sudo chmod +x /usr/local/bin/realvnc`  
Utilisé pour se connecter via VNC sur les raspberry pi (*serveur vnc connect by RealVNC*)  
     * **PDF**
         * **xournal**, prendre des notes sur des documents PDF, les annoter, ajouter des images personnalisées : `yay -S xournal`
         * **PDFTK** est un programme en ligne de commande permettant d'effectuer certaines manipulations de documents PDF, comme la mise en arrière-plan, la concaténation, extraction de pages, le remplissage des formulaires, etc.: `yay -S pdftk` , [documentation](https://doc.ubuntu-fr.org/pdftk)
* Terminal
    * **Terminator** : `yay -S terminator` ,fichier configuration **~/.config/terminator/config** et modifier "Applications Favorites"
    *  [xterm](https://wiki.archlinux.fr/Xterm) est un émulateur de terminal hautement configurable. `sudo pacman -S xterm`
    * [minicom] ({{ site.url_wiki }}/doku.php?id=yannick:minicom) `yay -S minicom`
    * [Broot](https://dystroy.org/broot/) tree,ls,cd,etc...
        * `sudo wget -O /usr/local/bin/broot https://dystroy.org/broot/download/x86_64-linux/broot`
        * `sudo chmod +x /usr/local/bin/broot`
*  [Enregistreurs de terminaux](https://linuxtaka.wordpress.com/2017/05/21/enregistreurs-de-terminaux-5-tools-qui-vous-facilitent-la-vie/)
    *  **TermRecord** est un simple enregistreur de session de terminal qui produit les enregistrements vers une sortie HTML autonome facile à partager.  
        * `sudo pip install TermRecord`  # Installation (python pip préalablement installé)
        * `TermRecord -o /var/www//termrecord/` # Créer fichier/ et démarrer , `exit` pour arrêt
        * <https://server/termrecord/> , lecture
    *  **script** ,une commande dans Linux qui sert à enregistrer les activités du terminal.
        * `script --timing=time.txt  linuxtakademo.txt` # Départ enregistrement , `exit` # Arrêt enregistrement
        * `scriptreplay  --timing=time.txt linuxtakademo.txt` # Lecture enregistrement
* Enregistreur et capture d'écran
    * **[Simple Screen Recorder](https://www.maartenbaert.be/simplescreenrecorder/)** est un logiciel à base de Linux pour l'enregistrement des jeux et des programmes `sudo pacman -S simplescreenrecorder`
* Copie d'écran
    * **[Flameshot](https://github.com/lupoDharkael/flameshot)** c’est un peu THE TOOL pour faire des captures d’écrans `sudo pacman -S flameshot`
* **ANDROID**
    * Outil adb : `yay -S android-tools`
    * **[scrcpy](https://github.com/Genymobile/scrcpy)** outil pour visualiser et contrôler votre téléphone Android depuis un bureau Linux, Windows ou MacOS : `yay -S scrcpy` 
* **[Postman](https://www.postman.com/)** est un outil permettant de manipuler une API depuis une interface graphique
    1. télécharger la version native de PostMan correspondant à sa plateforme, à savoir x86 ou x64, depuis l’adresse <https://www.getpostman.com/apps>
    * décompressé dans le répertoire /opt : `    sudo tar -xzf [LeNomDeVotreArchive.tar.gz] -C /opt`
    * mettre l’application PostMan dans le chemin des exécutables : `sudo ln -s /opt/Postman/Postman /usr/local/bin/postman`
* Navigateurs
    * **Firefox**
        * **Add custom search engine** <https://addons.mozilla.org/fr/firefox/addon/add-custom-search-engine/> ,pour ajout moteur de recherche (plus nécessaire avec les versions récentes)
        * Désactiver pocket: `about:config`  &rarr; `extensions.pocket.enabled false`
    * **Tor Browser** : [Le navigateur Tor (fr) avec linux](/posts/navigateur-Tor-fr-linux/)
    * **Epiphany** : `yay -S epiphany`
    * **Brave** : `yay -S brave-bin`
* Messagerie **Signal** **Tunanota**
    * Signal : `yay -S signal-desktop`
    * Courrier électronique crypté.**Tutanota** est le service d'e-mails le plus sécurisé au monde : `yay -S tutanota-desktop-bin` 
* Météo
     * Météo Radar <https://webcatalog.io/fr/desktop/>


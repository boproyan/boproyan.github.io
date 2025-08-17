+++
title = 'Create/Compress/Archive , Extract/Uncompress/Unarchive Almost Any File in Linux (tar, tar.gz, tar.bz2, gz, bz, zip, 7z, rar, etc…)'
date = 2023-01-13 00:00:00 +0100
categories = ['outils']
+++
## Create/Compress/Archive , Extract/Uncompress/Unarchive Almost Any File in Linux (tar, tar.gz, tar.bz2, gz, bz, zip, 7z, rar, etc…)

[Archiving and compression](https://wiki.archlinux.org/title/Archiving_and_compression)

Suffixe Archive | Create/Compress/Archive | Extract/Uncompress/Unarchive 
 -------------- |-----------------------| ----------------------------
**.tar** | tar cvf filename.tar /dir | tar xvf filename.tar
**.tar.gz** | tar czvf filename.tar.gz /dir | tar xzvf filename.tar.gz
**.tgz** | tar cvzf filename.tgz /dir | tar xvzf filename.tgz
**.tar.bz** | tar cjvf filename.tar.bz /dir | tar xjvf filename.tar.bz
**.tbz** | tar cjvf filename.tbz /dir | tar xjvf filename.tbz
**.tar.bz2** | tar cjvf filename.tar.bz2 /dir | tar xjvf filename.tar.bz2
**.tar.xz** | tar cvf -  filenames | lzma > filename.tar.xz  
**.xz** | xz filename | xz -d -v filename.tar.xz
**.gz** | gzip filename | gunzip filename.gz
**.gz2** | You probably mean .bz2 | You probably mean .bz2
**.bz** | bzip filename | bunzip filename.bz
**.bz2** | bzip2 filename | bunzip2 filename.bz2
**.xz** | lzma filename | unlzma filename.xz
**.zip** | zip -r filename.zip /dir | unzip filename.zip
**.7z** | 7z a -t7z filename.7z /dir | 7z x filename.7z
**.rar** | rar a filename.rar /dir | unrar x filename.rar


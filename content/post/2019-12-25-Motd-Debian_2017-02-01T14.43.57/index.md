+++
title = 'Motd-Debian_2017-02-01T14.43.57'
date = 2018-11-23 00:00:00 +0100
categories = []
+++
Motd-Debian 2017-02-01T14.43.57
========================
---
layout: article
title:  motd , message de bienvenue sur connexion en ligne de commande
toc: true
ref: (falcutatif)
date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
tags: [debian]
lang: fr
description: un message de bienvenue accueille l'utilisateur lors d'une connexion en ligne de commande (motd)
---

## Motd

  * <https://oitibs.com/debian-wheezy-dynamic-motd/>
  * <https://nickcharlton.net/posts/debian-ubuntu-dynamic-motd.html>

Avec Ubuntu, un ensemble de scripts fournis existe dans **/etc/update-motd.d/**, ils sont exécutés dans l'ordre croissant pour produire un fichier statique **/etc/motd**. \\
<wrap hi>Comme la fonctionnalité est gérée par PAM, sur Debian, nous avons juste besoin de créer le répertoire **/etc/update-motd.d/** et le remplir avec un ensemble de scripts.</wrap>\\
Comme la plupart des configurations de style **config.d**, les fichiers sont exécutés dans l'__ordre croissant__(préfixer chaque script avec 00-99 organisera l'ordre de la sortie finale). 

	sudo -s

### Prérequis

```
# Générateur ASCII Art 
apt install figlet -y
# create directory
mkdir /etc/update-motd.d/
# create dynamic files
touch /etc/update-motd.d/{00-header,10-sysinfo,20-updates,99-footer}
# make files executable
chmod +x /etc/update-motd.d/*
# remove MOTD file
rm /etc/motd
# symlink dynamic MOTD file
ln -s /var/run/motd /etc/motd
#Make sure you have a working python-apt package.
apt install python-apt
# goto folder
cd /etc/update-motd.d/
```

### Entête 00-header

Le fichier **00-header**

	nano 00-header

```
#!/bin/sh
#
#    00-header - create the header of the MOTD
#    Copyright (c) 2013 Nick Charlton
#    Copyright (c) 2009-2010 Canonical Ltd.
#
#    Authors: Nick Charlton <hello@nickcharlton.net>
#             Dustin Kirkland <kirkland@canonical.com>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

[ -r /etc/lsb-release ] && . /etc/lsb-release

if [ -z "$DISTRIB_DESCRIPTION" ] && [ -x /usr/bin/lsb_release ]; then
        # Fall back to using the very slow lsb_release utility
        DISTRIB_DESCRIPTION=$(lsb_release -s -d)
fi

figlet $(hostname)
printf "\n"

printf "Bienvenue sur %s (%s).\n" "$DISTRIB_DESCRIPTION" "$(uname -r)"
printf "\n"
```

### Info système 10-sysinfo OU 11-sysinfo

>Utiliser l'un des 2 fichiers , **10-sysinfo OU 11-sysinfo**  
10-sysinfo pour les noyaux < 3.00.0  
11-sysinfo pour les noyaux récents > 3.00.0  

Informations systèmes

	nano 10-sysinfo

```
#!/bin/bash
#
#    10-sysinfo - generate the system information
#    Copyright (c) 2013 Nick Charlton
#
#    Authors: Nick Charlton <hello@nickcharlton.net>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

date=`date`
load=`cat /proc/loadavg | awk '{print $1}'`
total_disk=`df -h / | awk '/\// {print $(NF-4)}'`
root_usage=`df -h / | awk '/\// {print $(NF-1)}'`
memory_usage=`free -m | awk '/Mem/ { printf("%3.1f%%", $3/$2*100) }'`
swap_usage=`free -m | awk '/Swap/ { printf("%3.1f%%", $3/$2*100) }'`
users=`users | wc -w`

echo "System information as of: $date"
echo
printf "Total disk:\t%s\tMemory usage:\t%s\n" $total_disk $memory_usage
printf "Usage on /:\t%s\tSwap usage:\t%s\n" $root_usage $swap_usage
printf "Local users:\t%s\n" $users
echo
```

Informations systèmes (debian jessie)

	nano 11-sysinfo

```
#!/bin/bash
#
#    11-sysinfo - generate the system information
#    Copyright (c) 2013 Nick Charlton
#
#    Authors: Nick Charlton <hello@nickcharlton.net>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.
 
date=`date`
load=`cat /proc/loadavg | awk '{print $1}'`
total_disk=`df -h / | awk '/\// {print $(NF-4)}'`
root_usage=`df -h / | awk '/\// {print $(NF-1)}'`
memory_available=`cat /proc/meminfo | awk '/MemAvailable/ {printf($2)}'`
memory_total=`cat /proc/meminfo | awk '/MemTotal/ {printf($2)}'`
swap_usage=`free -m | awk '/Swap/ { printf("%3.1f%%", $3/$2*100) }'`
users=`users | wc -w`
 
echo "System information as of: $date"
echo
printf "Total disk:\t%s\tUsage on /:\t%s\tSwap usage:\t%s\n" $total_disk $root_usage $swap_usage
printf "Memory Total:\t%s KB\tMemory Available:\t%s KB\n" $memory_total $memory_available
echo
```

### Mises à jour 20-updates

Mises à jour (updates)

	nano 20-updates

```
#!/usr/bin/python
#
#   20-updates - create the system updates section of the MOTD
#   Copyright (c) 2013 Nick Charlton
#
#   Authors: Nick Charlton <hello@nickcharlton.net>
#   Based upon prior work by Dustin Kirkland and Michael Vogt.
#
#   This program is free software; you can redistribute it and/or modify
#   it under the terms of the GNU General Public License as published by
#   the Free Software Foundation; either version 2 of the License, or
#   (at your option) any later version.
#
#   This program is distributed in the hope that it will be useful,
#   but WITHOUT ANY WARRANTY; without even the implied warranty of
#   MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#   GNU General Public License for more details.
#
#   You should have received a copy of the GNU General Public License along
#   with this program; if not, write to the Free Software Foundation, Inc.,
#   51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

import sys
import subprocess
import apt_pkg

DISTRO = subprocess.Popen(["lsb_release", "-c", "-s"],
                          stdout=subprocess.PIPE).communicate()[0].strip()

class OpNullProgress(object):
    '''apt progress handler which supresses any output.'''
    def update(self):
        pass
    def done(self):
        pass

def is_security_upgrade(pkg):
    '''
    Checks to see if a package comes from a DISTRO-security source.
    '''
    security_package_sources = [("Ubuntu", "%s-security" % DISTRO),
                               ("Debian", "%s-security" % DISTRO)]

    for (file, index) in pkg.file_list:
        for origin, archive in security_package_sources:
            if (file.archive == archive and file.origin == origin):
                return True
    return False

# init apt and config
apt_pkg.init()

# open the apt cache
try:
    cache = apt_pkg.Cache(OpNullProgress())
except SystemError, e:
    sys.stderr.write("Error: Opening the cache (%s)" % e)
    sys.exit(-1)

# setup a DepCache instance to interact with the repo
depcache = apt_pkg.DepCache(cache)

# take into account apt policies
depcache.read_pinfile()

# initialise it
depcache.init()

# give up if packages are broken
if depcache.broken_count > 0:
    sys.stderr.write("Error: Broken packages exist.")
    sys.exit(-1)

# mark possible packages
try:
    # run distro-upgrade
    depcache.upgrade(True)
    # reset if packages get marked as deleted -> we don't want to break anything
    if depcache.del_count > 0:
        depcache.init()

    # then a standard upgrade
    depcache.upgrade()
except SystemError, e:
    sys.stderr.write("Error: Couldn't mark the upgrade (%s)" % e)
    sys.exit(-1)

# run around the packages
upgrades = 0
security_upgrades = 0
for pkg in cache.packages:
    candidate = depcache.get_candidate_ver(pkg)
    current = pkg.current_ver

    # skip packages not marked as upgraded/installed
    if not (depcache.marked_install(pkg) or depcache.marked_upgrade(pkg)):
        continue

    # increment the upgrade counter
    upgrades += 1

    # keep another count for security upgrades
    if is_security_upgrade(candidate):
        security_upgrades += 1

    # double check for security upgrades masked by another package
    for version in pkg.version_list:
        if (current and apt_pkg.version_compare(version.ver_str, current.ver_str) <= 0):
            continue
        if is_security_upgrade(version):
            security_upgrades += 1
            break

print "%d updates to install." % upgrades
print "%d are security updates." % security_upgrades
print "" # leave a trailing blank line
```

### Météo (FACULTATIF)

Vérifier ou installer **bc** et **jq** 

	sudo apt install bc jq

Copier le fichier **ansiweather-fr.sh** sous **/usr/local/bin**

```
#!/bin/sh

###############################################################################
#                                                                             #
# AnsiWeather 1.08                                                            #
# Copyright (c) 2013-2016, Frederic Cambus                                    #
# https://github.com/fcambus/ansiweather                                      #
#                                                                             #
# Created: 2013-08-29                                                         #
# Last Updated: 2016-07-26                                                    #
#                                                                             #
# AnsiWeather is released under the BSD 2-Clause license.                     #
# See LICENSE file for details.                                               #
# Traduction FR 2016-09-28                                                    #
###############################################################################



###[ Configuration options ]###################################################

LC_ALL=C; export LC_ALL

config_file=${ANSIWEATHERRC:-~/.ansiweatherrc}

get_config() {
	ret=""
	if [ -f "$config_file" ]
	then
		ret=$(grep "^$1:" "$config_file" | awk -F: '{print $2}')
	fi

	if [ "X$ret" = "X" ]
	then
		return 1
	else
		echo "$ret"
	fi
}

fetch_cmd=$(get_config "fetch_cmd" || echo "curl -sf")



###[ Parse the command line ]##################################################

# Get config options from command line flags
while getopts k:l:u:f:Fd:a:s:h option
do
	case "${option}"
	in
		l) location=${OPTARG};;
		u) units=${OPTARG};;
		f) forecast=${OPTARG};;
		F) forecast="5";;
		d) daylight=${OPTARG};;
		a) ansi=${OPTARG};;
		s) symbols=${OPTARG};;
		k) api_key=${OPTARG};;
		h) usage=true;;
	esac
done



###[ Display usage ]###########################################################

if [ "$usage" = true ]
then
	printf "%s\n" \
		"" \
		"AnsiWeather 1.08.01" \
		"Copyright (c) 2013-2016, Frederic Cambus" \
		"Traduction FR 2016" \
		"USAGE: ansiweather [options]" \
		"" \
		"Options :" \
		"" \
		"	-l Spécifier Ville ,Code Pays(2 car)" \
		"	-u Système de mesure utilisé (metric or imperial)" \
		"	-f Prévision météo (nombre de jour)" \
		"	-F Prévision sur les 5 jours à venir" \
		"	-d Basculement Jour/Nuit" \
		"	-a Toggle ANSI colors display" \
		"	-s Affichage des symboles" \
		"	-k Spécifier une clé API OpenWeatherMap" \
		"	-h Afficher l'aide" \
		"" \
		"EXEMPLE: ansiweather -l Cholet,FR -u metric -s true -f 5 -d true" \
		""
	exit
fi



###[ Check if bc and jq are installed ]########################################

jqpath=$(which jq)
if [ "$jqpath" = "" ]
then
	echo "ERREUR : Binaire jq introuvable"
	exit 69 # EX_UNAVAILABLE
fi

bcpath=$(which bc)
if [ "$bcpath" = "" ]
then
	echo "ERREUR : Binaire bc introuvable"
	exit 69 # EX_UNAVAILABLE
fi



###[ Set options that are not set from command line (Définir les options non définies en ligne de commande) ]##########################

# OpenWeatherMap API key
[ -z "$api_key" ] && api_key=$(get_config "api_key" || echo "85a4e3c55b73909f42c6a23ec35b7147")

# Location : example "Rzeszow,PL"
[ -z "$location" ] && location=$(get_config "location" || echo "Rzeszow,PL")

# System of Units : "metric" or "imperial"
[ -z "$units" ] && units=$(get_config "units" || echo "metric")

# Show forecast : How many days, example "5". "0" is standard output
[ -z "$forecast" ] && forecast=$(get_config "forecast" || echo 0)

# Show daylight : "true" or "false"
[ -z "$daylight" ] && daylight=$(get_config "daylight" || echo false)

# Display ANSI colors : "true" or "false"
[ -z "$ansi" ] && ansi=$(get_config "ansi" || echo true)

# Display symbols : "true" or "false" (requires an Unicode capable display)
[ -z "$symbols" ] && symbols=$(get_config "symbols" || echo true)

dateformat=$(get_config "dateformat" || echo "%a %b %d")
timeformat=$(get_config "timeformat" || echo "%b %d %r")



###[ Colors and characters ]###################################################

background=$(get_config "background" || echo "\033[44m")
text=$(get_config "text" || echo "\033[36;1m")
data=$(get_config "data" || echo "\033[33;1m")
delimiter=$(get_config "delimiter" || echo "\033[35m=>")
dashes=$(get_config "dashes" || echo "\033[34m-")



###[ Text Labels (Étiquettes de texte) ]#######################################

greeting_text=$(get_config "greeting_text" || echo "Météo actuelle à")
wind_text=$(get_config "wind_text" || echo "Vent")
humidity_text=$(get_config "humidity_text" || echo "Humidité")
pressure_text=$(get_config "pressure_text" || echo "Pression")
sunrise_text=$(get_config "sunrise_text" || echo "Lever")
sunset_text=$(get_config "sunset_text" || echo "Coucher")
forecast_text=$(get_config "forecast_text" || echo "prévision")



###[ Unicode Symbols for icons ]###############################################

sun=$(get_config "sun" || echo "\033[33;1m\xe2\x98\x80")
moon=$(get_config "moon" || echo "\033[36m\xe2\x98\xbd")
clouds=$(get_config "clouds" || echo "\033[37;1m\xe2\x98\x81")
rain=$(get_config "rain" || echo "\033[37;1m\xe2\x98\x94")
fog=$(get_config "fog" || echo "\033[37;1m\xe2\x96\x92")
mist=$(get_config "mist" || echo "\033[34m\xe2\x96\x91")
haze=$(get_config "haze" || echo "\033[33m\xe2\x96\x91")
snow=$(get_config "snow" || echo "\033[37;1m\xe2\x9d\x84")
thunderstorm=$(get_config "thunderstorm" || echo "\033[33;1m\xe2\x9a\xa1")



###[ Fetch Weather data ]######################################################

api_cmd=$([ "$forecast" != 0 ] && echo "forecast/daily" || echo "weather")

if [ "$location" -gt 0 ] 2> /dev/null
then
	# Location is all numeric
	weather=$($fetch_cmd "http://api.openweathermap.org/data/2.5/$api_cmd?id=$location&units=$units&appid=$api_key")
else
	# Location is a string
	location=$(echo "$location" | sed "s/ /_/g")
	#echo $fetch_cmd "http://api.openweathermap.org/data/2.5/$api_cmd?q=$location&units=$units&appid=$api_key"
	weather=$($fetch_cmd "http://api.openweathermap.org/data/2.5/$api_cmd?q=$location&units=$units&appid=$api_key")
fi

if [ -z "$weather" ]
then
	echo "ERREUR : données météo inaccessibles"
	exit 75 # EX_TEMPFAIL
fi

status_code=$(echo "$weather" | jq -r '.cod' 2>/dev/null)

if [ "$status_code" != 200 ]
then
	echo "ERREUR : données météo inaccessibles pour la localisation demandée"
	exit 69 # EX_UNAVAILABLE
fi



###[ Process Weather data ]####################################################

epoch_to_date() {
	if date -j -r "$1" +"%a %b %d" > /dev/null 2>&1; then
		# BSD
		ret=$(date -j -r "$1" +"$dateformat")
	else
		# GNU
		ret=$(date -d "@$1" +"$dateformat")
	fi
	echo "$ret"
}

if [ "$forecast" != 0 ]
then
	city=$(echo "$weather" | jq -r '.city.name')
	flength=$(echo "$weather" | jq '.list | length')
	forecast=$([ "$forecast" -gt "$flength" ] && echo "$flength" || echo "$forecast")
else
	city=$(echo "$weather" | jq -r '.name')
	temperature=$(echo "$weather" | jq '.main.temp' | xargs printf "%.0f")
	humidity=$(echo "$weather" | jq '.main.humidity')
	pressure=$(echo "$weather" | jq '.main.pressure')
	sky=$(echo "$weather" | jq -r '.weather[0].main')
	sunrise=$(echo "$weather" | jq '.sys.sunrise')
	sunset=$(echo "$weather" | jq '.sys.sunset')
	wind=$(echo "$weather" | jq '.wind.speed')
	azimuth=$(echo "$weather" | jq '.wind.deg')
fi



###[ Process Wind data (Données sur le Vent) ]##################################

#set -- N NNE NE ENE E ESE SE SSE S SSW SW WSW W WNW NW NNW
set -- N NNE NE ENE E ESE SE SSE S SSO SO OSO O ONO NO NNO

if [ "$forecast" = 0 ]
then
	shift "$(echo "scale=0; ($azimuth + 11.25)/22.5 % 16" | bc)"
	direction=$1
  	#echo "direction : $1"
fi



###[ Process Sunrise and Sunset data (Données Lever et Coucher du soleil) ]######

epoch_to_time() {
	if date -j -r "$1" +"%r" > /dev/null 2>&1; then
		# BSD
		ret=$(date -j -r "$1" +"$timeformat")
	else
		# GNU
		ret=$(date -d "@$1" +"$timeformat")
	fi
	echo "$ret"
}

if [ "$forecast" = 0 ]
then
	if [ -n "$sunrise" ]
	then
		sunrise_time=$(epoch_to_time "$sunrise")
	fi

	if [ -n "$sunset" ]
	then
		sunset_time=$(epoch_to_time "$sunset")
	fi
fi



###[ Set the period ]##########################################################

now=$(date +%s)

if [ "$forecast" != 0 ]
then
	period="none"
else
	if [ -z "$sunset" ] || [ -z "$sunrise" ]
	then
		period="day"
	elif [ "$now" -ge "$sunset" ] || [ "$now" -le "$sunrise" ]
	then
		period="night"
	else
		period="day"
	fi
fi



###[ Set the scale ]###########################################################

case $units in
	metric)
		scale="°C"
		speed_unit="m/s"
		pressure_unit="hPa"
		pressure=$(echo "$pressure" | xargs printf "%.0f")
		;;
	imperial)
		scale="°F"
		speed_unit="mph"
		pressure_unit="inHg"
		if [ "$forecast" = 0 ]
		then
			pressure=$(echo "$pressure*0.0295" | bc | xargs printf "%.2f")
		fi
		;;
esac



###[ Set icons ]###############################################################

get_icon() {
	case $1 in
		Clear)
			if [ $period = "night" ]
			then
				echo "$moon "
			else
				echo "$sun "
			fi
			;;
		Clouds)
			echo "$clouds "
			;;
		Rain)
			echo "$rain "
			;;
		Fog)
			echo "$fog "
			;;
		Mist)
			echo "$mist "
			;;
		Haze)
			echo "$haze "
			;;
		Snow)
			echo "$snow "
			;;
		Thunderstorm)
			echo "$thunderstorm "
			;;
	esac
}



###[ Display current Weather ]#################################################

if [ "$forecast" != 0 ]
then
	output="$background$text $city $forecast_text $text$delimiter "

	i=0
	while [ $i -lt "$forecast" ]
	do
		day=$(echo "$weather" | jq ".list[$i]")
		date=$(epoch_to_date "$(echo "$day" | jq -r '.dt')")
		low=$(echo "$day" | jq -r '.temp.min' | xargs printf "%.0f")
		high=$(echo "$day" | jq -r '.temp.max' | xargs printf "%.0f")

		icon=""
		if [ "$symbols" = true ]
		then
			sky=$(echo "$day" | jq -r '.weather[0].main')
			icon=$(get_icon "$sky")
		fi

date: 2018-11-23 00:00:00 +0100
last_modified_at: 2018-11-23
		if [ $i -lt $((forecast-1)) ]
		then
			output="$output$dashes "
		fi

		i=$((i + 1))
	done
else
	if [ "$symbols" = true ]
	then
		icon="$(get_icon "$sky")"
	fi
	output="$background$text $greeting_text $city $delimiter$data $temperature $scale $icon$dashes$text $wind_text $delimiter$data $wind $speed_unit $direction $dashes$text $humidity_text $delimiter$data $humidity %% $dashes$text $pressure_text $delimiter$data $pressure $pressure_unit "

	if [ "$daylight" = true ]
	then
		output="$output $dashes$text $sunrise_text $delimiter$data $sunrise_time $dashes$text $sunset_text $delimiter$data $sunset_time "
	fi
fi

if [ "$ansi" = true ]
then
	env printf "$output\033[0m\n"
else
	env printf "$output\n" | sed "s/$(printf '\033')\[[0-9;]*m//g"
fi
```

Rendre exécutable

	chmod +x /usr/local/bin/ansiweather-fr.sh

Remplacer Ville et Pays (Pays sur 2 caractères ,exemples : FR ,GB ,AT ,etc...)

	nano 98-meteo

```
#!/bin/bash
/usr/local/bin/ansiweather-fr.sh -l Ville,Pays -u metric -s true 

```

Rendre exécutable

	chmod +x 98-meteo

### 99-footer

	nano 99-footer

```
#!/bin/sh
#
#    99-footer - write the admin's footer to the MOTD
#    Copyright (c) 2013 Nick Charlton
#    Copyright (c) 2009-2010 Canonical Ltd.
#
#    Authors: Nick Charlton <hello@nickcharlton.net>
#             Dustin Kirkland <kirkland@canonical.com>
#
#    This program is free software; you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation; either version 2 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License along
#    with this program; if not, write to the Free Software Foundation, Inc.,
#    51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA.

[ -f /etc/motd.tail ] && cat /etc/motd.tail || true
```


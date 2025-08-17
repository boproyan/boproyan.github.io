+++
title = 'Python venv OVH API'
date = 2024-12-30 00:00:00 +0100
categories = ['ovh']
+++
*Les API disponibles sur https://api.ovh.com/ vous permettent d'acheter, gérer, mettre à jour et configurer des produits OVHcloud sans utiliser une interface graphique comme l'espace client.*

![](ovh-logo-a.png)

## OVH API

* [How to use service accounts to connect to OVHcloud APIs](https://help.ovhcloud.com/csm/en-api-service-account-connection?id=kb_article_view&sysparm_article=KB0059336)

### Préalables

1. Disposer d'un compte OVH
2. Créer un token pour accès API: <https://www.ovh.com/auth/api/createToken?GET=/*&PUT=/*&POST=/*&DELETE=/*>  
![](ovh-api01.png){:width="300"}


Exemple

```
curl -X GET "https://eu.api.ovh.com/v1/domain/icevps.xyz"  -H "accept: application/json" -H "authorization: Bearer eyJhbGciOiJFZERTQSIsImtpZCI6IkVGNThFMkUxMTFBODNCREFEMDE4OUUzMzZERTk3MDhFNjRDMDA4MDEiLCJraW5kIjoib2F1dGgyIiwidHlwIjoiSldUIn0.eyJBY2Nlc...ArEsT1X2_asA-dTNHvQTah5RTaSiZzDg" |jq
```

## OVH API Python

### Environnement virtuel python

Prérequis,venv pour votre version de python 

    sudo apt install python3-venv 

Créer un dossier `ovh_api`

```bash
sudo mkdir /home/leno/ovh_api
sudo chown $USER:$USER /home/leno/ovh_api
cd /home/leno/ovh_api
```

Pour créer un environnement, utilisez la commande `python -m venv <environment name>`
Créer un environnement pour l'application

    python3 -m venv ApiOvh

activer l'environnement virtuel  

    source ApiOvh/bin/activate

On arrive sur un prompt ![](apiovh01.png)  

Mettre à jour pip dans l'environnement

    python -m pip install --upgrade pip

![](apiovh03.png)  

Installer ApiOVH

    pip install ovh

### API - Domaine

[Get service information](https://eu.api.ovh.com/console/?section=%2Fdomain&branch=v1#get-/domain/zone/-zoneName-/serviceInfos)

Créer un script python pour lire les infos du domaine

    nano domain.py


```python
'''
First, install the latest release of Python wrapper: $ pip install ovh
'''
import json
import ovh
import sys
from datetime import datetime

# Instantiate an OVH Client.
# You can generate new credentials with full access to your account on
# the token creation page (https://api.ovh.com/createToken/index.cgi?GET=/*&PUT=/*&POST=/*&DELETE=/*)
client = ovh.Client(
	endpoint='ovh-eu',               # Endpoint of API OVH (List of available endpoints: https://github.com/ovh/python-ovh#2-configure-your-application)
	application_key='xxxxxxxxxx',    # Application Key
	application_secret='xxxxxxxxxx', # Application Secret
	consumer_key='xxxxxxxxxx',       # Consumer Key
)

#result = client.get("/domain")
# Pretty print
#print(json.dumps(result, indent=4))

for i in range(1, len(sys.argv)):
    #print('argument:', i, 'value:', sys.argv[i])
    domaine=str(sys.argv[i])
    result = client.get("/domain/" + domaine)
    # Pretty print
    #print(json.dumps(result, indent=4))
    resp = json.loads(json.dumps(result))
   
    # La date d'expiration (resp['expirationDate']) est au format ISO '2024-12-25T16:14:07Z'
    # Extraction des éléments
    dt_object = datetime.fromisoformat(resp['expirationDate'])
    print(f"{domaine} expire le {dt_object.day:02d}/{dt_object.month:02d}/{dt_object.year}")

```

Exécuter le script

    python domain.py xoyize.xyz cinay.eu xoyaz.xyz ouestline.xyz rnmkcy.eu yanfi.net icevps.xyz xoyize.net iceyan.xyz

![](apiovh04.png)  

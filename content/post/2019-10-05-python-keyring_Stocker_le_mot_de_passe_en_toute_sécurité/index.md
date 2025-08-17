+++
title = 'python-keyring Stocker le mot de passe en toute sécurité'
date = 2019-10-05 00:00:00 +0100
categories = python
+++
## python-keyring Stocker le mot de passe en toute sécurité

[Securely Store Password](https://github.com/sup-heliotrope/sup/wiki/Securely-Store-Password)  
Cette page décrit comment stocker le mot de passe d'une manière sécurisée et récupérer le mot de passe depuis offlineimap et msmtp.  
**Porte-clés d'accès depuis Python**

Installez Python et le module python-keyring pour votre système :

Archlinux: python2-keyring on AUR  
Debian / Ubuntu: `sudo apt install python-keyring`  
Fedora: python-keyring  

Vous pouvez maintenant stocker votre mot de passe en toute sécurité via Python :

```
$ python -c "import keyring; keyring.set_password('cinay', 'yannick', 'PASSWORD')"
# Test that the password is successfully stored:
$ python -c "import keyring; print(keyring.get_password('cinay', 'yannick'))"
PASSWORD
```


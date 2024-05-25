---
title: "DURCISSEMENT OS - Authentification"
description: "Scripts de durcissement et tests associ�s pour la plupart�"
pubDate: "May 25 2024"
heroImage: "/post_img.webp"
tags: ["durcissement"]
---


>>>> Durcissement AUTHENTIFICATION

1- Script

```
#!/bin/sh

# EXIGENCE.AUTH.200
# Les droits sur le fichier /etc/login.defs doivent limiter l’accès à l’administrateur.

chmod 0600 /etc/login.defs

exit $?

```

1- Test associ�

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# EXIGENCE.AUTH.200
# Les droits sur le fichier /etc/login.defs doivent limiter l’accès à l’administrateur.


tperms 600 root root /etc/login.defs

treport

```

2- Script


```
#!/bin/sh

# EXIGENCE.AUTH.220
# Les droits sur le fichier /etc/shadow doivent limiter l’accès en lecture et écriture à l’administrateur.

chmod 0600 /etc/shadow

exit $?

```

2.1 - Test associé ?


```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# EXIGENCE.AUTH.220
# Les droits sur le fichier /etc/shadow doivent limiter l’accès en lecture et écriture à l’administrateur.


tperms 600 root root /etc/shadow

treport

```

3- Script


```
#!/bin/sh

# EXIGENCE.AUTH.400
# Les comptes de services doivent être désactivés

set -e

listAcc="
        operator
        hacluster
        elasticsearch
        postgres
"

for user in ${listAcc}; do
        if id ${user} >/dev/null 2>/dev/null; then
                echo "Désactivation du compte ${user}"
                usermod -s /sbin/nologin ${user}
        fi
done

```

3.1 - Test associé 

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.400
# Les comptes de services doivent être désactivés

# Liste des comptes de service
listAcc=$(/bin/cat "${STAGE4_PATH}"/etc/passwd | awk -F ":" '$3 <1000 {print $1}')

for user in $listAcc; do
        tbegin "Vérification de la désactivation du compte $user"
        loginType=$(/bin/grep "^$user:" "${STAGE4_PATH}"/etc/passwd | awk -F ":" '{print $7}')
        if [ "$loginType" = "/bin/false" ]; then
                tpass
        elif [ "$loginType" = "/sbin/nologin" ]; then
                tpass
        elif [ "$loginType" = "/sbin/shutdown" ] && [ "$user" = "shutdown" ]; then
                tpass
        elif [ "$loginType" = "/sbin/halt" ] && [ "$user" = "halt" ]; then
                tpass
        elif [ "$loginType" = "/bin/sync" ] && [ "$user" = "sync" ]; then
                tpass
        else
                tfail "le compte de service $user n'est pas désactivé"
        fi
done

treport

```

4- Script 


```
#!/bin/sh

# EXIGENCE.AUTH.700
# Les droits sur le fichier /etc/securetty doivent limiter l’accès à
# l’administrateur et le fichier doit contenir au plus les entrées :
# console, vc/1 - 11, tty1 - tty11

chmod 0600 /etc/securetty

```
4.1 - Test associé 


```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# EXIGENCE.AUTH.700
# Les droits sur le fichier /etc/securetty doivent limiter l’accès à
# l’administrateur et le fichier doit contenir au plus les entrées :
# console, vc/1 - 11, tty1 - tty11


tperms 600 root root /etc/securetty

tbegin "Test de la désactivation du login root local"
if [ ! -f "${STAGE4_PATH}/etc/securetty" ]; then
        tfail "Le fichier /etc/securetty est absent"
else
        if [ -s "${STAGE4_PATH}/etc/securetty" ]; then
                tfail "Le fichier /etc/securetty n'est pas vide"
        else
                tpass
        fi
fi

treport

```

5- Script


```
#!/bin/sh

# EXIGENCE.AUTH.1400
# La valeur umask par défaut au niveau système doit être 0027

UMASKFILE="/etc/profile.d/umask.sh"

mkdir -p "/etc/profile.d"
touch ${UMASKFILE} && chmod 0644 ${UMASKFILE}
echo "umask 0027" > ${UMASKFILE}

exit $?

```

5.1- Test associé

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1400
# La valeur umask par défaut au niveau système doit être 0027
tbegin "Test non implémenté"
tfail


treport

```

6- Script

```
#!/bin/sh

# EXIGENCE.AUTH.1410
# La valeur umask du fichier /etc/login.defs doit être paramétrée à 0077

FILEDEF="/etc/login.defs"

currentUmask=$(grep ^UMASK ${FILEDEF} | awk '{ print $2}')

if [ $currentUmask -ne 0077 ]; then
        sed -i -e 's/^UMASK[[:blank:]]*[[:digit:]]\{3\}/UMASK    0077/' ${FILEDEF}
        [ $? -ne 0 ] && exit $?
fi

exit 0

```

6.1 - Test associé


```
#!/bin/sh

. ./tests/core.sh

tuse config

tinit

# EXIGENCE.AUTH.1410
# La valeur umask du fichier /etc/login.defs doit être paramétrée à 0077


tbegin "Vérification du umask"
cconftest UMASK 0077 /etc/login.defs
res=$?
if [ $res -eq 0 ]; then
        tpass
elif [ $res -eq 1 ]; then
        tfail "Le umask n'est pas positionné à \"0077\""
else
        tfail "Le umask n'existe pas"
fi

treport
```

>>>> AUTRES SCRIPTS DE TESTS DE DURCISSEMENT AUTHENTIFICATION

1- Test 100

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.100
# Les comptes utilisateurs doivent avoir une date d'expiration
tbegin "Test non implémenté"
tfail

treport
```

2. Test 210

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# EXIGENCE.AUTH.210
# Les droits sur le fichier /etc/passwd doivent limiter l’accès en écriture à l’administrateur.


tperms 644 root root /etc/passwd

treport
```

3. Test 210

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# EXIGENCE.AUTH.210
# Les droits sur le fichier /etc/passwd doivent limiter l’accès en écriture à l’administrateur.


tperms 644 root root /etc/passwd

treport
```

4. Test 230

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# EXIGENCE.AUTH.230
# Les droits sur le fichier /etc/group doivent limiter l’accès en écriture à l’administrateur.

tperms 644 root root /etc/group

treport
```

5. Test 240

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# EXIGENCE.AUTH.240
# Les droits sur le fichier /etc/gshadow doivent limiter l’accès en écriture à l’administrateur.


tperms 400 root root /etc/gshadow

treport
```

6. Test 500

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.500
# Chaque service doit posséder un compte distinct

# Liste des comptes de service
getent passwd | awk -F ':' '$3 < 1000 { print $1 " : " $7}'

# Vérification
tbegin "Test non implémenté"
tfail

treport
```

7. Test 600

```
#!/bin/sh

. ./tests/core.sh

tuse config

tinit

# EXIGENCE.AUTH.600
# Le compte root doit être désactivé, que ce soit en login local ou en login distant


tbegin "Test de la désactivation du login root en SSH"
cconftest PermitRootLogin no /etc/ssh/sshd_config
res=$?
if [ $res -eq 0 ]; then
        tpass
elif [ $res -eq 1 ]; then
        tfail "PermitRootLogin n'est pas positionné à \"no\""
else
        tfail "PermitRootLogin n'existe pas"
fi

tbegin "Test de la désactivation du login root local"
if [ ! -f "${STAGE4_PATH}/etc/securetty" ]; then
        tfail "Le fichier /etc/securetty est absent"
else
        if [ -s "${STAGE4_PATH}/etc/securetty" ]; then
                tfail "Le fichier /etc/securetty n'est pas vide"
        else
                tpass
        fi
fi

treport
```


8. Test  800

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.800
# La session utilisateur doit se verrouiller (mode graphique) ou se
# fermer (mode console) au bout d'un temps d'inactivité
tbegin "Test non implémenté"
tfail

treport
```

9. Test 900

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.900
# Un utilisateur accédant au système doit voir un message d'avertisseme
# d'accès au système avant, pendant, et après le lo
tbegin "Test non implément�
tfail

treport
```

10. Test 1000

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1000
# Les services utilisant PAM doivent correspondre à ceux listés dans
# la documentation d’architecture
tbegin "Test non implémenté"
tfail


treport
```

11. Test 1100

```
#!/bin/sh

. ./tests/core.sh

tuse config

tinit

# EXIGENCE.AUTH.1100
# Tout le trafic vers le service d'authentification doit être chiffré via le protocole TLS

tbegin "Test du chiffrement du service d'authentification"
if cconfgrep "ldaps" /etc/openldap/ldap.conf; then
        tpass
else
        tfail "Pas de chiffrement TLS pour le service d'authentification"
fi

treport
```

12. Test 1200

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1200
# Les mots de passe des comptes utilisateurs doivent être hashés en mode
# SHA-512 (mode 6 dans /etc/shadow)


# Liste des utilisateurs ayant le droit de se logger sur la machine
list_user=$(/bin/cat ${STAGE4_PATH}/etc/passwd | /bin/grep 'sh$' | /bin/awk -F":" '{print $1}')

# Test du hash des mots de passe
for user in $list_user; do
        tbegin "Test du hash des mots de passe du compte $user"
        ret=$(/bin/grep "^$user:" ${STAGE4_PATH}/etc/shadow | awk -F ":" '{print $2}')
        if [ "$ret" = "!" ] || [ "$ret" = "*" ]; then
                tpass
        else
                if /bin/grep "^$user:" ${STAGE4_PATH}/etc/shadow | /bin/grep -q '\$6\$'; then
                        tpass
                else
                        tfail "L'utilisateur $user ne possède pas un mot de passe hasché en SHA512"
                fi
        fi
done

treport
```

13. Test 1300

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1300
# Le listage des utilisateurs de l'annuaire centralisé par un utilisateur
# du système doit être désactivé
tbegin "Test non implémenté"
tfail

treport
```

12. Test 1500

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1500
# Les mots de passe des comptes utilisateurs doivent respecter une politique
# de mot de passe avec les paramètres minimaux décrits en annexe 6.5 ci-dessous
tbegin "Test non implémenté"
tfail


treport
```

13. Test 1600

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1600
# Les mots de passe usine d'administration doivent être changés au premier
# login ou à l'installation, en respectant la politique de mot de passe
tbegin "Test non implémenté"
tfail

treport
```

14. Test 1610

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1610
# Chaque machine doit avoir un mot de passe d’administrateur local différent
tbegin "Test non implémenté"
tfail


treport
```

15. Test 1700

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1700
# Les mots de passe doivent avoir une longueur strictement supérieure à 0
tbegin "Test non implémenté"
tfail


treport
```


16. Test 

```
#!/bin/sh

. ./tests/core.sh

tinit

# EXIGENCE.AUTH.1800
# L'uid des utilisateurs du système doit être différent de 0 (à part root)


# Liste des utilisateurs ayant le droit de se logger sur la machine
list_user=$(/bin/cat ${STAGE4_PATH}/etc/passwd | /bin/grep 'sh$' | /bin/awk -F":" '{print $1}')

# Vérification du uid 0
for user in $list_user; do
        tbegin "Vérification de l'uid de l'utilisateur $user"
        uid=$(/bin/grep $user ${STAGE4_PATH}/etc/passwd | /bin/awk -F":" '{print $3}')

        if [ "$uid" = 0 ]; then
                # En règle général, ce cas n'arrive jamais car le login root est désactivé
                if [ "$user" = "root" ]; then
                        tpass
                else
                        tfail "L'utilisateur $user possède l'uid 0"
                fi
        else
                tpass
        fi
done

treport
```

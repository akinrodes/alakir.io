---
title: "DURCISSEMENT OS"
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

#============================================
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
#============================================
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
#============================================
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
#============================================
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
#============================================
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

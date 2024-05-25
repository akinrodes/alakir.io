---
title: "DURCISSEMENT OS - Cron"
description: "Scripts de durcissement et tests associ�s pour la plupart�"
pubDate: "May 25 2024"
heroImage: "/post_img.webp"
tags: ["durcissement"]
---


>>>> DURCISSEMENT CRON 
 
1- Script

```
#!/bin/sh


# SEDUCS.SYS.CRON.300
# Les droits sur le fichier /etc/crontab doivent restreindre
# l’accès à l’administrateur


chmod 0600 /etc/crontab
[ $? -ne 0 ] && exit 1

exit 0
```

1.1 - Test associé

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# SEDUCS.SYS.CRON.300
# Les droits sur le fichier /etc/crontab doivent restreindre
# l’accès à l’administrateur


if [ -f ${STAGE4_PATH}/etc/crontab ]; then
        tperms 600 root root /etc/crontab
else
        tbegin "Vérification des droits du fichier /etc/crontab"
        tabort "Le fichier /etc/crontab est absent"
fi

treport
```

2- Script

```
#!/bin/sh

# SEDUCS.SYS.CRON.400
# Les droits sur les fichiers de type « cron.xxxly » doivent restreindre
# laccèsadministrateur

CRONDIRS="
/etc/cron.hourly
/etc/cron.daily
/etc/cron.weekly
/etc/cron.monthly
"


for file in $(ls /etc/cron.d); do
        chmod 0600 /etc/cron.d/${file}
done

chmod 0700 /etc/cron.d
[ $? -ne 0 ] && exit 1

for crondir in ${CRONDIRS}; do
        chmod 0700 $crondir
        [ $? -ne 0 ] && exit 1
done

exit 0
```

2.1 - Test associé ?

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# SEDUCS.SYS.CRON.400
# Les droits sur les fichiers de type « cron.xxxly » doivent restreindre
# l’accès à l’administrateur


if [ -d ${STAGE4_PATH}/etc/cron.d ]; then
        tperms 700 root root /etc/cron.d
else
        tbegin "Vérification des droits du dossier /etc/cron.d"
        tabort "Le répertoire /etc/cron.d est absent"
fi

listCronfiles=$(ls ${STAGE4_PATH}/etc/cron.d/*)
for file in $listCronfiles; do
        tperms 600 root root /etc/cron.d/$(basename $file)
done

listCronlyfiles=$(find ${STAGE4_PATH}/etc/cron.*ly -not -path '*/\.*')
for file in $listCronlyfiles; do
        tperms 700 root root ${file/${STAGE4_PATH}/}
done

treport
```

3- Script

```
#!/bin/sh

# SEDUCS.SYS.CRON.500
# Les droits sur le répertoire /var/spool/cron doivent restreindre
# l’accès à l’administrateur

chmod 0700 /var/spool/cron
[ $? -ne 0 ] && exit 1

exit 0
```
   
3.1 - Test associé 

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# SEDUCS.SYS.CRON.500
# Les droits sur le répertoire /var/spool/cron doivent restreindre
# l’accès à l’administrateur


if [ -e  ${STAGE4_PATH}/var/spool/cron ]; then
        tperms 700 root root /var/spool/cron
else
        tbegin "Vérification des droits du répertoire /var/spool/cron"
        tabort "Le répertoire /var/spool/cron est absent"
fi

treport
```


>>>> Autres scripts de tests de DURCISSEMENT CRON 
 
1- Test 100

```
#!/bin/sh

. ./tests/core.sh

tinit

# SEDUCS.SYS.CRON.100
# L'accès à crontab et à at doit être restreint au compte root


tbegin "Vérification de l'absence du fichier /etc/cron.deny"
if [ -f ${STAGE4_PATH}/etc/cron.deny ]; then
        tfail "Le fichier /etc/cron.deny est présent"
else
        tpass
fi

tbegin "Vérification de l'absence du fichier /etc/at.deny"
if [ -f ${STAGE4_PATH}/etc/at.deny ]; then
        tfail "Le fichier /etc/at.deny est présent"
else
        tpass
fi

tbegin "Vérification du contenu du fichier /etc/cron.allow"
if grep -vq root ${STAGE4_PATH}/etc/cron.allow; then
        tfail "Le fichier /etc/cron.allow d'autres entrées que root"
else
        tpass
fi

if [ -f ${STAGE4_PATH}/usr/bin/at ]; then
        tbegin "Vérification du contenu du fichier /etc/at.allow"
        if grep -vq root ${STAGE4_PATH}/etc/at.allow; then
                tfail "Le fichier /etc/at.allow d'autres entrées que root"
        else
                tpass
        fi
fi

treport
```

2. Test 200

```
#!/bin/sh

. ./tests/core.sh

tuse file-tests

tinit

# SEDUCS.SYS.CRON.200
# Les droits sur les fichiers cron.allow et at.allow doivent restreindre
# l’accès en lecture au compte administrateur.


if [ -f ${STAGE4_PATH}/etc/cron.allow ]; then
        tperms 400 root root /etc/cron.allow
else
        if [ -x ${STAGE4_PATH}/usr/sbin/cron ]; then
                tbegin "Vérification des droits du fichier /etc/cron.allow"
                tabort "Le fichier /etc/cron.allow est absent"
        fi
fi

if [ -f ${STAGE4_PATH}/etc/at.allow ]; then
        tperms 400 root root /etc/at.allow
else
        if [ -x ${STAGE4_PATH}/usr/bin/at ]; then
                tbegin "Vérification des droits du fichier /etc/at.allow"
                tabort "Le fichier /etc/at.allow est absent"
        fi
fi

treport
```

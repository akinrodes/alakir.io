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
 1 #!/bin/sh
 2
 3
 4 # SEDUCS.SYS.CRON.300
 5 # Les droits sur le fichier /etc/crontab doivent restreindre
 6 # l’accès à l’administrateur
 7
 8
 9 chmod 0600 /etc/crontab
10 [ $? -ne 0 ] && exit 1
11
12 exit 0
   ```

1.1 - Test associé

   ```
 1 #!/bin/sh
 2
 3 . ./tests/core.sh
 4
 5 tuse file-tests
 6
 7 tinit
 8
 9 # SEDUCS.SYS.CRON.300
10 # Les droits sur le fichier /etc/crontab doivent restreindre
11 # l’accès à l’administrateur
12
13
14 if [ -f ${STAGE4_PATH}/etc/crontab ]; then
15         tperms 600 root root /etc/crontab
16 else
17         tbegin "Vérification des droits du fichier /etc/crontab"
18         tabort "Le fichier /etc/crontab est absent"
19 fi
20
21 treport
   ```

#============================================
2- Script

   ```
 1 #!/bin/sh
 2
 3 # SEDUCS.SYS.CRON.400
 4 # Les droits sur les fichiers de type « cron.xxxly » doivent restreindre
 5 # l’accès à l’administrateur
 6
 7 CRONDIRS="
 8 /etc/cron.hourly
 9 /etc/cron.daily
10 /etc/cron.weekly
11 /etc/cron.monthly
12 "
13
14
15 for file in $(ls /etc/cron.d); do
16         chmod 0600 /etc/cron.d/${file}
17 done
18
19 chmod 0700 /etc/cron.d
20 [ $? -ne 0 ] && exit 1
21
22 for crondir in ${CRONDIRS}; do
23         chmod 0700 $crondir
24         [ $? -ne 0 ] && exit 1
25 done
26
27 exit 0
   ```

2.1 - Test associé ?

   ```
 1 #!/bin/sh
 2
 3 . ./tests/core.sh
 4
 5 tuse file-tests
 6
 7 tinit
 8
 9 # SEDUCS.SYS.CRON.400
10 # Les droits sur les fichiers de type « cron.xxxly » doivent restreindre
11 # l’accès à l’administrateur
12
13
14 if [ -d ${STAGE4_PATH}/etc/cron.d ]; then
15         tperms 700 root root /etc/cron.d
16 else
17         tbegin "Vérification des droits du dossier /etc/cron.d"
18         tabort "Le répertoire /etc/cron.d est absent"
19 fi
20
21 listCronfiles=$(ls ${STAGE4_PATH}/etc/cron.d/*)
22 for file in $listCronfiles; do
23         tperms 600 root root /etc/cron.d/$(basename $file)
24 done
25
26 listCronlyfiles=$(find ${STAGE4_PATH}/etc/cron.*ly -not -path '*/\.*')
27 for file in $listCronlyfiles; do
28         tperms 700 root root ${file/${STAGE4_PATH}/}
29 done
30
31 treport
   ```

#============================================
3- Script

   ```
 1 #!/bin/sh
 2
 3 # SEDUCS.SYS.CRON.500
 4 # Les droits sur le répertoire /var/spool/cron doivent restreindre
 5 # l’accès à l’administrateur
 6
 7 chmod 0700 /var/spool/cron
 8 [ $? -ne 0 ] && exit 1
 9
10 exit 0
   ```
   
3.1 - Test associé 

   ```
 1 #!/bin/sh
 2
 3 . ./tests/core.sh
 4
 5 tuse file-tests
 6
 7 tinit
 8
 9 # SEDUCS.SYS.CRON.500
10 # Les droits sur le répertoire /var/spool/cron doivent restreindre
11 # l’accès à l’administrateur
12
13
14 if [ -e  ${STAGE4_PATH}/var/spool/cron ]; then
15         tperms 700 root root /var/spool/cron
16 else
17         tbegin "Vérification des droits du répertoire /var/spool/cron"
18         tabort "Le répertoire /var/spool/cron est absent"
19 fi
20
21 treport
   ```


>>>> Autres scripts de tests de DURCISSEMENT CRON 
 
1- Test 100

   ```
 1 #!/bin/sh
 2
 3 . ./tests/core.sh
 4
 5 tinit
 6
 7 # SEDUCS.SYS.CRON.100
 8 # L'accès à crontab et à at doit être restreint au compte root
 9
10
11 tbegin "Vérification de l'absence du fichier /etc/cron.deny"
12 if [ -f ${STAGE4_PATH}/etc/cron.deny ]; then
13         tfail "Le fichier /etc/cron.deny est présent"
14 else
15         tpass
16 fi
17
18 tbegin "Vérification de l'absence du fichier /etc/at.deny"
19 if [ -f ${STAGE4_PATH}/etc/at.deny ]; then
20         tfail "Le fichier /etc/at.deny est présent"
21 else
22         tpass
23 fi
24
25 tbegin "Vérification du contenu du fichier /etc/cron.allow"
26 if grep -vq root ${STAGE4_PATH}/etc/cron.allow; then
27         tfail "Le fichier /etc/cron.allow d'autres entrées que root"
28 else
29         tpass
30 fi
31
32 if [ -f ${STAGE4_PATH}/usr/bin/at ]; then
33         tbegin "Vérification du contenu du fichier /etc/at.allow"
34         if grep -vq root ${STAGE4_PATH}/etc/at.allow; then
35                 tfail "Le fichier /etc/at.allow d'autres entrées que root"
36         else
37                 tpass
38         fi
39 fi
40
41 treport
   ```

2. Test 200

   ```
 1 #!/bin/sh
 2
 3 . ./tests/core.sh
 4
 5 tuse file-tests
 6
 7 tinit
 8
 9 # SEDUCS.SYS.CRON.200
10 # Les droits sur les fichiers cron.allow et at.allow doivent restreindre
11 # l’accès en lecture au compte administrateur.
12
13
14 if [ -f ${STAGE4_PATH}/etc/cron.allow ]; then
15         tperms 400 root root /etc/cron.allow
16 else
17         if [ -x ${STAGE4_PATH}/usr/sbin/cron ]; then
18                 tbegin "Vérification des droits du fichier /etc/cron.allow"
19                 tabort "Le fichier /etc/cron.allow est absent"
20         fi
21 fi
22
23 if [ -f ${STAGE4_PATH}/etc/at.allow ]; then
24         tperms 400 root root /etc/at.allow
25 else
26         if [ -x ${STAGE4_PATH}/usr/bin/at ]; then
27                 tbegin "Vérification des droits du fichier /etc/at.allow"
28                 tabort "Le fichier /etc/at.allow est absent"
29         fi
30 fi
31
32 treport
   ```

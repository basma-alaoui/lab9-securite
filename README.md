          Philosophie et objectifs du laboratoire
Plutôt que d’exploiter un root pour prendre le contrôle, cette mise en situation inverse la perspective : on observe comment une application protégée détecte un environnement rooté, puis on utilise des techniques d’instrumentation dynamique pour faire échouer silencieusement ces détections. L’apprenant apprend à manipuler Frida et Medusa sur Uncrackable Level 3, une application volontairement vulnérable. Les compétences visées incluent l’identification des points de contrôle Java et natifs, l’injection de hooks, l’automatisation du bypass et le diagnostic des échecs.

           Mise en place technique de l’environnement
Avant toute opération, on s’assure que Frida est installé sur le poste de travail, que les platform-tools (ADB) sont disponibles, et qu’un émulateur Android (ou un appareil physique rooté autorisé) est connecté. On vérifie l’architecture CPU de l’appareil (adb shell getprop ro.product.cpu.abi) afin de télécharger le frida-server adapté. Le binaire est poussé dans /data/local/tmp/, rendu exécutable (chmod 755), puis exécuté avec l’option -l 0.0.0.0 pour écouter sur toutes les interfaces. Les redirections de ports (adb forward tcp:27042 tcp:27042 et 27043) permettent à Frida de communiquer. La commande frida-ps -Uai confirme la bonne connexion.

           Mécanismes de détection root et anti-Frida
Les applications peuvent utiliser des vérifications côté Java : présence du tag test-keys dans Build.TAGS, existence des binaires su ou busybox via File.exists() ou Runtime.exec(), et même des librairies comme RootBeer. Côté natif, les appels système open(), access(), stat() ou la lecture de /proc/mounts trahissent souvent un environnement rooté. Enfin, certaines applications implémentent des contre-mesures anti-Frida : scrutation des ports (27042 par défaut), recherche de chaînes “frida” dans les processus ou détection de débogage. Ce laboratoire montre comment les surmonter.

                  Phase initiale – constat du blocage
Lorsque l’on lance Uncrackable Level 3 sur un appareil rooté (émulateur userdebug ou appareil physique rooté), l’application affiche un message : “Rooting or tampering detected. This is unacceptable. The app is now going to exit.” Puis elle se ferme immédiatement. Aucune interaction n’est possible.

Injection du bypass via Medusa
Medusa simplifie l’utilisation de modules Frida préconfigurés. La commande exécutée est :
python medusa.py --usb --spawn com.example.uncrackable3 --module root-bypass.
Medusa se connecte à l’appareil, démarre l’application en mode spawn, et injecte un ensemble de hooks. Parmi eux :

Remplacement de Build.TAGS par la valeur "release-keys" (au lieu de "test-keys").

Interception de File.exists() pour tout chemin contenant su, busybox, xbin, etc., avec retour forcé à false.

Blocage de Runtime.exec() sur les commandes suspectes (su, busybox, which su).
Le résultat est que l’application ne détecte plus aucun indicateur de root et poursuit son exécution normalement. Les logs Frida permettent de visualiser chaque vérification interceptée.

                 Gestion des cas complexes – code natif
Si l’application comporte des contrôles en C/C++ (via JNI), les hooks Java seuls sont insuffisants. Medusa propose alors des Native Hooks qui ciblent les fonctions libc comme fopen(), access() ou stat(). On active cette option pour intercepter les appels système au niveau natif. Une autre cause fréquente d’échec est un décalage de version entre le client Frida (sur PC) et frida-server (sur l’appareil) : les deux numéros doivent être rigoureusement identiques (frida --version).

                 cadre légal et bonnes pratiques
L’ensemble des manipulations décrites n’est autorisé que dans un environnement contrôlé et avec l’accord explicite du propriétaire de l’application (ici, un APK pédagogique). Il est formellement interdit d’utiliser ces techniques sur des applications tierces réelles, des applications protégées par des droits d’auteur ou dans un but malveillant. Le laboratoire se veut une introduction défensive aux tests d’intrusion mobiles, en accord avec les recommandations OWASP MASVS et MASTG. Les captures d’écran illustrent les étapes clés : détection root initiale, console Frida/Medusa, bypass confirmé et interface de l’application fonctionnant sur l’émulateur.

# Introduction

L’environnement Arduino est limité à **setup()** et **loop()** et il ne supporte pas le multi-tâches. Or, la plupart des systèmes d’exploitation permettent à plusieurs programmes ou threads de s’exécuter en même temps. En réalité, chaque processeur ne peut exécuter qu’un seul programme en un temps donné. Aussi, une partie du système d’exploitation est composé de l’ordonnanceur (scheduler), responsable de l’ordonnancement (ordre des tâches), et donc en charge de décider quel programme doit s’exécuter. Cela donne l’illusion d’une exécution simultanée en switchant rapidement d’un programme à l’autre.

### Introduction à FreeRTOS

**FreeRTOS** est un système d’exploitation temps réel (Real Time Operating System) open source. Il est parmi les plus utilisé parmi les systèmes d’exploitation temps réel et disponible gratuitement sous licence GPL.

L’unité d’exécution contrôlée par FreeRTOS est une tâche. Le nombre de tâches exécutées simultanément et leur priorité ne sont limités que par le matériel. L'ordonnancement est un système de file d'attente basé sur les Sémaphores et les Mutex (explications plus loin dans le cours). Il fonctionne selon le modèle Round-Robin avec gestion des priorités. Conçu pour être très compact, il n'est composé que de quelques fichiers en langage C, et n'implémente aucun driver matériel.

Les domaines d'applications sont assez larges, car les principaux avantages de FreeRTOS sont l’exécution temps réel, un code source ouvert et une taille très faible. Il est donc utilisé principalement pour les systèmes embarqués qui ont des contraintes d'espace pour le code, mais aussi pour des systèmes de traitement vidéo et des applications réseau qui ont des contraintes "temps réel". Bien évidement parmi les applications les plus récentes et le plus exigeantes par rapport à la gestion des activités en temps réel sont les architectures d’objets connectés dans l’IoT (Internet of Things).

### L'ordonnanceur

L’ordonnanceur (scheduler) des tâches a pour but principal de décider parmi les tâches qui sont dans l'état prêt, laquelle exécuter. Pour faire ce choix, l'ordonnanceur de FreeRTOS se base uniquement sur la priorité des tâches. Les tâches en FreeRTOS se voient assigner à leur création, un niveau de priorité représenté par un nombre entier. Le niveau le plus bas vaut zéro et il doit être strictement réservé pour la tâche **Idle**. Plusieurs tâches peuvent appartenir à un même niveau de priorité. Dans FreeRTOS Il n'y a aucun mécanisme automatique de gestion des priorités (cas du système Linux). La priorité d'une tâche ne pourra être changée qu'à la demande explicite du développeur. Les tâches sont de simples fonctions qui généralement, mais pas exclusivement, s’exécutent en boucle infinie.

Dans le cas d'un microcontrôleur possédant un seul cœur (AVR, ARM-Cortex4, ESP12, ...), il y aura à tout moment une seule tâche en exécution. L’ordonnanceur garantira toujours que la tâche de plus haute priorité pouvant s’exécuter sera sélectionnée pour entrer dans l'état d’exécution. Si deux tâches partagent le même niveau de priorité et sont toutes les deux capables de s'exécuter, alors les deux tâches s'exécuteront en alternance par rapport aux **réveils de l’ordonnanceur (tick)**.

Afin de choisir la tâche à exécuter, l'ordonnanceur doit lui-même s'exécuter et préempter la tâche en état d'exécution. Afin d'assurer le réveil de l'ordonnanceur, FreeRTOS définit une interruption périodique nommé la "**tickinterrupt**". Cette interruption s'exécute infiniment selon une certaine fréquence qui est définie dans le fichier FreeRTOSConfig.h par la constante :

configTICK\_RATE\_HZ               // tick frequency interrupt in Hz

Cette constante décrit alors la période de temps allouée au minimum pour chaque tâche ou expliqué autrement, l'intervalle séparant deux réveils de l’ordonnanceur.

### La tâche IDLE

Un microcontrôleur doit toujours avoir quelque chose à exécuter. En d'autres termes, il doit toujours y avoir une tâche en exécution. FreeRTOS gère cette situation en définissant la tâche **IDLE** qui est créée au lancement de l’ordonnanceur. La plus petite priorité du noyau est attribuée à cette tâche. Malgré cela, la tâche **IDLE** peut avoir plusieurs fonctions à remplir dont :

* Libérer l'espace occupé par une tâche supprimée
* Placer le microcontrôleur en veille afin d'économiser l'énergie du système lorsque aucune tâche applicative n'est en exécution.
* Mesurer le taux d'utilisation du processeur.

### Les tâches sous FreeRTOS

Les tâches sous FreeRTOS peuvent exister sous 5 états : **supprimé**, **suspendu**, **prêt**, **bloqué** ou **en exécution**.

![](<../.gitbook/assets/image (3).png>)

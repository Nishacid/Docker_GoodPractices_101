# Hardening d'un container docker

## 1. Docker Daemon Exposé

Vous devez créer un utilisateur spécifique pour l'utilisation de docker, sans aucune autre permission, et **NE PAS UTILISER** cet utilisateur pour autre chose que docker.

Ne donnez pas le groupe **docker** à n'importe quel utilisateur sauf à l'utilisateur spécifique.

Même chose dans un conteneur si vous utilisez Docker à l'intérieur.

***

## 2. Configurez l'utilisateur à l'intérieur du conteneur 

Par défaut, l'utilisateur créé par docker est `root`, mais si vous installez un serveur web, le service sera exécuté en tant que root. Imaginez qu'un attaquant réussisse à accéder à votre conteneur, il aura les droits root à l'intérieur.
Donc n'oubliez pas les bonnes pratiques à l'extérieur des conteneurs à comme l'intérieur !

***

## 3. Docker Capabilities et Privileges 

Chaque conteneur doit être démarré sans aucune **capabilities** avec l'option **--cap-drop=ALL**.

Si un conteneur doit être configuré avec une **capability** absolument nécessaire, il doit être configuré de la sorte : 
> **--cap-drop=ALL** --cap-add{"capability"}

Voici une liste non exhaustive des **capabilities** sensibles parmi les 35 disponibles. : 

    - CAP_AUDIT_CONTROL
    - CAP_CHOWN
    - CAP_DAC_OVERRIDE
    - CAP_FOWNER
    - CAP_FSETID
    - CAP_IPC_OWNER
    - CAP_MKNOD
    - CAP_SETFCAP
    - CAP_SETPCAP
    - CAP_SETUID
    - CAP_SYS_ADMIN
    - CAP_SYS_CHROOT
    - CAP_SYS_BOOT
    - CAP_SYS_MODULE
    - CAP_SYS_PTRACE
    - CAP_SYS_RAWIO
    - CAP_SYS_TTY_CONFIG  


## 3.1 Isolez les systèmes de fichiers sensibles de l'hôte

Chaque conteneur doit être démarré sans partager les systèmes de fichiers sensibles de l'hôte : les systèmes de fichiers `root (/)`, `device (/dev)`, `process (/proc)` ou `virtual (/sys)`. 
Par conséquent, l'option **--privileged** ne doit PAS être utilisée.

***

## 4. Restreindre l'accès aux périphériques hôtes

Si un conteneur doit être démarré avec un périphérique hôte, ce dernier doit être ajouté au conteneur avec l'option **--device** et les options supplémentaires `rwm` spécifiées pour limiter l'accès au minimum nécessaire.

***

## 5. Réseau 

## 5.1 Désactivez la connexion du conteneur au docker0 network bridge

Le service Docker doit être configuré pour démarrer un conteneur sans le connecter, par défaut, au `network bridge`, `docker0`, avec l'option **--bridge=none**.

## 5.2 Isolez l'interface réseau de l'hôte

Chaque conteneur doit être démarré sans partager l'interface réseau de l'hôte. L'option **--network host** ne doit PAS être utilisée.

## 5.3 Créez un réseau dédié pour chaque connexion

Si le conteneur doit `exposer` un port sur l'interface réseau de l'hôte ou doit être connecté à un ou plusieurs autres conteneurs, un réseau dédié doit être créé pour chaque connexion.

## 6. Namespaces 

Containers utilise 3 composants du **Kernel Linux** pour isoler les conteneurs les uns des autres : 

- Namespaces
- Cgroups
- OverlayFS

Les Namespaces séparent essentiellement les ressources du système, telles que les processus, les fichiers et la mémoire, des autres Namespaces.

Chaque processus tournant sous Linux se verra attribuer deux choses :

- Un **namespace**
- Un **process identifier** (PID)

## 6.1 Dédier les namespaces, PID, IPC et UTS à chaque conteneur.

Chaque conteneur doit être démarré avec namespaces, PID, IPC et UTS séparés de l'hôte et des autres conteneurs et donc sans les options **--pid**, **--ipc** et **--uts**.

## 6.2 Dédier un USER ID namespaces pour chaque conteneur

Le service Docker doit être configuré pour démarrer tous les conteneurs avec un `USER ID` distinct de celui de l'hôte et des autres conteneurs avec l'option **--usernsremap**. Cette configuration doit être accompagnée d'une interdiction de créer de nouveaux namespaces `USERID` à l'intérieur du conteneur, par exemple avec le profil `Seccomp` par défaut qui définit une politique de sécurité pour désactiver cette fonctionnalité à l'intérieur du conteneur.

## 6.3 Restreindre la création des namespaces USER ID à l'utilisateur root

Sur les systèmes Linux basés sur Debian, la configuration système `kernel.unprivileged_userns_clone` doit être égale à `0` (valeur par défaut) pour restreindre la création d'un espace `USER ID` à l'utilisateur root (UID=0).

## 6.4 Restreindre le partage du namespace USER ID de l'hôte

Lorsqu'un conteneur partage le `USER ID` avec l'hôte au démarrage, et que le service Docker est configuré pour démarrer tous les conteneurs avec un `USER ID` distinct de l'hôte, il est possible d'utiliser l'option **--userns=host** lors du démarrage du conteneur. Dans ce cas, il faut prendre en compte le risque que l'utilisateur root de l'hôte (UID=0) soit compromis par le conteneur.

***

## 7. Restricting access to resources

## 7.1 Déterminez des Control Groups pour chaque conteneur

Chaque conteneur doit être démarré avec des `Control Groups` séparés de l'hôte et donc sans l'option **--cgroup-parent**.

## 7.2 Limitez l'utilisation de la mémoire de l'hôte pour chaque conteneur

Chaque conteneur doit être démarré avec une limite d'utilisation maximale de la mémoire de l'hôte, avec l'option **--memory**, et une limite d'utilisation maximale de la mémoire de l'hôte `SWAP` avec l'option **--memory-swap**.

## 7.3 Limitez l'utilisation du CPU de l'hôte pour chaque conteneur

Chaque conteneur doit être démarré avec une limite d'utilisation maximale du `host CPU` avec l'option **--cpus**, ou avec les options **--cpu-period** et **--cpu-quota**.

## 7.4 Restreindre l'accès en lecture au système de fichiers root de chaque conteneur

Chaque conteneur doit être démarré avec son `root file system` en `read-only` avec l'option **--read-only**.

## 7.5 Limiter l'écriture de l'espace de stockage de chaque conteneur

Un conteneur peut être démarré avec une limite maximale d'utilisation de l'espace disque de l'hôte pour son système de fichiers root, en lecture et en écriture avec l'option **--storage-opt**.

# 7.6 Limiter l'écriture dans l'espace de stockage de tous les conteneurs

Un conteneur peut être démarré avec son `root file system` pour `read and write`, tant que la zone de stockage locale de Docker, par défaut `/var/lib/docker/`, est une partition dédiée séparée des partitions de l'hôte.
Dans ce cas, la menace d'un `déni de service` d'un conteneur à un autre est à considérer.


***

## 8. Logs d'un container

## 8.1 Exportation des logs avec Docker

Le service Docker doit être configuré pour collecter les journaux d'événements de l'application conteneurisée et les exporter vers un serveur centralisé avec l'option **--log-driver=\<logging_driver>**.

## 8.2 Exportation des logs depuis l'intérieur du conteneur

L'exportation des logs de l'application conteneurisée peut être mise en œuvre à l'intérieur du conteneur par une solution distincte de Docker (STDERR ou STDOUT).

***

## 9. Amélioration de la sécurité des conteneurs

## 9.1 Seccomp

Secure computing mode (`seccomp`) est une fonctionnalité du noyau Linux. Vous pouvez l'utiliser pour restreindre les actions disponibles dans le conteneur.. L'appel système `seccomp()` opère sur l'état seccomp du processus appelé. Vous pouvez utiliser cette fonction pour restreindre l'accès de votre application.

Cette fonctionnalité n'est disponible que si Docker a été construit avec `seccomp` et que le noyau est configuré avec `CONFIG_SECCOMP` activé.

Par exemple, nous pouvons refuser au conteneur la possibilité d'effectuer des actions telles que l'utilisation d'un `mount namepsace` ou d'un. [Linux system calls](https://filippo.io/linux-syscall-table/).

Plus d'information sur seccomp [ici](https://docs.docker.com/engine/security/seccomp/).

***

## 9.2 AppArmor

`**AppArmor` est une amélioration du kernel pour confiner les programmes à un ensemble limité de ressources. Il s'agit d'un contrôle d'accès obligatoire ou `MAC` qui lie les attributs de **access control** aux programmes plutôt qu'aux utilisateurs. 
Le confinement AppArmor est assuré par des **profiles loaded into the kernel**, généralement au démarrage. \

Les profils AppArmor peuvent être dans l'un des **deux modes** : 
- Les profils `Enforcement` auront pour effet d'appliquer la politique définie dans le profil et de signaler les tentatives de violation de la politique (via syslog ou auditd).
- Les profils en mode `Complain` n'appliquent pas la politique mais signalent les tentatives de violation de la politique.

AppArmor diffère de certains autres systèmes MAC sous Linux : il est basé sur les chemins d'accès, il permet de mélanger les profils `Enforcement` et `Complain`, il utilise des fichiers d'inclusion pour faciliter le développement, et sa barrière d'entrée est bien plus faible que celle des autres systèmes MAC populaires.
***
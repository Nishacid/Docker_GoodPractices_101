# Checklist de configuration d'un container Docker : 

- Inspectez l'image Docker avant de l'utiliser, il n'est pas impossible que l'image a été corrompue ou soit malveillante.

- N'utilisez pas l'argument `--privileged` et ne montez pas un Docker socket à l'intérieur du conteneur. Le socket Docker permet de créer des conteneurs, c'est donc un moyen facile de prendre le contrôle total de l'hôte, par exemple en exécutant un autre conteneur avec l'argument `--privileged`.


- Ne vous exécutez pas en tant que root à l'intérieur du conteneur. Utilisez un utilisateur ou namespaces d'utilisateurs différents. Le root dans le conteneur est la même que sur l'hôte, sauf si il est remappé avec des namespaces. Il n'est que légèrement restreint par, principalement, les `namespaces`, les `capabilities` et les `cgroups` de Linux.

- Supprimez toutes les `capabilities` (`--cap-drop=all`) et activez seulement celles qui sont nécessaires (`--cap-add=...`). Beaucoup de processus n'ont pas besoin de capacités et les ajouter augmente la portée d'une attaque potentielle.

- Utilisez l'option de sécurité `no-new-privileges` pour empêcher les processus d'obtenir plus de privilèges, par exemple par le biais de binaires `suid`.

- Limiter les ressources disponibles pour le conteneur. Les limites de ressources peuvent protéger la machine contre les attaques par déni de service.

- Ajustez les profils seccomp, AppArmor pour restreindre les actions et les appels système disponibles pour le conteneur au minimum requis

- Reconstruisez régulièrement vos images pour appliquer les correctifs de sécurité.
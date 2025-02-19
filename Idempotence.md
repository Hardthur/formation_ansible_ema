# atelier07

## Installation des paquets

Nous allons installer les paquets `tree`, `git` et `nmap` sur les TH.

```sh
ansible all -m package -a "name=tree,git,nmap"
rocky | CHANGED => {
    "changed": true,
    "msg": "",
    "rc": 0,
    "results": [
        "Installed: git-core-doc-2.43.5-2.el9_5.noarch",
        "Installed: perl-Error-1:0.17029-7.el9.noarch",
        "Installed: nmap-ncat-3:7.92-3.el9.x86_64",
        "Installed: nmap-3:7.92-3.el9.x86_64"
    ]
}
```

En exécutant à nouveau la commande, nous obtenons la réponse suivante : `"changed": false`.
Il s'agit d'un comportement normal puisque les paquets ont déjà été installés.

```sh
ansible all -m package -a "name=tree,git,nmap"
debian | SUCCESS => {
    "changed": false,
    "name": [
        "tree",
        "git",
        "nmap"
    ],
    "rc": 0,
    "state": "present",
    "update_cache": false
}
```

## Désinstallation des paquets

Désinstallation des paquets avec state=absent

```sh
ansible all -m package -a "name=tree,git,nmap state=absent"
suse | CHANGED => {
    "changed": true,
    "cmd": [
        "/usr/bin/zypper",
        "--quiet"
    ]
}
```

En exécutant à nouveau la commande, nous obtenons la réponse suivante : `"changed": false`.

```sh
ansible all -m package -a "name=tree,git,nmap state=absent"
debian | SUCCESS => {
    "changed": false
}
suse | SUCCESS => {
    "changed": false,
    "name": [
        "tree",
        "git",
        "nmap"
    ],
    "rc": 0,
    "state": "absent",
    "update_cache": false
}
```

## Copie de fichier

Copions le fichier `/etc/fstab` du contrôleur vers le fichier `/tmp/test3.txt` de toutes les machines cibles.

```sh
ansible all -m copy -a "src=/etc/fstab dest=/tmp/test3.txt"
rocky | CHANGED => {
    "changed": true,
    "dest": "/tmp/test3.txt",
    "gid": 0,
    "group": "root",
    "mode": "0644",
    "owner": "root",
    "size": 743,
    "state": "file",
    "uid": 0
}
```

En exécutant à nouveau la commande, nous obtenons la réponse suivante : `"changed": false`.

## Suppression du fichier

Suppression du fichier `/tmp/test3.txt`.

```sh
ansible all -m file -a "path=/tmp/test3.txt state=absent"

debian | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
rocky | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
suse | CHANGED => {
    "changed": true,
    "path": "/tmp/test3.txt",
    "state": "absent"
}
```

En exécutant à nouveau la commande, nous obtenons la réponse suivante : `"changed": false`.

## Affichage de l'espace disque

Affichage de l'espace utilisé sur les TH 

```sh
ansible all -m command -a "df -h /"
debian | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       124G  2.3G  115G   2% /
suse | CHANGED | rc=0 >>
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda3       124G  2.8G  118G   3% /
rocky | CHANGED | rc=0 >>
Filesystem                  Size  Used Avail Use% Mounted on
/dev/mapper/rl_rocky9-root   70G  2.4G   68G   4% /
```

Cette commande ne peut pas être idempotente car le résultat va souvent varier. 


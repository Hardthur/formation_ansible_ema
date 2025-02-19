# Atelier 06

## Exercice

Démarrage des VM présentes dans le répertoire `atelier-06`.

```sh
vagrant up
```

On se connecte au contrôleur.

```sh
vagrant ssh control
```

On rajoute des entrées DNS locales pour nos target host.

```sh
sudo nano /etc/hosts
```

```sh
192.168.56.10  control.sandbox.lan     control
192.168.56.20  target01.sandbox.lan    target01
192.168.56.30  target02.sandbox.lan    target02
192.168.56.40  target03.sandbox.lan    target03
```

On récupère les clés publiques des TH

```sh
ssh-keyscan -t rsa target01 target02 target03 >> .ssh/known_hosts
```

Et nous générons notre paire de clé RSA.

```sh
ssh-keygen
```

Enfin, distribuons notre clé publique aux différents hôtes cibles.

```sh
ssh-copy-id target01 && ssh-copy-id target02 && ssh-copy-id target03

```

Installons Ansible en utilisant les dépôts d'Ubuntu.

```sh
sudo apt update
sudo apt install ansible -y
```

Nous pouvons à présent tester notre configuration.

```sh
ansible all -i target01,target02,target03 -u vagrant -m ping
```

> Exemple de sortie de la commande :

```sh
vagrant@control:~$ ansible all -i target01,target02,target03 -u vagrant -m ping
target02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Créons un répertoire de travail pour notre projet Ansible ainsi qu'un fichier de configuration

```sh
mkdir ~/monprojet && touch ~/monprojet/ansible.cfg && cd monprojet
```

Vérifions que le fichier de configuration est bien pris en compte par Ansible.

```sh
cd ~/monprojet/
ansible --version | head -n 2
```

```sh
ansible 2.10.8
config file = /home/vagrant/monprojet/ansible.cfg
```

Modifions le fichier `ansible.cfg` pour y intégrer un fichier d'inventaire.

```sh
nano ansible.cfg
```

```sh
[defaults]
inventory = ./hosts
log_path = ~/journal/ansible.log
```

Testons la journalisation.

```sh
ansible all -i target01,target02,target03 -u vagrant -m ping
cat ~/journal/ansible.log
```

Après avoir lancé une nouvelle fois le module ping d'ansible, on obtient précisément le stdout dans les logs.

```sh
vagrant@control:~/monprojet$ cat ~/journal/ansible.log 
2025-02-18 11:42:25,286 p=4640 u=vagrant n=ansible | target01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
2025-02-18 11:42:25,292 p=4640 u=vagrant n=ansible | target03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
2025-02-18 11:42:25,295 p=4640 u=vagrant n=ansible | target02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Créons notre fichier d'inventaire et ajoutons un groupe `[testlab]` comprenant nos trois machines cibles.

```sh
nano hosts

[testlab]
target01
target02
target03

[testlab:vars]
ansible_user=vagrant
ansible_become=yes
```

Testons notre configuration.

```sh
vagrant@control:~/monprojet$ ansible  all -m ping
target02 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target01 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
target03 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}
```

Affichons la première ligne du fichier `/etc/shadow` de tous les hôtes cibles.

```sh
ansible all -a "head -n 1 /etc/shadow"
target02 | CHANGED | rc=0 >>
root:*:19769:0:99999:7:::
target01 | CHANGED | rc=0 >>
root:*:19769:0:99999:7:::
target03 | CHANGED | rc=0 >>
root:*:19769:0:99999:7:::
```

L'atelier est terminé. Nettoyons l'environnement.

```sh
exit
vagrant destroy -f
```


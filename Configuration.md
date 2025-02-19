# Atelier 06

## Exercice

Nous allons commencer par démarrer l'ensemble des machines virtuelles présentes dans le répertoire `atelier-06`.

```sh
vagrant up
```

Connectons-nous maintenant au contrôleur.

```sh
vagrant ssh control
```

Nous allons ensuite modifier le fichier `/etc/hosts` en associant les adresses IP des machines cibles à leurs noms d'hôtes.

```sh
sudo nano /etc/hosts
```

> Ajoutez les lignes suivantes au fichier :

```sh
192.168.56.10  control.sandbox.lan     control
192.168.56.20  target01.sandbox.lan    target01
192.168.56.30  target02.sandbox.lan    target02
192.168.56.40  target03.sandbox.lan    target03
```

Collectons les clés SSH publiques des hôtes cibles.

```sh
ssh-keyscan -t rsa target01 target02 target03 >> .ssh/known_hosts
```

Générons maintenant une clé SSH sur le contrôleur.

```sh
ssh-keygen
```

Enfin, distribuons notre clé publique aux différents hôtes cibles.
Le mot de passe par défaut des machines virtuelles est `vagrant`.

```sh
ssh-copy-id vagrant@target01
ssh-copy-id vagrant@target02
ssh-copy-id vagrant@target03
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
target02 | SUCCESS => {"ping": "pong"}
target03 | SUCCESS => {"ping": "pong"}
target01 | SUCCESS => {"ping": "pong"}
```

Créons un répertoire de travail pour notre projet Ansible.

```sh
mkdir ~/monprojet
```

Puis créons un fichier `ansible.cfg` dans ce répertoire.

```sh
touch ~/monprojet/ansible.cfg
```

Vérifions que le fichier est bien pris en compte par Ansible.

```sh
cd ~/monprojet/
ansible --version | head -n 2
```

> Exemple de sortie :

```sh
ansible 2.10.8
config file = /home/vagrant/monprojet/ansible.cfg
```

Modifions le fichier `ansible.cfg` pour y intégrer un fichier d'inventaire.

```sh
nano ansible.cfg
```

> Ajoutez les lignes suivantes au fichier :

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

Créons notre fichier d'inventaire et ajoutons un groupe `[testlab]` comprenant nos trois machines cibles.

```sh
nano hosts
```

> Ajoutez les lignes suivantes au fichier :

```sh
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
ansible all -m ping
```

> Exemple de sortie :

```sh
target02 | SUCCESS => {"ping": "pong"}
target01 | SUCCESS => {"ping": "pong"}
target03 | SUCCESS => {"ping": "pong"}
```

Affichons la première ligne du fichier `/etc/shadow` de tous les hôtes cibles.

```sh
ansible all -a "head -n 1 /etc/shadow"
```

> Exemple de sortie :

```sh
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


#Labo Installation

##Exercice 1

Activation & connexion à la vm ubuntu.
```vagrant up ubuntu && vagrant ssh ubuntu```

apt-update
apt-cache search --names-only "ansible"
apt install ansible
vérifier si l'installation s'est bien déroulée ? apt show ?
version d'ansible visible avec apt show

==
apt show ansible -a 
On remarque une version complètement différente "10.7.0-1ppa~jammy" priority optional
ansible --version

==

vagrant up rocky
" ssh "
dnf install python
$ python3 -m venv ~/.venv/ansible
$ source ~/.venv/ansible/bin/activate
(ansible) [vagrant@rocky ~]$ pip install --upgrade pip
(ansible) [vagrant@rocky ~]$ pip install ansible
(ansible) [vagrant@rocky ~]$ ansible --version
ansible [core 2.15.13]

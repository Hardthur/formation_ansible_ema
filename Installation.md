# Labo Installation

## Exercice 1

Activation & connexion à la vm ubuntu.  

```vagrant up ubuntu && vagrant ssh ubuntu```  

Rafraichissement des paquets  

```apt-update```  

Recherche du paquet ansible  

```apt-cache search --names-only "ansible"```  

Installation d'Ansible  

```apt install ansible```  

Vérification version d'ansible  

```ansible --version```  

Déconnexion et suppression
```vagrant destroy ubuntu```

## Exercice 2

Configuration d'un dépôt PPA  

```sudo apt-add-repository ppa:ansible/ansible```  

Grâce à la commande apt show ansible, on remarque une version complètement différente "10.7.0-1ppa~jammy" priority optional

## Exercice 3

Création et connexion à la vm rocky linux  
```vagrant up rocky && vagrant ssh rocky```  

Installation de python  
```dnf install python```

Activation du virtual environment  

```python3 -m venv ~/.venv/ansible && source ~/.venv/ansible/bin/activate```


Installation d'ansible via PIP et vérification de la version  
```(ansible) [vagrant@rocky ~]$ pip install --upgrade pip
(ansible) [vagrant@rocky ~]$ pip install ansible
(ansible) [vagrant@rocky ~]$ ansible --version
ansible [core 2.15.13]```

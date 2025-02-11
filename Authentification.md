# Labo Authentification  


Démarrage des VM et connexion au Control Host
`vagrant up && vagrant ssh control`  

Modification du fichier /etc/hosts pour rajouter quatre entrées DNS locales pour nos machines.  

```
nano /etc/hosts
192.168.56.10 control
192.168.56.20 target01
192.168.56.30 target02
192.168.56.40 target03
```

Création d'une paire de clés RSA  

`ssh-keygen`  
Envoi de notre clé publique sur les trois target hosts.  
`ssh-copy-id target01 && ssh-copy-id target02 && ssh-copy-id target03`  

Enfin, test de ping/ssh avec un module intégré à ansible.  


`ansible all -i target01,target02,target03 -m ping`

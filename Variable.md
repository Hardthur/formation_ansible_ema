# atelier14

## Exercice


Ecriture d'un premier playbook 

```sh
nano playbooks/myvars1.yml

---  # myvars1.yml

- hosts: localhost
  gather_facts: false

  vars:
    mycar: Mercedes S63 AMG
    mybike: Mercedes ne fait que des voitures

  tasks:
    - debug:
        msg: "Voiture préférée : {{mycar}}, Moto préférée : {{mybike}}"

...
```

Vérification de la syntaxe + lancement du playbook
NB : si yamllint échoue, les doubles esperluettes ne lanceront pas la commande ansible-playbook

```sh
yamllint myvars1.yml && ansible-playbook myvars1.yml

PLAY [localhost] ****************************************************************************************************************************************************************************

TASK [debug] ********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Voiture préférée : Mercedes S63 AMG, Moto préférée : Mercedes ne fait que des voitures"
}

PLAY RECAP **********************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

On override la variable mycar 

```sh
ansible-playbook myvars1.yml -e mycar=Mercedes CLS63 AMG

PLAY [localhost] ****************************************************************************************************************************************************************************

TASK [debug] ********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Voiture préférée : Mercedes CLS63 AMG, Moto préférée : Mercedes ne fait que des voitures"
}

PLAY RECAP **********************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```sh
ansible-playbook myvars1.yml -e mybike=Mercedes ne fait vraiment que des voitures

PLAY [localhost] ****************************************************************************************************************************************************************************

TASK [debug] ********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Voiture préférée : Mercedes S63 AMG, Moto préférée : Mercedes ne fait vraiment que des voitures"
}

PLAY RECAP **********************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```sh
ansible-playbook myvars1.yml -e mycar=Mercedes-Maybach S650 -e mybike=Les motos c'est dangereux

PLAY [localhost] ****************************************************************************************************************************************************************************

TASK [debug] ********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Voiture préférée : Mercedes-Maybach S650, Moto préférée : Les motos c'est dangereux"
}

PLAY RECAP **********************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


Passons à un second *playbook* nommé `myvars2.yml` faisant la même chose que `myvars1.yml`, mais en utilisant `set_fact` pour les deux variables.

```sh
nano playbooks/myvars2.yml

---  # myvars2.yml

- hosts: localhost
  gather_facts: false

  tasks:

    - name: Define variables
      set_fact:
        mycar: Mercedes-Maybach S650
        mybike: Les motos c'est dangereux

    - debug:
        msg: "Voiture préférée : {{mycar}}, Moto préférée : {{mybike}}"

...
```

```sh
yamllint myvars2.yml && ansible-playbook myvars2.yml

PLAY [localhost] ****************************************************************************************************************************************************************************

TASK [Define variables] *********************************************************************************************************************************************************************
ok: [localhost]

TASK [debug] ********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Voiture préférée : Mercedes-Maybach S650, Moto préférée : Les motos c'est dangereux"
}

PLAY RECAP **********************************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

On va remplacer ces nouvelles variables par les extra_vars, qui prendront le dessus. 

```sh
ansible-playbook myvars2.yml -e mycar=Mercedes SLS AMG -e mybike=Si Mercedes faisait des motos pourquoi pas

PLAY [localhost] ****************************************************************************************************************************************************************************

TASK [Define variables] *********************************************************************************************************************************************************************
ok: [localhost]

TASK [debug] ********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "Voiture préférée : Mercedes SLS AMG, Moto préférée : Si Mercedes faisait des motos pourquoi pas"
}

PLAY RECAP **********************************************************************************************************************************************************************************
localhost                  : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

3ème playbook, vous me demandez de diffamer ma marque préférée et je serai dans l'incapacité de faire cela. 

```sh
nano playbooks/myvars3.yml

---  # myvars3.yml

- hosts: all
  gather_facts: false

  tasks:
    - debug:
        msg: "Voiture préférée : {{mycar}}, Moto préférée : {{mybike}}"

...
```

Création d'un répertoire group_vars 

```sh
mkdir ~/ansible/projets/ema/group_vars

nano ~/ansible/projets/ema/group_vars/all.yml

---  # group_vars/all.yml

mycar: Mercedes-Benz
mybike: Mercedes-Maybach

...
```

Testons à présent le `playbook` non diffamatoire 

```sh
yammlint myvars3.yml && ansible-playbook myvars3.yml

PLAY [all] **********************************************************************************************************************************************************************************

TASK [debug] ********************************************************************************************************************************************************************************
ok: [target02] => {
    "msg": "Voiture préférée : Mercedes-Benz, Moto préférée : Mercedes-Maybach"
}
ok: [target01] => {
    "msg": "Voiture préférée : Mercedes-Benz, Moto préférée : Mercedes-Maybach"
}
ok: [target03] => {
    "msg": "Voiture préférée : Mercedes-Benz, Moto préférée : Mercedes-Maybach"
}

PLAY RECAP **********************************************************************************************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

J'aperçois enfin un playbook à moitié-moins diffamatoire
Créons un fichier yaml target02.yml dans le répertoire host_vars

```sh
mkdir ~/ansible/projets/ema/host_vars
nano ~/ansible/projets/ema/host_vars/target02.yml

---  # host_vars/target02.yml

mycar: Mercedes
mybike: Aucune autre marque n'existe que Mercedes

...
```

```sh
yamllint myvars3.yml && ansible-playbook myvars3.yml

PLAY [all] **********************************************************************************************************************************************************************************

TASK [debug] ********************************************************************************************************************************************************************************
ok: [target01] => {
    "msg": "Voiture préférée : Mercedes-Benz, Moto préférée : Mercedes-Maybach"
}
ok: [target02] => {
    "msg": "Voiture préférée : Mercedes, Moto préférée : Aucune autre marque n'existe que Mercedes"
}
ok: [target03] => {
    "msg": "Voiture préférée : Mercedes-Benz, Moto préférée : Mercedes-Maybach"
}

PLAY RECAP **********************************************************************************************************************************************************************************
target01                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target02                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target03                   : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Dernier exercice
```sh
nano playbooks/display_user.yml

---  # display_user.yml

- hosts: localhost
  gather_facts: false

  vars_prompt:

    - name: user
      prompt: Tu t'appelles comment ?
      default: microlinux
      private: false

    - name: password
      prompt: Mot de passe hyper secret stp
      default: yatahongaga
      private: true

  tasks:
    - debug:
        msg: "L'utilisateur {{user}} a pour mot de passe : {{password}}"

...
```

```sh
yamllint display_user.yml && ansible-playbook display_user.yml

Tu t'appelles comment ? [microlinux]: Laisse moi zoum zoum zang
Mot de passe hyper secret stp [yatahongaga]:

PLAY [localhost] ****************************************************************************************************************************************************************************

TASK [debug] ********************************************************************************************************************************************************************************
ok: [localhost] => {
    "msg": "L'utilisateur Laisse moi zoum zoum zang a pour mot de passe : Dans ta benz benz benz"
}

PLAY RECAP **********************************************************************************************************************************************************************************
localhost                  : ok=1    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

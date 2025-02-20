# atelier16

## Exercice

Ecriture d'un playbook affichant le gestionnaire de paquet utilisé

```sh
nano pkg-info.yml

---  # pkg-info.yml

- hosts: all
  gather_facts: true

  tasks:

    - name: Display package manager
      debug:
        msg: "Le gestionnaire de paquets utilisé est : {{ ansible_pkg_mgr }}"

...
```


```sh
yamllint pkg-info.yml && ansible-playbook pkg-info.yml

PLAY [all] *********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
ok: [debian]
ok: [suse]
ok: [rocky]

TASK [Display package manager] *************************************************************************************************************************************************************************************************************
ok: [rocky] => {
    "msg": "Le gestionnaire de paquets utilisé est : dnf"
}
ok: [debian] => {
    "msg": "Le gestionnaire de paquets utilisé est : apt"
}
ok: [suse] => {
    "msg": "Le gestionnaire de paquets utilisé est : zypper"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Puis, playbook similaire nous donnant la verison de Python installée.

```sh
nano python-info.yml

---  # python-info.yml

- hosts: all
  gather_facts: true

  tasks:
    - name: Display Python version
      debug:
        msg: "La version de Python installée est : {{ ansible_python_version }}"

...
```


```sh
yamllint python-info.yml && ansible-playbook python-info.yml

PLAY [all] *********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Display Python version] **************************************************************************************************************************************************************************************************************
ok: [rocky] => {
    "msg": "La version de Python installée est : 3.9.18"
}
ok: [debian] => {
    "msg": "La version de Python installée est : 3.11.2"
}
ok: [suse] => {
    "msg": "La version de Python installée est : 3.6.15"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Pour finir, nous allons voir quels sont les serveurs DNS utilisés sur nos TH.

```sh
nano dns-info.yml

---  # dns-info.yml

- hosts: all
  gather_facts: true

  tasks:
    - name: Display DNS servers
      debug:
        msg: "Serveur(s) DNS utilisé(s) : {{ ansible_facts.dns.nameservers }}"

...
```


```sh
yamllint dns-info.yml && ansible-playbook dns-info.yml

PLAY [all] *********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]

TASK [Display DNS servers] *****************************************************************************************************************************************************************************************************************
ok: [rocky] => {
    "msg": "Serveur(s) DNS utilisé(s) : ['10.0.2.3']"
}
ok: [debian] => {
    "msg": "Serveur(s) DNS utilisé(s) : ['4.2.2.1', '4.2.2.2', '208.67.220.220']"
}
ok: [suse] => {
    "msg": "Serveur(s) DNS utilisé(s) : ['10.0.2.3']"
}

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

# atelier18

## Exercice

Ecriture d'un playbook chrony qui configurera différemment l'applicatif en fonction de la distribution. 
Il suppose que chrony soit déjà installé, notamment grâce à notre précédent playbook.

```sh
nano chrony.yml

---  # chrony.yml
- name: Configure Chrony on different distributions
  hosts: all
  tasks:
    - name: Determine Chrony configuration path
      set_fact:
        chrony_conf_path: >-
          {% if ansible_facts['distribution'] in ['Rocky', 'openSUSE'] %}/etc/chrony.conf
          {% elif ansible_facts['distribution'] in ['Debian', 'Ubuntu'] %}/etc/chrony/chrony.conf
          {% else %}/etc/chrony.conf{% endif %}

    - name: Deploy Chrony configuration
      template:
        src: chrony.conf.j2
        dest: "{{ chrony_conf_path }}"
        mode: '0644'
      notify: Restart Chrony

  handlers:
    - name: Restart Chrony
      service:
        name: chronyd
        state: restarted

```
Création d'un répertoire templates avec le fichier chrony.conf.j2
```sh
mkdir templates
nano templates/chrony.conf.j2
```

> Retrouvez ci-dessous le contenu du fichier :

```sh
# {{chrony_conf_path}}
server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst 

driftfile /var/lib/chrony/drift
makestep 1.0 {{ 3 + inventory_hostname | length }}
rtcsync
logdir /var/log/chrony_{{ inventory_hostname }}

```

La variable `{{chrony_conf_path}}` contient le chemin du fichier de configuration de chrony, qui varie en fonction des distributions.   
Cette configuration est dynamique, elle permet de la rendre unique à la machine. 

test du playbook

```sh
yamllint chrony.yml && ansible-playbook chrony.yml


PLAY [all] *********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]
ok: [ubuntu]

TASK [Determine Chrony configuration path] ***********************************************************************************************************************************************************************************************
ok: [rocky]
ok: [suse]
ok: [debian]
ok: [ubuntu]

TASK [Deploy Chrony configuration] *****************************************************************************************************************************************************************************************
ok: [rocky]
ok: [suse]
ok: [debian]
ok: [ubuntu]

RUNNING HANDLER [Restart chronyd] **********************************************************************************************************************************************************************************************************
changed: [debian]
changed: [ubuntu]
changed: [rocky]
changed: [suse]

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
debian                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rocky                      : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
suse                       : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
ubuntu                     : ok=3    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Connectons-nous à l'hôte `suse` pour vérifier le bon fonctionnement de notre configuration.

```sh
ssh suse

vagrant@suse:~> cat /etc/chrony.conf
# /etc/chrony.conf

server 0.fr.pool.ntp.org iburst
server 1.fr.pool.ntp.org iburst
server 2.fr.pool.ntp.org iburst
server 3.fr.pool.ntp.org iburst

driftfile /var/lib/chrony/drift
makestep 1.0 14
rtcsync
logdir /var/log/chrony_suse

```

Cela conclut nos ateliers. Bonne journée.  
N'oubliez pas, il n'existe que Mercedes sur le marché automobile.

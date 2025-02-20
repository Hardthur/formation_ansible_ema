# atelier17

## Exercice

Ecriture d'un *playbook* `chrony-01.yml` mettant en place la synchronisation NTP avec `chrony` sur l'ensemble des *TH*.

```sh
nano chrony-01.yml

---  # chrony-01.yml

- hosts: all

  tasks:

    ## Install chrony

    - name: Update package information on Debian/Ubuntu
      apt:
        update_cache: true
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Install chrony on Debian/Ubuntu
      apt:
        name: chrony
      when: ansible_os_family == "Debian"

    - name: Install chrony on Rocky Linux
      dnf:
        name: chrony
      when: ansible_distribution == "Rocky"

    - name: Install chrony on SUSE Linux
      zypper:
        name: chrony
      when: ansible_distribution == "openSUSE Leap"


    ## Start & enable chronyd

    - name: Start & enable chronyd on Debian/Ubuntu
      service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_os_family == "Debian"

    - name: Start & enable chronyd on Rocky Linux
      service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_distribution == "Rocky"

    - name: Start & enable chronyd on SUSE Linux
      service:
        name: chronyd
        state: started
        enabled: true
      when: ansible_distribution == "openSUSE Leap"


    ## Configure chrony servers and options

    - name: Configure chrony servers on Debian/Ubuntu
      copy:
        dest: /etc/chrony/chrony.conf
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart chronyd
      when: ansible_os_family == "Debian"

    - name: Configure chrony servers on Rocky Linux
      copy:
        dest: /etc/chrony.conf
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart chronyd
      when: ansible_distribution == "Rocky"

    - name: Configure chrony servers on SUSE Linux
      copy:
        dest: /etc/chrony.conf
        content: |
          server 0.fr.pool.ntp.org iburst
          server 1.fr.pool.ntp.org iburst
          server 2.fr.pool.ntp.org iburst
          server 3.fr.pool.ntp.org iburst
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart chronyd
      when: ansible_distribution == "openSUSE Leap"

  handlers:

    - name: Restart chronyd
      service:
        name: chronyd
        state: restarted

...
```

Ce playbook est excessivement mal optimisé, on va le tester et le lancer.

```sh
yamllint chrony-01.yml && ansible-playbook chrony-01.yml

PLAY [all] *********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [ubuntu]
ok: [suse]

TASK [Update package information on Debian/Ubuntu] *****************************************************************************************************************************************************************************************
skipping: [rocky]
skipping: [suse]
ok: [debian]
ok: [ubuntu]

TASK [Install chrony on Debian/Ubuntu] *****************************************************************************************************************************************************************************************************
skipping: [rocky]
skipping: [suse]
changed: [debian]
changed: [ubuntu]

TASK [Install chrony on Rocky Linux] *******************************************************************************************************************************************************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
ok: [rocky]

TASK [Install chrony on SUSE Linux] ********************************************************************************************************************************************************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
changed: [suse]

TASK [Start & enable chronyd on Debian/Ubuntu] *********************************************************************************************************************************************************************************************
skipping: [rocky]
skipping: [suse]
ok: [debian]
ok: [ubuntu]

TASK [Start & enable chronyd on Rocky Linux] ***********************************************************************************************************************************************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
ok: [rocky]

TASK [Start & enable chronyd on SUSE Linux] ************************************************************************************************************************************************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
changed: [suse]

TASK [Configure chrony servers on Debian/Ubuntu] *******************************************************************************************************************************************************************************************
skipping: [rocky]
skipping: [suse]
changed: [debian]
changed: [ubuntu]

TASK [Configure chrony servers on Rocky Linux] *********************************************************************************************************************************************************************************************
skipping: [debian]
skipping: [suse]
skipping: [ubuntu]
changed: [rocky]

TASK [Configure chrony servers on SUSE Linux] **********************************************************************************************************************************************************************************************
skipping: [rocky]
skipping: [debian]
skipping: [ubuntu]
changed: [suse]

RUNNING HANDLER [Restart chronyd] **********************************************************************************************************************************************************************************************************
changed: [debian]
changed: [ubuntu]
changed: [rocky]
changed: [suse]

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
debian                     : ok=6    changed=3    unreachable=0    failed=0    skipped=6    rescued=0    ignored=0
rocky                      : ok=5    changed=2    unreachable=0    failed=0    skipped=7    rescued=0    ignored=0
suse                       : ok=5    changed=4    unreachable=0    failed=0    skipped=7    rescued=0    ignored=0
ubuntu                     : ok=6    changed=3    unreachable=0    failed=0    skipped=6    rescued=0    ignored=0
```
On va utiliser une méthode un peu plus propre en définissant trois variables. 

```sh
nano chrony-02.yml

---  # chrony-02.yml

- hosts: all

  tasks:

    - name: Load distribution-specific parameters
      include_vars: >
        chrony_{{ansible_distribution|lower|replace(" ", "-") }}.yml

    - name: Update package information on Debian/Ubuntu
      apt:
        update_cache: true
        cache_valid_time: 3600
      when: ansible_os_family == "Debian"

    - name: Install chrony
      package:
        name: "{{chrony_package}}"

    - name: Start & enable chronyd
      service:
        name: "{{chrony_service}}"
        state: started
        enabled: true

    - name: Configure chrony servers and options
      copy:
        dest: "{{chrony_confdir}}"
        content: |
          server 0.fr.pool.ntp.org
          server 1.fr.pool.ntp.org
          driftfile /var/lib/chrony/drift
          makestep 1.0 3
          rtcsync
          logdir /var/log/chrony
      notify: Restart chronyd

  handlers:

    - name: Restart chronyd
      service:
        name: "{{chrony_service}}"
        state: restarted

...
```

Nous devons ensuite créer le répertoire `vars` dans le dossier `playbooks` et y définir les fichiers de variables, un pour chaque distribution (`debian`, `ubuntu`, `rocky` et `openSUSE`).

```sh
mkdir vars
nano vars/chrony_debian.yml

---  # vars/chrony_debian.yml

chrony_package: chrony
chrony_service: chronyd
chrony_confdir: /etc/chrony/chrony.conf

...
```

```sh
nano vars/chrony_ubuntu.yml

---  # vars/chrony_ubuntu.yml

chrony_package: chrony
chrony_service: chronyd
chrony_confdir: /etc/chrony/chrony.conf

...
```

```sh
nano vars/chrony_rocky.yml

---  # vars/chrony_rocky.yml

chrony_package: chrony
chrony_service: chronyd
chrony_confdir: /etc/chrony.conf

...
```

```sh
nano vars/chrony_opensuse-leap.yml

---  # vars/chrony_opensuse-leap.yml

chrony_package: chrony
chrony_service: chronyd
chrony_confdir: /etc/chrony.conf

...
```

Nous pouvons à présent exécuter le *playbook* :
```sh
yamllint chrony-02.yml && ansible-playbook chrony-02.yml.

PLAY [all] *********************************************************************************************************************************************************************************************************************************

TASK [Gathering Facts] *********************************************************************************************************************************************************************************************************************
ok: [debian]
ok: [rocky]
ok: [suse]
ok: [ubuntu]

TASK [Load distribution-specific parameters] ***********************************************************************************************************************************************************************************************
ok: [rocky]
ok: [debian]
ok: [suse]
ok: [ubuntu]

TASK [Update package information on Debian/Ubuntu] *****************************************************************************************************************************************************************************************
skipping: [rocky]
skipping: [suse]
ok: [debian]
ok: [ubuntu]

TASK [Install chrony] **********************************************************************************************************************************************************************************************************************
ok: [debian]
ok: [suse]
ok: [ubuntu]
ok: [rocky]

TASK [Start & enable chronyd] **************************************************************************************************************************************************************************************************************
ok: [debian]
ok: [ubuntu]
ok: [rocky]
ok: [suse]

TASK [Configure chrony servers and options] ************************************************************************************************************************************************************************************************
changed: [debian]
changed: [ubuntu]
changed: [rocky]
changed: [suse]

RUNNING HANDLER [Restart chronyd] **********************************************************************************************************************************************************************************************************
changed: [debian]
changed: [ubuntu]
changed: [rocky]
changed: [suse]

PLAY RECAP *********************************************************************************************************************************************************************************************************************************
debian                     : ok=7    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rocky                      : ok=6    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
suse                       : ok=6    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0
ubuntu                     : ok=7    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


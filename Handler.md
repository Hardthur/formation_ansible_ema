# atelier12

## Exercice

Ecriture d'un playbook "chrony.yml" qui synchronisera le NTP des TH. 

```sh
nano playbooks/chrony.yml

---  # chrony.yml

- hosts: redhat

  tasks:

    - name: Install chrony
      dnf:
        name: chrony
        state: present

    - name: Start & enable chronyd
      service:
        name: chronyd
        state: started
        enabled: true

    - name: Configure chrony servers and options
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

  handlers:

    - name: Restart chronyd
      service:
        name: chronyd
        state: restarted

...
```

Test de la syntaxe avec yamllint. Aucun stdout donc le playbook est validé. 
```sh
yamllint chrony.yml
```

```sh
ansible-playbook chrony.yml
PLAY [redhat] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************
ok: [target03]
ok: [target01]
ok: [target02]

TASK [Install chrony] ***********************************************************************************************************************************************************************
changed: [target01]
changed: [target03]
changed: [target02]

TASK [Start & enable chronyd] ***************************************************************************************************************************************************************
changed: [target03]
changed: [target01]
changed: [target02]

TASK [Configure chrony servers and options] *************************************************************************************************************************************************
changed: [target01]
changed: [target02]
changed: [target03]

RUNNING HANDLER [Restart chronyd] ***********************************************************************************************************************************************************
changed: [target01]
changed: [target02]
changed: [target03]

PLAY RECAP **********************************************************************************************************************************************************************************
target01                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target02                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target03                   : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Vérification de l'idempotence du playbook. Grâce au handler, toutes les tâches devraient être au vert (`changed=0`).

```sh
[vagrant@control playbooks]$ ansible-playbook chrony.yml

PLAY [redhat] *******************************************************************************************************************************************************************************

TASK [Gathering Facts] **********************************************************************************************************************************************************************
ok: [target03]
ok: [target02]
ok: [target01]

TASK [Install chrony] ***********************************************************************************************************************************************************************
ok: [target02]
ok: [target03]
ok: [target01]

TASK [Start & enable chronyd] ***************************************************************************************************************************************************************
ok: [target02]
ok: [target03]
ok: [target01]

TASK [Configure chrony servers and options] *************************************************************************************************************************************************
ok: [target02]
ok: [target01]
ok: [target03]

PLAY RECAP **********************************************************************************************************************************************************************************
target01                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target02                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
target03                   : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

L'idempotence est présente. 

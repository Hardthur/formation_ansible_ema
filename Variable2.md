# atelier15

## Exercice


Ecriture d'un playbook kernel.yml qui affiche des informations sur notre noyau.

```sh
nano kernel.yml

---  # kernel.yml

- hosts: all
  gather_facts: false

  tasks:

    - name: Display kernel detailed info
      command: uname -a
      changed_when: false
      register: df_cmd

    - debug:
        msg: "{{df_cmd.stdout_lines}}"

...
```

Lancement du playbook
```sh
yamllint kernel.yml && ansible-playbook kernel.yml

PLAY [all] **********************************************************************************************************************************************************************************

TASK [Display kernel detailed info] *********************************************************************************************************************************************************
ok: [debian]
ok: [suse]
ok: [rocky]

TASK [debug] ********************************************************************************************************************************************************************************
ok: [rocky] => {
    "msg": [
        "Linux rocky 5.14.0-362.13.1.el9_3.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 13 14:07:45 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux"
    ]
}
ok: [debian] => {
    "msg": [
        "Linux debian 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x86_64 GNU/Linux"
    ]
}
ok: [suse] => {
    "msg": [
        "Linux suse 5.14.21-150500.55.39-default #1 SMP PREEMPT_DYNAMIC Tue Dec 5 10:06:35 UTC 2023 (2e4092e) x86_64 x86_64 x86_64 GNU/Linux"
    ]
}

PLAY RECAP **********************************************************************************************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

A la place de msg, on va essayer vars

```sh
---  # kernel.yml

- hosts: all
  gather_facts: false

  tasks:

    - name: Display kernel detailed info
      command: uname -a
      changed_when: false
      register: df_cmd

    - debug:
        var: df_cmd.stdout_lines

...
```

```sh
yamllint kernel.yml && ansible-playbook kernel.yml

PLAY [all] **********************************************************************************************************************************************************************************

TASK [Display kernel detailed info] *********************************************************************************************************************************************************
ok: [debian]
ok: [suse]
ok: [rocky]

TASK [debug] ********************************************************************************************************************************************************************************
ok: [rocky] => {
    "df_cmd.stdout_lines": [
        "Linux rocky 5.14.0-362.13.1.el9_3.x86_64 #1 SMP PREEMPT_DYNAMIC Wed Dec 13 14:07:45 UTC 2023 x86_64 x86_64 x86_64 GNU/Linux"
    ]
}
ok: [debian] => {
    "df_cmd.stdout_lines": [
        "Linux debian 6.1.0-17-amd64 #1 SMP PREEMPT_DYNAMIC Debian 6.1.69-1 (2023-12-30) x86_64 GNU/Linux"
    ]
}
ok: [suse] => {
    "df_cmd.stdout_lines": [
        "Linux suse 5.14.21-150500.55.39-default #1 SMP PREEMPT_DYNAMIC Tue Dec 5 10:06:35 UTC 2023 (2e4092e) x86_64 x86_64 x86_64 GNU/Linux"
    ]
}

PLAY RECAP **********************************************************************************************************************************************************************************
debian                     : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Ecriture d'un playbook packages.yml qui affichera la quantité de paquets installés sur une distribution suse et rocky.

```sh
nano packages.yml

---  # packages.yml

- hosts: rocky, suse
  gather_facts: false

  tasks:

    - name: Get number of installed RPM packages
      shell: rpm -qa | wc -l
      changed_when: false
      register: df_cmd

    - debug:
        var: df_cmd.stdout_lines

...
```

```sh
yamllint packages.yml && ansible-playbook packages.yml

PLAY [rocky, suse] **************************************************************************************************************************************************************************

TASK [Get number of installed RPM packages] *************************************************************************************************************************************************
ok: [rocky]
ok: [suse]

TASK [debug] ********************************************************************************************************************************************************************************
ok: [rocky] => {
    "df_cmd.stdout_lines": [
        "671"
    ]
}
ok: [suse] => {
    "df_cmd.stdout_lines": [
        "917"
    ]
}

PLAY RECAP **********************************************************************************************************************************************************************************
rocky                      : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
suse                       : ok=2    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```


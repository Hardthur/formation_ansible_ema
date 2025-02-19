# atelier10

## Exercice

Ecriture d'un playbook `apache-debian.yml` qui installera Apache sur l'hôte `debian`.  

```sh
nano playbooks/apache-debian.yml

---  # apache-debian.yml

- hosts: debian

  tasks:

    - name: Update package information
      apt:
        update_cache: true
        cache_valid_time: 3600

    - name: Install Apache
      apt:
        name: apache2

    - name: Start & enable Apache
      service:
        name: apache2
        state: started
        enabled: true

    - name: Install custom web page
      copy:
        dest: /var/www/html/index.html
        mode: 0644
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Apache on Debian Linux</title>
            </head>
            <body>
              <h1>Mercedes-Benz</h1>
            </body>
          </html>

...
```

Test de la syntaxe yaml avec l'outil yamllint 

```sh
yamllint apache-debian.yml
```

Exécution du playbook suite à une confirmation de la syntaxe.  

```sh
ansible-playbook apache-debian.yml

PLAY [debian] *************************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************
ok: [debian]

TASK [Update package information] *****************************************************************************************************************************************
changed: [debian]

TASK [Install Apache] *****************************************************************************************************************************************************
changed: [debian]

TASK [Start & enable Apache] **********************************************************************************************************************************************
ok: [debian]

TASK [Install custom web page] ********************************************************************************************************************************************
changed: [debian]

PLAY RECAP ****************************************************************************************************************************************************************
debian                     : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Affichage du contenu de la page

```sh
[vagrant@ansible playbooks]$ curl debian
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Apache on Debian Linux</title>
  </head>
  <body>
    <h1>Mercedes-Benz</h1>
  </body>
</html>
```

Deuxième playbook pour l'installation sur Rocky. 

```sh
nano playbooks/apache-rocky.yml

---  # apache-rocky.yml

- hosts: rocky

  tasks:

    - name: Update package information
      dnf:
        name: '*'
        state: latest
        update_cache: true

    - name: Install Apache
      dnf:
        name: httpd
        state: present

    - name: Start & enable Apache
      service:
        name: httpd
        state: started
        enabled: true

    - name: Install custom web page
      copy:
        dest: /var/www/html/index.html
        mode: 0644
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Apache on Rocky Linux</title>
            </head>
            <body>
              <h1>Mercedes S-Klasse</h1>
            </body>
          </html>

...
```

Après avoir testé la syntaxe, on exécute le playbook. 


```sh
ansible-playbook apache-rocky.yml
PLAY [rocky] **************************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************
ok: [rocky]

TASK [Update package information] *****************************************************************************************************************************************
ok: [rocky]

TASK [Install Apache] *****************************************************************************************************************************************************
changed: [rocky]

TASK [Start & enable Apache] **********************************************************************************************************************************************
changed: [rocky]

TASK [Install custom web page] ********************************************************************************************************************************************
changed: [rocky]

PLAY RECAP ****************************************************************************************************************************************************************
rocky                      : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

Affichage de la page web.

```sh
curl rocky

[vagrant@ansible playbooks]$ curl rocky
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Apache on Rocky Linux</title>
  </head>
  <body>
    <h1>Mercedes S-Klasse</h1>
  </body>
</html>

```

Pour finir, écriture d'un dernier playbook pour l'installation d'apache sur Suse.

```sh
nano playbooks/apache-suse.yml
---  # apache-suse.yml

- hosts: suse

  tasks:

    - name: Install Apache
      zypper:
        name: apache2
        state: present

    - name: Start & enable Apache
      service:
        name: apache2
        state: started
        enabled: true

    - name: Install custom web page
      copy:
        dest: /srv/www/htdocs/index.html
        mode: 0644
        content: |
          <!doctype html>
          <html>
            <head>
              <meta charset="utf-8">
              <title>Apache on SUSE Linux</title>
            </head>
            <body>
              <h1>Mercedes CLS-Klasse</h1>
            </body>
          </html>

...
```

Yamllint confirme la syntaxe, on exécute le playbook

```sh
ansible-playbook apache-suse.yml

PLAY [suse] ***************************************************************************************************************************************************************

TASK [Gathering Facts] ****************************************************************************************************************************************************
ok: [suse]

TASK [Install Apache] *****************************************************************************************************************************************************
changed: [suse]

TASK [Start & enable Apache] **********************************************************************************************************************************************
changed: [suse]

TASK [Install custom web page] ********************************************************************************************************************************************
changed: [suse]

PLAY RECAP ****************************************************************************************************************************************************************
suse                       : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0
```

```sh
curl suse

<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>Apache on SUSE Linux</title>
  </head>
  <body>
    <h1>Mercedes CLS-Klasse</h1>
  </body>
</html>
```

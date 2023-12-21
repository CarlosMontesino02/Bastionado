
### Inventariado y primeras pruebas

Creo la ruta /etc/ansible/hosts y en ese fichero defino los diferentes hosts de la red agrupaddos. También podriamsoc rear el nuestro e indicarlo con la opción -i.

``` 
/etc/ansible/hosts:

[hosts]
192.168.122.231
```

**ansible all --list-hosts**

Ahora para realizar pruebas, vamos a hacer un ping, primero hay que configurar los servidores copiando nuestra clave pública, para ello:

``` bash
$ ssh-keygen
$ ssh-copy-id usuario_remoto@ip_servidor_remoto
#test
carlosmofe@carlosmofe-MS-7C39:~$ ansible all -m ping
192.168.122.231 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false,
    "ping": "pong"
}

```

Otro ejemplo de archivo de inventariado:

``` yml
virtualmachines:
  hosts:
    vm01:
      ansible_host: 192.0.2.50
    vm02:
      ansible_host: 192.0.2.51
    vm03:
      ansible_host: 192.0.2.52
webservers:
  hosts:
    webserver01:
      ansible_host: 192.0.2.140
      http_port: 80
    webserver02:
      ansible_host: 192.0.2.150
      http_port: 443
```

Para listarlo y hacer ping

```
ansible-inventory -i inventory.yaml --list
ansible virtualmachines -m ping -i inventory.yaml
```

Si queremos tener un archivo especifico en la msima carpeta, creamos un archivo ansible.cfg que contiene lo siguiente:

```
[defaults]

inventory = hosts
```

Y un archivo hosts al mismo nivel que contiene el mismo contenido que los anteriores.
### Playbooks

Ahora crearemos un archivo .yml con nuestros diferentes playbooks, un ejemplo:

``` yml
---
- name: Instalación y configuración de apache
	hosts: web
	become: yes
	remote_user: carlosmofe

tasks:

- name: Instala apache latest.
	apt:
		name: apache2
		update_cache: yes
		state: latest

- name: Cambiar el purto de escucha al 8081
	lineinfile: dest=/etc/apache2/ports.conf regexp="^Listen 80" line="Listen 8081" state=present
	notify:
		- restart apache2

handlers:
- name: restart apache2
	service:
		name: apache2
		state: restarted
```

La opción become se utiliza cuando vamos a usar sudo para cualquier cosa, con la opción 
-K en el comando nos pedirá la contraseña.

Para ejecutarlo:

``` sh
~/Escritorio/ansible$ ansible-playbook apache.yml -K
BECOME password: (Nos pide la contraseña del sudo)
PLAY [Instalación y configuración de apache] *********************************************************************************

TASK [Gathering Facts] *********************************************************************************
ok: [192.168.122.231]

TASK [Instala apache latest.] *********************************************************************************
changed: [192.168.122.231]

TASK [Cambiar el purto de escucha al 8081] ************************************************************************************************************************************************************************
changed: [192.168.122.231]

RUNNING HANDLER [restart apache2] *********************************************************************************
changed: [192.168.122.231]

PLAY RECAP *********************************************************************************
192.168.122.231            : ok=4    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
``` 

y en modo de check(no realiza los cambios, se usa como test antes de hacerlos)

**ansible-playbook --check playbook.yaml**

### Playbook más elaborado

En el siguiente playbook exportamos configuraciones de ftp a nuestro servidor, creamos un grupo y un usuario para uso, queda desarrollarlo más pero es un comienzo.

``` yml
---

- name: Instalación de vsftpd
hosts: web
become: yes
remote_user: carlosmofe

tasks:

- name: Instala vsftpd latest.
	apt:
		name: vsftpd
		update_cache: yes
		state: latest

- name: Crea grupo
	group:
		name: anpipada
		state: present

- name: Crea un usuario para el ftp
	ansible.builtin.user:
		name: anpipada
		comment: apnpiada
		uid: 1040
		group: anpipada

- name: Configuración vsftpd
	ansible.builtin.copy:
		src: ./ftp_conf/vsftpd.conf
		dest: /etc/vsftpd.conf

- name: Usuario vsftpd
	ansible.builtin.copy:
		src: ./ftp_conf/vsftpd.chroot_list
		dest: /etc/vsftpd.chroot_list

```

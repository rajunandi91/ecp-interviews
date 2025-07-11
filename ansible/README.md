# Ansible

## Complete the code so that it install the required packages based on the host os family and also shows the result.
### Inventory
```ini
[dev]
ubuntu-node ansible_host=192.168.10.10

[prod]
centos-node ansible_host=192.168.10.11

```
### Playbook Snippet
```yaml
- name: Install web server based on OS type
  hosts: all
  become: yes
  
  tasks:

    - name: Install nginx on Ubuntu
      apt:
        name: nginx
        state: present

    - name: Show nginx install result
      debug:
        msg: "nginx installation status: {{ nginx_install_result }}"

    - name: Install httpd on RedHat
      yum:
        name: httpd
        state: present

    - name: Show httpd install result
      debug:
        msg: "httpd installation status: {{ httpd_install_result }}"

```

## Evaluation
- On ubuntu-node, nginx should be installed using apt
- On centos-node, httpd should be installed using yum
- Code Solution
```yaml
- name: Install web server based on OS type
  hosts: all
  become: yes
  gather_facts: yes

  tasks:

    - name: Install nginx on Ubuntu
      apt:
        name: nginx
        state: present
        update_cache: yes
      when: ansible_facts['os_family'] == 'Debian'
      register: nginx_install_result

    - name: Show nginx install result
      debug:
        msg: "nginx installation status: {{ nginx_install_result }}"
      when: ansible_facts['os_family'] == 'Debian'

    - name: Install httpd on RedHat
      yum:
        name: httpd
        state: present
      when: ansible_facts['os_family'] == 'RedHat'
      register: httpd_install_result

    - name: Show httpd install result
      debug:
        msg: "httpd installation status: {{ httpd_install_result }}"
      when: ansible_facts['os_family'] == 'RedHat'
```

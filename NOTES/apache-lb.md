---
- name: Install Apache Load Balancer
  hosts: load_balancer_servers
  become: true

  tasks:
    - name: Update apt cache
      apt:
        update_cache: yes
      when: ansible_os_family == 'Debian'

    - name: Install Apache
      apt:
        name: apache2
        state: present
      when: ansible_os_family == 'Debian'

    - name: Install Apache
      yum:
        name: httpd
        state: present
      when: ansible_os_family == 'RedHat'

    - name: Enable Apache modules
      apache2_module:
        name: "{{ item }}"
        state: present
      with_items:
        - proxy
        - proxy_http
        - proxy_balancer
        - lbmethod_byrequests
      when: ansible_os_family == 'Debian'

    - name: Enable Apache modules
      command: a2enmod proxy proxy_http proxy_balancer lbmethod_byrequests
      when: ansible_os_family == 'RedHat'

    - name: Copy load balancer configuration file
      template:
        src: load_balancer.conf.j2
        dest: /etc/apache2/conf-available/load_balancer.conf
      when: ansible_os_family == 'Debian'

    - name: Copy load balancer configuration file
      template:
        src: load_balancer.conf.j2
        dest: /etc/httpd/conf.d/load_balancer.conf
      when: ansible_os_family == 'RedHat'

    - name: Enable load balancer configuration
      command: a2enconf load_balancer
      when: ansible_os_family == 'Debian'

    - name: Start and enable Apache service
      service:
        name: apache2
        state: started
        enabled: yes
      when: ansible_os_family == 'Debian'

    - name: Start and enable Apache service
      service:
        name: httpd
        state: started
        enabled: yes
      when: ansible_os_family == 'RedHat'

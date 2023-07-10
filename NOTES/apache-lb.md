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


Make sure to create a file called load_balancer.conf.j2 in the same directory as the playbook, with the following content:


apache
Copy code

```
<VirtualHost *:80>
    ServerName myloadbalancer.example.com

    <Proxy balancer://mycluster>
        BalancerMember http://backend1.example.com:80
        BalancerMember http://backend2.example.com:80
        # Add more BalancerMembers for additional backend servers

        ProxySet lbmethod=byrequests
    </Proxy>

    ProxyPass / balancer://mycluster/
    ProxyPassReverse / balancer://mycluster/
</VirtualHost>
```

In this example, the playbook assumes you have a group of servers defined in your Ansible inventory file under the name load_balancer_servers. Adjust the group name as per your inventory setup. The playbook installs Apache HTTP Server on the target servers, enables the necessary modules, copies the load balancer configuration file, and starts the Apache service.


To execute the playbook, save it to a file (e.g., install_load_balancer.yml) and run the following command:


css
Copy code
ansible-playbook -i inventory.ini install_load_balancer.yml

Replace inventory.ini with your actual inventory file.


Please note that this playbook assumes the target servers are running either Debian-based (e.g., Ubuntu) or Red Hat-based (e.g., CentOS) distributions. Adjust the package manager commands (apt and yum) and other tasks as per your target server distribution and configuration.



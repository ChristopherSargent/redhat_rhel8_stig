# Stage
1. ssh cas@172.18.0.92
2. sudo -i
3. cd /var/lib/awx/projects
4. git clone https://github.com/ansible-lockdown/RHEL8-STIG.git
5. mv RHEL8-STIG rhel8_stig
6. cd rhel8_stig/
7. cp defaults/main.yml defaults/main.yml.ORIG
8. vim defaults/main.yml 
* rhel8stig_bootloader_password_hash: grub.pbkdf2.sha512.changethispassword to rhel8stig_bootloader_password_hash: grub.pbkdf2.sha512.T3ll3r
9. vim site.yml
* add become_user: root as prelim ssd tasks fail with permission errors
```
---
- hosts: all  # noqa: name[play]
  become: true
  become_user: root

  roles:

      - role: "{{ playbook_dir }}"
```
# Ensure selinux is enforcing 
10. cp /etc/selinux/config /etc/selinux/config.ORIG
11. sed -i -e 's|SELINUX=disabled|SELINUX=enforcing|g' /etc/selinux/config
12. rebbot

# Update execution envirnoment to get this running
10. docker exec -it tools_awx_1 bash
11. su - awx 
12. podman run -itd quay.io/ansible/awx-ee bash
13. podman exec -u:0 charming_banach ansible-galaxy collection install community.crypto -p /usr/share/ansible/collections
```
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/community-crypto-2.22.1.tar.gz to /root/.ansible/tmp/ansible-local-114pi1qi0w/tmp5iedkcr6/community-crypto-2.22.1-6jdbxepr
Installing 'community.crypto:2.22.1' to '/usr/share/ansible/collections/ansible_collections/community/crypto'
community.crypto:2.22.1 was installed successfully
```
14. podman exec -u:0 charming_banach ansible-galaxy collection install community.general -p /usr/share/ansible/collections
```
Starting galaxy collection install process
Process install dependency map
Starting collection install process
Downloading https://galaxy.ansible.com/api/v3/plugin/ansible/content/published/collections/artifacts/community-general-9.4.0.tar.gz to /root/.ansible/tmp/ansible-local-155exvpiw7/tmpbvztdnkx/community-general-9.4.0-qrme9j6_
Installing 'community.general:9.4.0' to '/usr/share/ansible/collections/ansible_collections/community/general'
community.general:9.4.0 was installed successfully
```
15. podman commit charming_banach quay.io/ansible/awx-ee:10072024
16. Add ee to AWX UI

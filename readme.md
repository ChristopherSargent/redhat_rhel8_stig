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
12. reboot

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

# Oscap 
1. rpm --import https://dl.fedoraproject.org/pub/epel/RPM-GPG-KEY-EPEL-8
2. dnf install -y https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
3. dnf install -y scap-security-guide yum-utils ansible
4. mkdir -p /home/cas/oscap
5. cd /home/cas/oscap && oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_stig --results-arf /tmp/arf.xml --report /home/cas/oscap/rhel8test.pre.report.html --fetch-remote-resources --oval-results /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
* 49%
6. chown -R cas:cas /home/cas
7. awx > run redhat_rhel8_stig template
8. cd /home/cas/oscap && oscap xccdf eval --profile xccdf_org.ssgproject.content_profile_stig --results-arf /tmp/arf.xml --report /home/cas/oscap/rhel8test.post.report.html --fetch-remote-resources --oval-results /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
* 93%
9. oscap xccdf eval --report /home/cas/oscap/report1.html --profile stig /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
* 93%

# Generate fix playbook from results
8. cd /root && mkdir oscap && cd /root/oscap
9. oscap xccdf eval --profile stig --results stig-results.xml /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml
10. oscap xccdf generate fix --fetch-remote-resources --fix-type ansible --result-id "" /root/oscap/stig-results.xml > /root/oscap/stig-playbook-fix.yml
11. ansible-playbook -i localhost, -c local /root/oscap/stig-playbook-fix.yml
12. oscap xccdf eval --report /home/cas/oscap/report2.html --profile stig /usr/share/xml/scap/ssg/content/ssg-rhel8-ds.xml

# Manual things
14. systemctl mask --now kdump.service
15. yum install postfix -y 
16. cp /etc/ssh/sshd_config /etc/ssh/sshd_config.ORIG
17. sed -i -e 's|#PermitEmptyPasswords no|PermitEmptyPasswords no|g' /etc/ssh/sshd_config
18. sed -i -e 's|#PermitUserEnvironment no|PermitUserEnvironment no|g' /etc/ssh/sshd_config
19. systemctl restart sshd 
20. cp /etc/selinux/config /etc/selinux/config.ORIG
21. sed -i -e 's|SELINUX=disabled|SELINUX=enforcing|g' /etc/selinux/config
22. reboot

# Custom ee
23. docker login https://registry.redhat.io
24. docker pull registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:1.0.0-812
25. docker run -itd registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:1.0.0-812 bash
26. docker exec affectionate_fermat ansible-galaxy collection list
27. docker exec affectionate_fermat ansible-galaxy collection install community.crypto -p /usr/share/ansible/collections
28. docker exec affectionate_fermat ansible-galaxy collection install community.general -p /usr/share/ansible/collections
29. docker exec affectionate_fermat ansible-galaxy collection list
```
# /usr/share/ansible/collections/ansible_collections
Collection                      Version
------------------------------- -------
amazon.aws                      7.4.0  
ansible.controller              4.6.1  
ansible.eda                     2.1.0  
ansible.hub                     1.0.0  
ansible.netcommon               7.1.0  
ansible.network                 4.0.0  
ansible.platform                2.5.0  
ansible.posix                   1.5.4  
ansible.scm                     3.0.0  
ansible.security                3.0.0  
ansible.snmp                    3.0.0  
ansible.utils                   5.1.0  
ansible.windows                 1.14.0 
ansible.yang                    3.0.0  
arista.eos                      10.0.0 
cisco.asa                       6.0.0  
cisco.ios                       9.0.1  
cisco.iosxr                     10.1.0 
cisco.nxos                      9.2.1  
cloud.common                    3.0.0  
cloud.terraform                 2.0.0  
community.crypto                2.22.1 
community.general               9.5.0  
frr.frr                         2.0.2  
ibm.qradar                      4.0.0  
junipernetworks.junos           9.1.0  
kubernetes.core                 3.0.1  
microsoft.ad                    1.1.0  
openvswitch.openvswitch         2.1.1  
redhat.amq_broker               1.3.0  
redhat.amq_streams              1.0.0  
redhat.data_grid                1.3.1  
redhat.eap                      1.3.1  
redhat.insights                 1.2.2  
redhat.jbcs                     1.0.1  
redhat.jws                      2.0.0  
redhat.openshift                3.0.1  
redhat.openshift_virtualization 1.2.3  
redhat.redhat_csp_download      1.2.2  
redhat.rhbk                     2.2.2  
redhat.rhel_idm                 1.10.0 
redhat.rhel_system_roles        1.21.1 
redhat.rhv                      2.4.2  
redhat.runtimes_common          1.1.3  
redhat.sap_install              1.2.1  
redhat.satellite                3.10.0 
redhat.satellite_operations     1.3.0  
redhat.sso                      1.3.0  
sap.sap_operations              1.0.4  
servicenow.itsm                 2.6.3  
splunk.es                       4.0.0  
trendmicro.deepsec              4.0.0  
vmware.vmware                   1.3.0  
vmware.vmware_rest              3.0.1  
vyos.vyos                       4.0.2 
```
30. docker commit affectionate_fermat registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:10102024
31. docker save -o ee-supported-rhel8_10102024.tar registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:10102024
32. cp ee-supported-rhel8-custom_1092024.tar /root/awx-ees/
33. docker cp ee-supported-rhel8_10102024.tar tools_awx_1:awx_devel
34. docker exec -it tools_awx_1 bash
35. su - awx
36. podman load -i /awx_devel/ee-supported-rhel8_10102024.tar 
37. podman run -itd registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:10102024 bash
38. podman exec heuristic_mendel ansible-galaxy collection list
39. https://swc-awx01.cas.local > Administration > Execution Enviornments > New > Save
```
Name  = ee-supported-rhel8-10102024
Image = registry.redhat.io/ansible-automation-platform-25/ee-supported-rhel8:10102024
Pull  = Always
```

# References
* https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/8/html/system_design_guide/scanning_the_system_for_security_compliance_and_vulnerabilities#scanning-the-system-with-a-customized-profile-using-scap-workbench_system-design-guide
* https://redhatgov.io/workshops/rhel_8/exercise1.7/
* https://access.redhat.com/RegistryAuthentication
* 



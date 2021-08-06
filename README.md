# README #

Ansible role to manage a self-signed PKI environment.
 
## What is this repository for? ##

* Create/Delete CA on a localhost directry
* Create/Delete certificate/keys on localhost directory
* Install/Uninstall CA-cert on remote hosts
* Install/Uninstall server certifificate/key on remote hosts

Note: For special usecases, the steps listed above can be used individually. Examles are described below.

Supported Plattforms:

* CentOS
* RedHat
* Debian
* MacOS

## How do I get set up? ##

Create a playbook and import/include the role. The easiest way is to use galaxy.
```
# vi requirements.yml

# ansible-role-pki
- src: git+https://github.com/realstuff/ansible-role-pki.git
  version: master
  name: securix.pki
```
```
ansible-galaxy install -r requirements.yml --force
```

## Configuration ##

### State ###
```pki_state``` Present(default): Used to create/install, absent: Used to delete/uninstall.
Note: pki state has no effect when single steps are executed (example: tasks_from: install-cert.yml)

### Standard configurations ###
#### State present ####
```
- name: PKI, make sure CA cert and key exist, create certs/keys for all servers and installs CA-Cert and server cert/key pairs on servers
  hosts: all
  roles:
    - securix.pki
  vars:
    # PKI CA certs
    pki_ca_key_password: "{{ vault_password_ca_key }}"
    pki_ca_properties:
      country_name: CH
      state_or_province_name: BE
      locality_name: Berne
      organization_name: ACME.inc
      organizational_unit_name: Test
      common_name: test.acme.com
      email_address: test@acme.com
      subject_alt_name:
        - DNS: test.acme.com

    # PKI Certificates
    pki_cert_properties:
      country_name: CH
      state_or_province_name: BE
      locality_name: Berne
      organization_name: ACME.inc
      organizational_unit_name: DevOps
      common_name: "{{inventory_hostname}}"
      email_address: info@acme.com
      subject_alt_name:
        - "DNS:{{inventory_hostname}}"
        - "IP:{{ansible_default_ipv4.address}}"
```

#### State absent ####
Note: By default, local CA cert and key are not uninstalled. Use ```pki_ca_force: yes``` to delete them as well 
```
- name: PKI, Uninstall and delete certs/keys and delete them from local cirectory.
  hosts: all
  roles:
    - securix.pki
  vars:
    # Force CA uninstall
    #pki_ca_force: yes

    # Uninstall and delete
    pki_state: absent
```

### Custom configurations ###

### Create CA ###
Creates a CA cert and key in a directory on localhost. The following settings are used.       
Variables:

* ```pki_ca_force``` **Default:** ***no*** Set to to yes to override exising key and certificate
* ```pki_ca_src_path``` **Default:** ***playbook_dir/files/ssl/CA*** Directory of local CA-cert and CA-key
* ```pki_ca_name``` **Default:** ***pki_ca_properties.common_name***
* ```pki_ca_cert_ext``` **Default:** ***crt*** The extensions of the certificate
* ```pki_ca_key_ext``` **Default:** ***key*** The extensions of the key    
* ```pki_ca_key_password``` The password of the CA-key    
* ```pki_ca_cipher_type``` **Default:** ***auto***    
* ```pki_ca_cipher_size``` **Default:** ***4096***    
* ```pki_ca_expire_after_days``` **Default:** ***1095***    
* ```pki_ca_properties```    
    * ```country_name``` Short name of Country, e.g. CH    
    * ```state_or_province_name``` Short name of province, e.g. BE    
    * ```locality_name``` City, e.g. Berne    
    * ```organization_name``` Company name    
    * ```organizational_unit_name``` Name of the OU    
    * ```common_name``` Common name of the certificate. E.g. acme.com    
    * ```email_address``` Email-Contact    
    * ```subject_alt_name``` List of alternative names. At least value of common name. See example of **State present**     

Example:    
```
- name: tasks_from examples
  hosts: all
  tasks:
    - name: Create CA
      include_tasks: securix.pki
      tasks_from: create-ca
      vars:
        # PKI CA certs
        pki_ca_name: intra.acme.ch
        pki_ca_key_password: "{{ vault_password_ca_key }}"
        pki_ca_cipher_type: auto
        pki_ca_cipher_size: 4096
        pki_ca_expire_after_days: 1095
        pki_ca_properties:
        country_name: CH
        state_or_province_name: BE
        locality_name: Berne
        organization_name: ACME.inc
        organizational_unit_name: Test
        common_name: intra.acme.com
        email_address: info@acme.com
        subject_alt_name:
          - DNS: intra.acme.com
```

### Create server certificate/key ###
Creates a server certificate/key pair for each server in the host list of the play and stores it on localhost.    
Variables:    

* ```pki_cert_force``` **Default:** ***no*** Set to to yes to override exising key and certificate
* ```pki_cert_src_path``` **Default** ***playbook_dir/files/ssl/certs*** Directory of local certificate and key
* ```pki_cert_src_group```**Default** ***staff*** Group of the local certificate and key
* ```pki_cert_src_key_mode```**Default** ***0644*** Mode of the local key
* ```pki_cert_name``` **Default** ***inventory_hostname***
* ```pki_cert_cert_ext```: **Default** ***crt*** The extensions of the certificate
* ```pki_cert_key_ext```: **Default** ***key*** The extensions of the key
* ```pki_cert_key_password``` The password of the key. Optional
* ```pki_cert_key_cipher``` The cipher of the key. Optional
* ```pki_cert_expire_after_days``` **Default** ***365*** Exiration time
* ```pki_cert_properties```
    * ```country_name``` Short name of Country, e.g. CH
    * ```state_or_province_name``` Short name of province, e.g. BE
    * ```locality_name``` City, e.g. Berne
    * ```organization_name``` Company name
    * ```organizational_unit_name``` Name of the OU
    * ```common_name``` Common name of the certificate. E.g. test.acme.com
    * ```email_address``` Email-Contact
    * ```subject_alt_name``` List of alternative names. At least value of common name. See example of **State present**

Example:
```
- name: Create certificate
  include_tasks: securix.pki
  tasks_from: create-cert.yml
  vars:
    # PKI Certificates
    pki_cert_cipher_size: 4096
    pki_cert_expire_after_days: 825
    pki_cert_properties:
    country_name: CH
    state_or_province_name: BE
    locality_name: Berne
    organization_name: ACME.inc
    organizational_unit_name: DevOps
    common_name: "{{inventory_hostname}}"
    email_address: info@acme.com
    subject_alt_name:
      - "DNS:{{inventory_hostname}}"
      - "IP:{{ansible_default_ipv4.address}}"
```

### Install CA certificate ###
Installs the CA certificate on the remote hosts    
Direct use: ```tasks_from: "{{ansible_os_family}}/install-cacert.yml}}"``` Usage examples see below    
Variables:

* ```pki_ca_name``` **Default** ***pki_ca_properties.common_name*** The filename of the certificate
* ```pki_ca_cert_dst_path``` **Default: OS specific, see below** The destination directory
* ```pki_ca_cert_ext```: **Default:** ***crt*** The extensions of the certificate
* ```pki_ca_cert_mode``` **Default:** ***0644***  The mode of the installed certificarte
* ```pki_ca_cert_owner``` **Default: inherited from ansible remote_user** The owner of the file
* ```pki_ca_cert_group``` **Default: inherited from ansible remote_user** The group of the file 

**Darwin**
Not used. Certificate it added to the keystore and file deleted afterwards.

**Debian**
pki_ca_cert_dst_path: "/usr/local/share/ca-certificates/{{pki_ca_name}}.{{pki_ca_cert_ext}}"

**RedHat***
pki_ca_cert_dst_path: "/etc/pki/ca-trust/source/anchors/{{pki_ca_name}}.{{pki_ca_cert_ext}}"

Example:    
```
- name: Install custom ca-cert
  include_tasks: securix.pki
  tasks_from: "{{ansible_os_family}}/install-cacert.yml"
  vars:
    pki_ca_name: acme.com
```

### Install Server cert/key ###
Installs the server certificate/key on the remote host     
Variables:

* ```pki_cert_name``` **Default** ***pki_ca_properties.common_name*** The filename of the certificate
* ```pki_cert_cert_dst_path``` **Default:** ***OS specific, see below** The destination directory of the key
* ```pki_cert_cert_ext```: **Default:** ***crt*** The extensions of the certificate
* ```pki_cert_cert_mode``` **Default:** ***0644***  The mode of the installed certificarte
* ```pki_cert_cert_owner``` **Default: inherited from ansible remote_user** The owner of the cert
* ```pki_cert_cert_group``` **Default: inherited from ansible remote_user** The group of the cert
* ```pki_cert_key_dst_path``` **Default:** ***OS specific, see below** The destination directory of the key
* ```pki_cert_key_ext```: **Default:** ***crt*** The extensions of the key
* ```pki_cert_key_mode``` **Default:** ***0644***  The mode of the installed key
* ```pki_cert_key_owner``` **Default: inherited from ansible remote_user** The owner of the key
* ```pki_cert_key_group``` **Default: inherited from ansible remote_user** The group of the key 

**Darwin**
pki_cert_cert_dst_path: "/etc/ssl/certs/{{pki_cert_name}}.{{pki_cert_cert_ext}}"
pki_cert_key_dst_path: "/etc/ssl/certs/{{pki_cert_name}}.{{pki_cert_key_ext}}"

**Debian**
pki_cert_cert_dst_path: "/etc/ssl/localcerts{{pki_ssl_cert_name}}.{{pki_ssl_cert_cert_ext}}"
pki_cert_key_dst_path: "/etc/ssl/localcerts/{{pki_cert_name}}.{{pki_cert_key_ext}}"

**RedHat***
pki_cert_cert_dst_path: "/etc/pki/tls/certs/{{pki_cert_name}}.{{pki_cert_cert_ext}}"
pki_cert_key_dst_path: "/etc/pki/tls/private/{{pki_cert_name}}.{{pki_cert_key_ext}}"

Example:    
```
- name: Install custom-certificate
  include_tasks: securix.pki
  tasks_from: "{{ansible_os_family}}/install-cert.yml"
  vars:
    pki_cert_name: star.acme.com
```

### Delete CA ###  
Deletes a CA cert and key on localhost.
Variables:

* ```pki_ca_src_path``` **Default: *playbook_dir/files/ssl/CA*** Directory of local CA-cert and CA-key
* ```pki_ca_name``` **Default: *pki_ca_properties.common_name***
* ```pki_ca_cert_ext```: **Default: *crt*** The extensions of the certificate
* ```pki_ca_key_ext```: **Default: *key*** The extensions of the key

Example:    
```
- name: Delete CA
  include_tasks: securix.pki
  tasks_from: "delete-ca.yml"
  vars:
    pki_ca_force: yes
    pki_cert_name: acme.com
```


### Delete Certificate/key ###
Deletes a server certificate and key on localhost.    
Direct use: ```tasks_from: delete-cert.yml```  
Variables:    
* ```pki_cert_src_path``` **Default: *playbook_dir/files/ssl/certs*** Directory of local certificate and key
* ```pki_cert_name``` **Default: *inventory_hostname***
* ```pki_cert_cert_ext```: **Default: *crt*** The extensions of the certificate
* ```pki_cert_key_ext```: **Default: *key*** The extensions of the key

```
- name: Delete custom certificate/key
  include_tasks: securix.pki
  tasks_from: "delete-cert.yml"
  vars:
    pki_cert_name: star.acme.com
```

### Uninstall CA certificate ###
Uninstalls the CA certificate on the remote hosts    
Direct use: ```tasks_from: "{{ansible_os_family}}/uninstall-cacert.yml"```    
Variables:

* ```pki_ca_name``` **Default:** ***pki_ca_properties.common_name*** The filename of the certificate
* ```pki_ca_cert_dst_path``` **Default:** ***OS specific, see below** The destination directory
* ```pki_ca_cert_ext```: **Default:** ***crt*** The extensions of the certificate (Not used on Darwin)

**Darwin**
Not used. Certificate it added to the keystore and file deleted afterwards.

**Debian**
pki_ca_cert_dst_path: "/usr/local/share/ca-certificates/{{pki_ca_name}}.{{pki_ca_cert_ext}}"

**RedHat***
pki_ca_cert_dst_path: "/etc/pki/ca-trust/source/anchors/{{pki_ca_name}}.{{pki_ca_cert_ext}}"

Example:    
```
- name: Uninstall CA
  include_tasks: securix.pki
  tasks_from: "{{ansible_os_family}}/uninstall-cacert.yml"
```


### Uninstall certificate/key ###
Unnstalls the certificate/key on the remote hosts    
Direct use: ```tasks_from: uninstall-cert.yml``` Usage examples see below    
Variables:    

* ```pki_cert_name``` **Default:** ***pki_ca_properties.common_name*** The filename of the certificate
* ```pki_cert_cert_dst_path``` **Default: OS specific, see below** The destination directory of the key
* ```pki_cert_cert_ext```: **Default:** ***crt*** The extensions of the certificate
* ```pki_cert_key_dst_path``` **Default: OS specific, see below** The destination directory of the key
* ```pki_cert_key_ext```: **Default:** ***crt*** The extensions of the key

**Darwin**
pki_cert_cert_dst_path: "/etc/ssl/certs/{{pki_cert_name}}.{{pki_cert_cert_ext}}"
pki_cert_key_dst_path: "/etc/ssl/certs/{{pki_cert_name}}.{{pki_cert_key_ext}}"

**Debian**
pki_cert_cert_dst_path: "/etc/ssl/localcerts{{pki_ssl_cert_name}}.{{pki_ssl_cert_cert_ext}}"
pki_cert_key_dst_path: "/etc/ssl/localcerts/{{pki_cert_name}}.{{pki_cert_key_ext}}"

**RedHat***
pki_cert_cert_dst_path: "/etc/pki/tls/certs/{{pki_cert_name}}.{{pki_cert_cert_ext}}"
pki_cert_key_dst_path: "/etc/pki/tls/private/{{pki_cert_name}}.{{pki_cert_key_ext}}"

Example:    
```
- name: Uninstall certificate
  include_tasks: securix.pki
  tasks_from: "{{ansible_os_family}}/uninstall-cert.yml"
```


## Contribution guidelines ##

Any comments, contributions and documentation welcome!

## TODO / Roadmap ##
* Improve README (Ansible Style)
* Support other platforms
  * Docker
  * Windows
  * Suse
  * Fedora

## Who do I talk to? ##

* Repo owner or admin bernhard.fluehmann@securix.ch
* Other community or team contact mathew.thekkekara@securix.ch


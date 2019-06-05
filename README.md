Certificate Authority keys
==========================

This is an Ansible role for generating private CA keys and signed certificates. These can be used to secure 
server/client applications, where both encryption of data in transit and authentication between nodes is required. 

When configured properly, keys signed by a private CA can prevent MITM attacks, and can be used for 2-way 
authentication. An example might be securing the connections between nodes in a Kafka or Elastic Stack cluster.

The role follows these rough steps:

- Generate a CA (Certificate Authority) keypair
- Generate keys and CSRs (Certificate Signing Requests) for specified nodes
- Sign the CSRs using the CA, to create signed certificates (CRTs)
- Optionally package the necessary keys into java keystore/truststore binaries


Defaults
--------

Check out all the defaults [here](defaults/main.yml) and override them as needed.


Usage
-----

Recommended usage: run the role in a local playbook (see below), then move the generated files somewhere safe. Don't 
leave the keys lying around. This code is provided as-is, how you use it and store the resulting keys is your 
responsibility! 

This role can be used to create new keys and CRTs from an existing CA certificate if one is present.

After they're created, move the required files into whatever project you're using them in. You can use
[Ansible Vault](https://docs.ansible.com/ansible/latest/user_guide/vault.html) to encrypt the files (including jks
binaries), and Ansible modules like `copy` will decrypt them during upload if supplied with the vault password.

Checking the keys
-----------------

You can (and should) check the keys with various `openssl` commands. Here are some examples:

```
openssl x509 -in generated_keys/CA.crt -text -noout
openssl rsa -in generated_keys/<key-name>.key -check
openssl x509 -in generated_keys/<key-name>-signed.crt -text -noout
```

Python limitation
-----------------

Due to an Ansible bug (fix now merged in `devel` branch), you have to use Python 2.7 when running the tasks to create 
java keystores. See [this issue](https://github.com/ansible/ansible/pull/51951) for details.

Example playbook
----------------

```
- name: Generate Keys
  hosts: 127.0.0.1
  connection: local
  
  vars:
    ansible_python_interpreter: '/usr/bin/python2.7'

  roles:
    - role: libre_ops.ca_keys
      vars:
        cert_organisation: Example Inc.
        cert_unit: Keys Department
        cert_country: FR
        cert_state: Paris
        cert_location: Paris
        
        create_keys:
          - filename: server
            subject:
              - "/CN=app.client.org"
              - "/O={{ cert_organisation }}"
              - "/OU={{ cert_unit }}"
              - "/C={{ cert_country }}"
              - "/ST={{ cert_state }}"
              - "/L={{ cert_location }}"
              
          - filename: client
            subject:
              - "/CN=app.server.org"
              - "/O={{ cert_organisation }}"
              - "/OU={{ cert_unit }}"
              - "/C={{ cert_country }}"
              - "/ST={{ cert_state }}"
              - "/L={{ cert_location }}"    
```

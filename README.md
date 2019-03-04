# Ansible playbooks to create and use CAs, Intermediate/SSL CAs and new certs and signing CSRs

When I setup labs and POC environments, I often need or want to test with a functioning SSL environment.  These playbooks are evolving to help me manage a Self Signed Root CA and Intermediate SSL CA environment.  

You can create multiple CA environments, which are trees in the tls subdirectory, and protected by .gitignore.

When signing certificates / CSRs, you have the opportunity to create:
* server certs
* client certs
* combo server+client certs (OneView needed this)

## Requirements

* [x] openssl installed on local system

## Tested on

* [x]  MacOS Mojave 10.14.3 with Homebrew openssl: stable 1.0.2q
* [x]  CentOS7 Linux release 7.6.1810 (Core) with OpenSSL 1.0.2k-fips


## Structure

```
tls
└── domain.tld
    ├── root-ca                   Root CA information
    │   ├── certs
    │   │   └── ca.crt.pem        CA public certificate
    │   ├── crl
    │   │   └── ca.crl.pem        List of revoked keys (publish)
    │   ├── csr
    │   ├── index.txt             Registered certificates
    │   ├── index.txt.attr
    │   ├── newcerts              Copy of each certificate signed
    │   │   └── 1000.pem
    │   ├── openssl.cnf           CA configuration
    │   ├── private
    │   │   └── ca.key.pem        CA private key (keep safe)
    │   └── serial                Next serial number
    ├── ssl-ca
    │   ├── certs
    │   │   ├── ca.crt.pem        CA public certificate
    │   │   └── chain.crt.pem     Int+Root CA public certificate (distribute)
    │   ├── crl
    │   │   └── ca.crl.pem        List of revoked keys (publish)
    │   ├── crlnumber             Next CRL number
    │   ├── csr
    │   │   └── ca.csr
    │   ├── index.txt             Registered certificates
    │   ├── index.txt.attr
    │   ├── newcerts              Copy of each certificate signed
    │   ├── openssl.cnf           CA configuration
    │   ├── private
    │   │   └── ca.key.pem        CA private key (keep safe)
    │   └── serial                Next serial number
    └── store
        └── host.domain.tld
            └── YYYY-MM-DD
                ├── openssl_csr_san.cnf           Host configuration
                ├── host.domain.tld.crt.pem       Host certificate
                ├── host.domain.tld.csr           Host signing request
                ├── host.domain.tld.key.pem       Host key
                └── host.domain.tld.keycrt.pem    Host key+certificate combo

```

## Prep

### Create ansible vault protected variables for the secrets to the Root CA key and the Intermediate/SSL CA key

#### Generate ansible vault secret

Note, the ansible.cfg in the directory assumes your ansible vault password is in `.vaultpass`

`$ ansible-vault encrypt_string --name root_passphrase 'my-super-secret'`
```
root_passphrase: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          62303038373766323261323532333765656433303332346263666237666464623934323430326633
          3636373032613131323566383331336332353563373561640a343239346634343361616136326235
          35646639666166333066666432613065636135396630646231623930343339643639636465633732
          3632383333613666360a633865373366353638373062343836386266313131373035643932646337
          6339
Encryption successful
```

#### Insert ansible vault variables into `hosts.yml`
```
---
all:
  hosts:
    localhost:
  vars:
    root_passphrase: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      62303038373766323261323532333765656433303332346263666237666464623934323430326633
      3636373032613131323566383331336332353563373561640a343239346634343361616136326235
      35646639666166333066666432613065636135396630646231623930343339643639636465633732
      3632383333613666360a633865373366353638373062343836386266313131373035643932646337
      6339
    intermediate_passphrase: !vault |
      $ANSIBLE_VAULT;1.1;AES256
      33323035396661376265313762306636666161376237353938333966653630313465316562366332
      6339383035626234373733363362663932323538656461650a643432306139346130613061383836
      65376138373663363234323034363936306334356635613832376534303964333230303536326361
      6162346138336266380a656165393935366330383864623132313361373032653665393033663566
      63313165356165643334646261373935376538386539333033626630616564663734
```


## Usage

### Create a CA

`$ ansible-playbook create-ca.yml`
```
Enter domain [domain.tld]:
Enter CA common name (Private CA will be appended automatically) [My Company]:
Enter the CA organization [My Company]:
Enter the CA organizational unit [Infrastructure Services]:
Enter the CA country (2 letters) [US]:
Enter the CA state [State]:
Enter the CA locality [City]:
Enter the CA duration in days [3650]:
Enter the CRL CA duration in days [30]:
Enter the CA email [admin@domain.tld]:

PLAY [localhost] ***********************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************
ok: [localhost]

TASK [set_fact] ************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Create directory structure] ******************************************************************************************************************************************************************************
changed: [localhost] => (item=['root-ca', 'certs'])
changed: [localhost] => (item=['root-ca', 'crl'])
changed: [localhost] => (item=['root-ca', 'newcerts'])
changed: [localhost] => (item=['root-ca', 'csr'])
changed: [localhost] => (item=['ssl-ca', 'certs'])
changed: [localhost] => (item=['ssl-ca', 'crl'])
changed: [localhost] => (item=['ssl-ca', 'newcerts'])
changed: [localhost] => (item=['ssl-ca', 'csr'])

TASK [Secure private directory] ********************************************************************************************************************************************************************************
changed: [localhost] => (item=root-ca)
changed: [localhost] => (item=ssl-ca)

TASK [Create store directory] **********************************************************************************************************************************************************************************
changed: [localhost]

TASK [Init registered certifiates] *****************************************************************************************************************************************************************************
changed: [localhost] => (item=['root-ca', 'index.txt'])
changed: [localhost] => (item=['root-ca', 'index.txt.attr'])
changed: [localhost] => (item=['ssl-ca', 'index.txt'])
changed: [localhost] => (item=['ssl-ca', 'index.txt.attr'])

TASK [Set serial for Root CA] **********************************************************************************************************************************************************************************
changed: [localhost]

TASK [Set serial for Intermediate CA] **************************************************************************************************************************************************************************
changed: [localhost]

TASK [Set CRL for Intermediate CA] *****************************************************************************************************************************************************************************
changed: [localhost]

TASK [Install Root CA openssl.cnf from template] ***************************************************************************************************************************************************************
changed: [localhost]

TASK [Install Intermediate CA openssl.cnf from template] *******************************************************************************************************************************************************
changed: [localhost]

TASK [Create Root CA private key] ******************************************************************************************************************************************************************************
changed: [localhost]

TASK [Create self-signed Root CA] ******************************************************************************************************************************************************************************
changed: [localhost]

TASK [Create private key and CSR for the Intermediate CA] ******************************************************************************************************************************************************
changed: [localhost]

TASK [Create the intermediate certificate] *********************************************************************************************************************************************************************
changed: [localhost]

TASK [Create the certificate chain] ****************************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *****************************************************************************************************************************************************************************************************
localhost                  : ok=16   changed=14   unreachable=0    failed=0
```

### Create a server certificate by creating a CSR and signing with the Intermediate SSL CA

`$ ansible-playbook create-sign-cert.yml`
```
Enter CA domain [domain.tld]:
Enter the certificate organization [My Company]:
Enter the certificate organizational unit [Infrastructure Services]:
Enter the certificate country [US]:
Enter the certificate state [State]:
Enter the certificate locality [City]:
Enter the certificate duration in days [3650]:
Enter server names (FQDN list separated by commas) [vip.domain.tld,www.domain.tld]:
Enter IPs (list separated by commas) [192.168.100.100]:
Server cert (true/false) [True]:
Client cert (true/false) [False]:

PLAY [localhost] ***********************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************
ok: [localhost]

TASK [set_fact] ************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [set_fact] ************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [set_fact] ************************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Create directory structure] ******************************************************************************************************************************************************************************
changed: [localhost]

TASK [Install Intermediate CA openssl.cnf from template] *******************************************************************************************************************************************************
changed: [localhost]

TASK [Create private key and CSR for the server] ***************************************************************************************************************************************************************
changed: [localhost]

TASK [Create the server certificate] ***************************************************************************************************************************************************************************
changed: [localhost]

TASK [Create the client certificate] ***************************************************************************************************************************************************************************
skipping: [localhost]

TASK [Create the server+client certificate] ********************************************************************************************************************************************************************
skipping: [localhost]

TASK [Create combined certificate] *****************************************************************************************************************************************************************************
changed: [localhost]

PLAY RECAP *****************************************************************************************************************************************************************************************************
localhost                  : ok=9    changed=5    unreachable=0    failed=0
```

### Sign a CSR generated by another process, like OneView or iLO

The playbook first examines the CSR to see if you want to sign it.
Exit with a 'Ctrl-C' + 'A'
'Enter/Return' to continue

`$ ansible-playbook sign-csr.yml`
```
Enter CA domain [domain.tld]:
Enter the CSR filename with path [server.csr]:
Enter the certificate duration in days [3650]:
Server cert (true/false) [True]:
Client cert (true/false) [False]:

PLAY [localhost] ***********************************************************************************************************************************************************************************************

TASK [Gathering Facts] *****************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Set TLS root] ********************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Verify CSR exists] ***************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Get text from CSR] ***************************************************************************************************************************************************************************************
changed: [localhost]

TASK [Confirm signing of CSR] **********************************************************************************************************************************************************************************
[Confirm signing of CSR]
Certificate Request:
    Data:
        Version: 0 (0x0)
        Subject: CN=server, OU=Infrastructure Services, O=My Company, L=City, ST=State, C=US
        Subject Public Key Info:
            Public Key Algorithm: rsaEncryption
                Public-Key: (2048 bit)
                Modulus:
                    00:cd:1f:b6:a2:d9:1e:2b:0e:3e:98:40:b3:74:ee:
                    2f:5b:a0:0f:e4:0c:ab:fb:49:33:1e:42:c8:a9:c5:
                    0a:0a:01:32:48:46:45:83:29:9a:72:a5:b4:a8:ef:
                    74:cb:c7:16:cf:fb:43:f2:e8:3f:a4:3a:00:89:fc:
                    52:d5:29:e2:a1:e6:63:41:37:b2:5e:42:98:37:3d:
                    2e:67:ae:7a:24:cc:f1:1b:13:82:e9:62:41:6f:d1:
                    af:8f:f9:db:3e:e3:36:b5:ce:08:ed:94:98:4a:ac:
                    98:94:48:47:92:64:21:ce:08:bb:79:c0:ac:98:1c:
                    82:63:e5:aa:2e:27:c2:12:97:46:6e:e6:61:34:66:
                    41:cd:ca:35:d2:0f:15:4f:e4:fc:0f:bf:03:d5:f9:
                    96:66:dc:d3:a9:71:da:57:16:f5:fd:0f:f1:05:4f:
                    30:07:e2:0e:19:2c:f3:0d:a3:76:f2:dd:82:f8:34:
                    26:bf:91:e4:7f:2b:13:c9:e2:5d:a6:95:df:19:55:
                    d6:7a:65:86:ff:df:b6:1c:07:45:3b:99:5c:7a:07:
                    3c:c5:b0:7c:ad:1a:2b:b0:40:9b:35:ea:aa:7e:96:
                    8a:32:b7:a8:2f:cc:ec:2e:5e:d1:8d:00:5c:5a:00:
                    ea:e0:a9:2d:eb:a1:ad:40:0f:c3:2f:bc:a8:01:a0:
                    de:2d
                Exponent: 65537 (0x10001)
        Attributes:
        Requested Extensions:
            X509v3 Subject Alternative Name:
                DNS:server, IP Address:10.10.10.10, IP Address:FE80:0:0:0:FFFF:FFFF:FFFF:FFFF, IP Address:2607:FFFF:0:1:FFFF:0:0:1
    Signature Algorithm: sha256WithRSAEncryption
         5b:42:80:2e:7f:88:4e:1f:fe:5c:9b:f8:88:6a:e0:a9:21:46:
         d1:4f:b9:f2:a4:4d:fd:99:3d:af:a3:9a:da:e1:5d:af:ec:0e:
         87:0c:e0:a6:c0:19:eb:64:46:9c:ab:7b:50:4d:3f:3e:f6:76:
         10:04:94:4e:f5:0e:74:a9:03:8b:fc:dc:dd:b3:72:5e:e6:87:
         93:84:4c:47:89:21:aa:08:8e:c1:c1:3e:94:08:e7:55:4c:ee:
         33:66:96:03:f0:d2:88:61:99:1d:a2:d7:f3:2a:ce:aa:e7:72:
         08:03:5a:e2:46:a4:aa:ba:ef:07:df:00:53:b0:9f:63:34:1c:
         98:f0:d8:44:74:65:36:19:8c:0d:43:40:c0:56:3e:ed:ca:32:
         83:27:08:a3:75:5a:8e:6e:a2:8e:da:43:e4:45:74:3a:d5:ef:
         4e:43:1a:f8:86:c3:55:2f:b8:71:f1:5e:f2:84:ac:30:54:ec:
         7e:eb:8a:da:2d:52:94:aa:fb:f3:0a:5a:1b:60:e8:e5:4f:dd:
         ec:a9:fb:06:c3:a5:d7:39:77:e0:21:1e:b5:1d:e8:f2:e3:ad:
         0e:a8:c7:1c:e3:c1:de:ba:01:32:e1:46:e3:36:aa:1d:e6:d2:
         da:f1:61:11:7c:09:60:41:5c:ee:6b:15:74:13:56:2f:68:a1:
         6e:96:1f:bc

Please press 'Ctrl-C' 'A' to abort
Please press Enter to continue
:
ok: [localhost]

TASK [Get subject from CSR] ************************************************************************************************************************************************************************************
changed: [localhost]

TASK [Extract CN from CSR subject] *****************************************************************************************************************************************************************************
ok: [localhost]

TASK [Set Host root] *******************************************************************************************************************************************************************************************
ok: [localhost]

TASK [Create directory structure] ******************************************************************************************************************************************************************************
changed: [localhost]

TASK [Copy CSR to TLS directory] *******************************************************************************************************************************************************************************
changed: [localhost]

TASK [Create the server certificate] ***************************************************************************************************************************************************************************
changed: [localhost]

TASK [Create the client certificate] ***************************************************************************************************************************************************************************
skipping: [localhost]

TASK [Create the server+client certificate] ********************************************************************************************************************************************************************
skipping: [localhost]

PLAY RECAP *****************************************************************************************************************************************************************************************************
localhost                  : ok=11   changed=5    unreachable=0    failed=0
```

## Ideas to develop

* [ ]  Automate webserver configuration for certificate revocation lists
* [ ]  Handle the certificate revocation process
* [ ]  Handle certificate renewals


## Credit to blogs and ideas

* http://dadhacks.org/2017/12/27/building-a-root-ca-and-an-intermediate-ca-using-openssl-and-debian-stretch/
* http://radiac.net/blog/2015/05/self-ca/

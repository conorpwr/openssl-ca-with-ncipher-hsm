# openssl-ca-with-ncipher-hsm

These are some basic notes on running an OpenSSL CA on top of a nCipher HSM. These are intended to be hints for anyone doing something similar and not a complete set up guide. Use at your own risk.

# Assumptions
1. Working Security World and HSMs with an existing pkcs11 key
1. You have the nCipher Compatibility pack installed

# Other places to read/check out
1. https://ncipher.zendesk.com/hc/en-us/articles/360003413317-HowTo-Integrate-OpenSC-LIBP11-OS-Linux-with-openSSL-and-nShield-HSMs
1. https://roll.urown.net/ca/ca_root_setup.html

# Steps
1. Ensure you have a combination of the openssl.cnf built to suit your needs based on the above section

# Sample commands

These commands assume you are using an OCS protected pkcs11 key. If you are just using softkey protection then running OpenSSL directly *should* work but I've not tested it fully.

```bash
Generate a CRL:

/opt/nfast/bin/preload -c ${NFAST_PRELOAD_CARDSET} /usr/bin/openssl ca -gencrl -engine ${OPENSSL_ENGINE} -keyform ${OPENSSL_KEYFORM} -keyfile ${OPENSSL_PRIVATE_KEY} -out "crl/${TIMESTAMP}.crl"

Sign an intermediate CA (intermed-ca_ext needs to be defined in openssl.cnf):

/opt/nfast/bin/preload -c ${NFAST_PRELOAD_CARDSET} /usr/bin/openssl ca -engine ${OPENSSL_ENGINE} -in ${SUBCA_CSR} -extensions intermed-ca_ext -keyform ${OPENSSL_KEYFORM} -keyfile ${OPENSSL_PRIVATE_KEY} -rand_serial
```

$NFAST_PRELOAD_CARDSET == the friendly name for the OCS cardset
$OPENSSL_ENGINE == pkcs11 (or whatever engine you are using)
$OPENSSL_KEYFORM == engine
$OPENSSL_PRIVATE_KEY == pkcs11 path to your key

# Getting PKCS11 path

With an OCS smartcard inserted in the HSM:

```
p11tool --list-all --provider=/opt/nfast/toolkits/pkcs11/libcknfast.so
```

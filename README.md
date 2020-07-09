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
1. Define the OPENSSL_CONF environment variable to be your configuration file created above

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

# Troubleshooting
## Preload asks for a password for the smartcard and OpenSSL asks again for a password?
This is expected. Preload will temporarily load the entire cardset. OpenSSL should just ask for the password of the smartcard currently inserted into the HSM

## Smartcard password incorrect even if you are sure it is correct
I've only seen this happen with an OCS cardset where a persistence timeout is configured. Remove the smartcard, re-insert and try again.

## OpenSSL doesn't like the PKCS11 key?
Double check if you are using key vs keyfile arguments (you should be using keyfile). OpenSSL treats them differently depending if you are in the x509 or CA context.

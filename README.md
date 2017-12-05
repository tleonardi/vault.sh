# vault.sh

_Vault is a simple interactive bash script that simplifies the management of gpg encrypted files._

`vault` decrypts [gpg](http://www.gnupg.org) files to a specified folder (the vault) and indexes them for later re-encryption.
After working on the decrypted files inside the vault, `vault` scans the indexed files, re-encrypts them and moves them back to their original location. `vault` can detect if the encrypted file was modified after decryption (e.g. manually by the user or by another program), and prompts the user before overwriting it.

A typical use case is to store sensitive data as encrypted files in the cloud and use `vault` to decrypt them to a local folder and put them back in the cloud after working on them.

## Features
* Simple interface
* Strong crypto (asymmetric encryption via gpg)
* Minimal dependecies (gpg, coreutils)
* Fast

## Examples

```ShellSession

tom@local ~ $ vault -h

vault simplifies encryption and decryption of files to a vault.

Usage: vault [OPTIONS] filename(s)

  -d       Decrypts the file to the vault and optionally makes 
           an index to re-encrypt it and copy it back to its
	   original location.
  -e       Re-encrypt indexed files in the vault and copy them
           to their original location.
  -p       Path of the vault. Deafults to $VAULT if set, other-
           wise $HOME/vault
  -k       GPG key to use to encrypt the files. If not provided
           uses $VAULT_KEY (if set) or the gpg option
	   --default-recipient-self.
  -q       Run non-interactively (automatically overwrites)
  -v       Print version
  -h	   Print this help message
```



### Decrypting a secret file
```ShellSession
tom@local ~/secret_files $ ls
secret.gpg
 
tom@local ~/secret_files $ vault secret.gpg 
gpg: encrypted with 2048-bit RSA key, ID 0xXXXXXXXX, created 2000-01-01
      "Tommaso Leonardi <tom@itm6.xyz>"
The file was decrypted to the vault: ~/vault/secret

1) Delete it
2) Add to Index for later re-encryption (default)
3) Leave it
2
tom@local ~/secret_files $ ls ~/vault
secret
```

### Re-encrypting a secret file
```ShellSession

tom@local ~/secret_files $ ls ~/vault
secret

tom@local ~/secret_files $ cat ~/vault/secret
This is my secret

tom@local ~/secret_files $ echo "My new secret" > ~/vault/secret

tom@local ~/secret_files $ vault -e
Processing file: secret
	Do you want to overwrite ~/secret_files/secret.gpg?(Y/n) 
Y
gpg: using "0xXXXXXXXXX" as default secret key for signing
	Do you want to delete the unencrypted file ~/vault/secret and its index?(Y/n) 
Y
tom@local ~/secret_files $ ls ~/vault
tom@local ~/secret_files $
```

### Re-encrypting after modifying the original file
```ShellSession

tom@local ~/secret_files $ ls
secret.gpg
 
tom@local ~/secret_files $ vault secret.gpg 
gpg: encrypted with 2048-bit RSA key, ID 0xXXXXXXXX, created 2000-01-01
      "Tommaso Leonardi <tom@itm6.xyz>"
The file was decrypted to the vault: ~/vault/secret

1) Delete it
2) Add to Index for later re-encryption (default)
3) Leave it
2
tom@local ~/secret_files $ ls ~/vault
secret
tom@local ~/secret_files $ echo "Original file changed" > ~/secret_files/secret.gpg

tom@local ~/secret_files $ vault -e
Processing file: secret
	## The original file has been modified after decryption! ##
	Do you want to overwrite ~/secret_files/secret.gpg?(Y/n) 
n
```

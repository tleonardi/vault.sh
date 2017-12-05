# vault.sh

Vault is a simple interactive bash script that simplifies the management of gpg encrypted files.

```
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

## Synopsis

Given a gpg-encrypted file, `vault` decrypts it to a specified folder (the vault) and indexes it for later re-encryption.
After working on the decrypted files in the vault, invoking `vault -e` scans the indexed files in the vault, re-encrypts them and moves them back to their original location. `vault.sh` can detect if the encrypted file was modified after decryption (e.g. manually by the user or by another program), and prompts the user before overwriting it.

## Examples

### Decrypting a secret file:
```
tom@local ~/secret_files $ ls
secret.gpg
 
tom@local ~/secret_files $ vault secret.gpg 
gpg: encrypted with 2048-bit RSA key, ID 0xXXXXXXXX, created 2000-01-01
      "Tommaso Leonardi <tom@itm6.xyz>"
The file was decrypted to the vault: /home/tleonardi/vault/secret

1) Delete it
2) Add to Index for later re-encryption (default)
3) Leave it
2
tom@local ~/secret_files $ ls ~/vault
secret
```

### Re-encrypting a secret file
```
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
tleonardi@toms_macbook ~/Bigcloud/Projects/vault $ ls
tleonardi@toms_macbook ~/Bigcloud/Projects/vault $
```

### Re-encrypting after modifying the original file
```
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
tom@local ~/secret_files $ echo "My new secret" > ~/vault/secret

tom@local ~/secret_files $ vault -e
Processing file: secret
	## The original file has been modified after decryption! ##
	Do you want to overwrite ~/secret_files/secret.gpg?(Y/n) 
n
```

---
layout: post
title:  "Offline Printable Bitwarden Backup"
slug:   offline-bitwarden-backup
date:   2022-03-15 10:53:11
categories: backups
tags: 
 - bitwarden
 - backups
 - print
 - qrencode
 - qr-codes
---

## Bitwarden Password Manager

Bitwarden and other password managers are an excellent tool for managing passwords day to day,
but are difficult to backup in an offline way. This post is a guide for backing up the vault
to a QR code that can be printed and safely stored offline with your other important documents.

Additionally, it will be stored in a plaintext manner which will allow easy recovery without
needing to load back into bitwarden, if you wish.

This guide requires `jq` and `qrencode`.

## Bitwarden Export

First, export your vault, using the unencrypted json option. Store to `vault.json`.


## Convert to Simpler Format

By default, the output is quite verbose, and for vaults of larger size, would require many, many
QR codes to fully export, due to the 4k size limit of a QR code.

This JQ line allows us to export it in a smaller format to get more content in each QR code.


```shell

echo "id\tname\turl\tusername\tpassword\tnote\t2FA totp token"> vault-passwords.txt

# handle typical login entries in a highly compressed way
cat vault.json | jq -r '.items[] | select(.type == 1) | (.id + "\t" + .name + "\t" + .login.uris[0].uri + "\t" + .login.username + "\t" + .login.password + "\t" + .note + "\t" + .totp)' >> vault-passwords.txt

# handle other types in a more generic way, removing unneeded fields
cat vault.json | jq '.items[] | select(.type != 1) | del(.favorite, .reprompt, .folderId, .collectionIds, .type, .organizationId)' > vault-other.txt
```

## Output to QR Codes

Now, you have `vault-passwords.txt` which contains your normal logins, and `vault-other.txt`
which contains all the other types of vault entries such as secure notes or credit card entries.

```shell
cat vault-passwords.txt | qrencode -o vault-passwords-qr.png -t png -Sv 40
cat vault-other.txt | qrencode -o vault-other-qr.png -t png -Sv 40
```

## Final Printing

Now, print all the QR codes, (and probably this page as well so people can tell what it is), and 
store in a safe place.

## Restoration

Simply scan all the QR codes, and get your passwords back in plaintext.


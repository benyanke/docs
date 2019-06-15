---
layout: post
title:  "Start with Pass - Standard Unix Password Manager"
slug:   start
date:   2018-11-02 23:58:29
categories: pass
tags: 
 - passwords
 - security
 - pass
 - gpg
---

`pass`, the standard unix password manager, is a simple shell script that 
provides an interface to easily store and access files encrypted with 
GPG, in a secure password vault.

`pass` uses `gpg` (an implementation of the pgp encryption protocol) to 
encrypt and decrypt your files, so you can access them with any tool 
which decrypts with your GPG keys, not just the `pass` tool. You can sync 
the password vault using any tool, or you can use git, which is 
integrated with `pass`, to provide automatic syncing between devices.

There are several other tools you can use to manage your pass database:

  * [qtpass](https://qtpass.org) - a desktop gui pass tool providing full 
    GUI management of the vault
  * [gopass](https://github.com/gopasspw/gopass) - a replacement for 
    `pass` with some additional features like search and coloring
  * `gpg` - you can directly decrypt your password files using `gpg`, 
    without any wrapper. This can be useful for building your own scripts 
    or tooling.


## Setting up your GPG key

First, you need to create your GPG key. You can skip this step if you 
already have a GPG key you would like to use.

First, generate your key:

```
$ gpg --gen-key
```

You will see a number of options. Generally, they are clear on 
their own, but here are the options you should check that may not be clear:

  * Select "DSA and RSA" key type
  * Create a 4096 bit key
  * Expiration - generally set to 0 (none) if you're not sure. Read 
    more below.

You'll then enter your name, email, password, and allow the key to 
generate. After you enter the password, it may take a while. Once 
the entropy is generated, the key generation is complete. The 
password must be secure, it is embedded in your key, used to 
encrypt it.

Finally, export from `gpg` and import to `gpg2`, so that tools 
which use gpg2 can also use the key.

```
$ gpg --export-secret-keys > keyfile
$ gpg2 --import keyfile;
$ rm keyfile;
```

Finally, your key is stored in ~/.gnupg.


TODO: Document pinetry options:
 - sudo nano ~/.gnupg/gpg-agent.conf
 - pinentry-program /usr/bin/pinentry-curses
  - OR:
 - pinentry-program /usr/bin/pinentry-qt


TODO: Document GPG expiration selection. 
TLDR: if you don't know what you're doing, never expire.

TODO: Document setting up GPG here

TODO: Document gpg backups

TODO: Document gpg security

## Setting up `pass`

Now that you have a key, either an older key, or one generated above, you can set up your pass vault.

First, list your keys using the gpg command:
```
$ gpg --list-secret-keys
/home/johndoen/.gnupg/secring.gpg
----------------------------------
sec   4096R/12345678 2018-11-02
uid                  John Doe (Comment) <jdoe@example.com>
ssb   4096R/12345678 2018-11-02
```

Your key is identified by the email (in this case, 
jdoe@example.com). Use this identifer to create your pass database, 
adding the key identifier to the end of the command.

```
$ pass init jdoe@example.com
mkdir: created directory '/home/johndoe/.password-store/'
Password store initialized for jdoe@example.com
```

Your password store has now been created at the path shown in the 
command output. You can now back up this directory, or sync it 
around, if you don't use the git option.

## Using `pass`

Passwords in `pass` are stored in a directory-like tree. 

We will view an example tree.

To view all your passwords, run:

```
$ pass

Password Store
├── email
│   ├── personal
│   │   ├── jdoe-gmail
│   │   └── jdoe-hotmail
│   └── work
│       └── ceo-doecorp
└── financial
    └── banks
        ├── bank-of-america
        ├── chase
        └── usbank
```


**Adding an Entry**

To add a password, run the `insert` command, followed by the path.

```
$ pass insert medical/hospital/billing
mkdir: created directory '/home/jdoe/.password-store/medical'
mkdir: created directory '/home/jdoe/.password-store/medical/hospital'
Enter password for medical/hospital/billing: 
Retype password for medical/hospital/billing: 
```

Type your password twice, and enter.

If you'd like to add more than a password, use the `edit` command documented below.

**Edit an Entry**

To edit an entry, run the `edit` command, followed by the path.

```
$ pass edit medical/hospital/billing
```

On running this command, it will ask for your password, and then open a text editor in the 
terminal where you can add as many details as you would like.


**View a Password**

To view a password, just specify your path without any command.

```
$ pass medical/hospital/billing
MyReallyLongPassword

email: jdoe@hotmail.com

Note: this account used for billing at downtown clinic,
not the suburb clinic.
```

You can also specify `-c` to copy the first line (usually the password itself) to the clipboard.

```
$ pass -c medical/hospital/billing
Copied medical/hospital/billing to clipboard. Will clear in 45 seconds.
```

You can now use it from your clipboard.


**Generate a Password**

Instead of using `insert`, you can use `generate` to create a new password for you, and store it. 
Also specify the character count. The command below will create and store a 10 character 
password.

```
$ pass generate medical/hospital/chart 10

The generated password for medical/hospital/chart is:
PB&<#t>oE1
```

You can then see your new password in the password store, and view it:

```
$ pass

Password Store
├── email
│   ├── personal
│   │   ├── jdoe-gmail
│   │   └── jdoe-hotmail
│   └── work
│       └── ceo-doecorp
├── financial
│   └── banks
│       ├── bank-of-america
│       ├── chase
│       └── usbank
└── medical
    └── hospital
        ├── billing
        └── chart
```

```
$ pass medical/hospital/chart
PB&<#t>oE1
```

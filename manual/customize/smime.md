---
title: 'S/MIME'
up: ../customize.md#sympa-services-optional-features
---

S/MIME
======

  * S/MIME support was initially introduced on Sympa 2.7.

Sympa supports features of
[S/MIME version 2](https://tools.ietf.org/html/rfc2311). It can
verify electronic signature of incoming message, decrypt incoming message
and re-encrypt them using users' certificates.

Requirements
------------

  - [Crypt-OpenSSL-X509](https://metacpan.org/release/Crypt-OpenSSL-X509)
    and [Crypt-SMIME](https://metacpan.org/release/Crypt-SMIME) Perl modules.

    ----
    Note:

      * On Sympa prior to 6.2, openssl(1) utility was required.  It is no
        longer required, however, to install Perl modules above, openssl
        library is required.

    ----

  - CA certificate: The certificate of certificate authority (CA) which
    issues users' certificates.
    To obtain CA certificate, consult the issuer organization.

    ----
    Note:

      * Several issuers provide one or more additional certificates, so-called
        "intermediate CA certificate", along with "root CA certificate".
        If it is the case, you have to obtain all of those certificates.

    ----
    Certificate files have to be in PEM format.

  - Key pair (certificate and private key) of address of Sympa itself.
    Certificate is used by users to encrypt messages bound for Sympa.
    Private key is used by Sympa to decrypt incoming message.

    You can either issue certificate by your own, or use certificate issued
    by appropriate CA.

Setup
-----

### Sympa configuration parameters

Following parameters in [`sympa.conf`](../layout.md#config) may configure
S/MIME support.

  - [`cafile`](../man/sympa.conf.5.md#cafile) and
    [`capath`](../man/sympa.conf.5.md#capath)

    Paths of trusted certificate stores.
    `cafile` is the path to a file including concatenated one or more
    certificates in PEM format.
    `capath` is the path to directory containing one or more certificates
    in PEM format, with file names are linked by hashed name using
    [`c_rehash(1)`](https://www.openssl.org/docs/manmaster/man1/c_rehash.html)
    utility.

  - [`key_passwd`](../man/sympa.conf.5.md#key_passwd)

    Passphrase to decrypt private key by which encrypted messages are
    decrypted, if the key is encrypted.

  - [`ssl_cert_dir`](../man/sympa.conf.5.md#ssl_cert_dir)

    Users' S/MIME certificates are saved in this directory
    (by default [``$EXPLDIR``](../layout.md#expldir)`/X509-user-certs`).

----
Note:

  * Message with *decrypted* format may be temporarily put into the directory
    specified by [`tmpdir`](../man/sympa.conf.5.md#tmpdir) (by default
    [``$SPOOLDIR``](../layout.md#spooldir)`/tmp`).
    Usually it may not be changed, but you should confirm that this directory
    is not exposed to public.

----

### Sympa setup

  1. Create directories for certificates (Note: Replace `$capath` and
     `$ssl_cert_dir` below)::
     ``` bash
     # mkdir $capath                (if capath parameter was set)
     # chmod 755 $capth             (ditto)
     # mkdir $ssl_cert_dir
     # chmod 755 $ssl_cert_dir
     # chown sympa $ssl_cert_dir
     ```
     Note that `ssl_cert_dir` directory must be writable by `sympa` user.

  2. Add appropriate parameters described in previous section to `sympa.conf`.

  3. Install obtained CA certificate(s) (see "[Requirements](#requirements)")
     into the directory and/or the file.  The CA certificate files must be
     readable (but not writable) by `sympa` user.

  4. Install key pair of Sympa as these names:

       - `cert.pem` for certificate.
       - `private_key` for private key.

     They may be put in one of following directories:

       - [``$EXPLDIR``](../layout.md#expldir)`/`*list path*`/`
         --- For a list address.
       - [``$EXPLDIR``](../layout.md#expldir)`/sympa/`
         --- For other addresses (see
         "[Addresses by mail domain](basics-addresses.md#addresses-by-mail-domain)").

     Note that they have to be readable by `sympa` user, however, private key
     *must not* be readable by other users.

### Obtaining users' certificates

User's certificate is used to verify signature of message, or to encrypt
message delivered to each user.

Sympa obtains user's certificate from the incoming message
automatically.  Or, you can install it manually into `ssl_cert_dir` directory.
Its file name is one of following by its usage:

  - *email@add.ress*`@enc`

    For encryption only.

  - *email@add.ress*`@sign`

    For signature verification only.

  - *email@add.ress*

    For both usages.

----
Note:

  * In fact, some punctuation characters included in *email@add.ress* have to
    be escaped to avoid limitation of filesystem encoding.
    By historical reason, escaping scheme is slightly wierd (`escape_chars()`
    in [Sympa::Tools::Text](../man/Sympa-Tools-Text.3.md) is used).
    This will be fixed on the future release of Sympa.

----

### User side setup

These certificates have to be distributed to users so that users may add them
to trusted certificate store in the MUA (mailer) of their own.

  - CA certificate(s).
  - Certificate of Sympa.

----
Note:

  * Private key *must never* be distributed.

----

How it works
------------

### Verifying S/MIME signature

  1. A user sends a message signed using their private key.

  2. Sympa verifies S/MIME signature of incoming message using the
     certificate included in it (or, use certificate cached in `ssl_cert_dir`
     directory).

  3. If verification succeeds, `smime` authentication method is assigned to
     the message, and it is used by scenario (see
     "[Authorization scenarios](basics-scenarios.md)", particularly
     "[Authentication methods](basics-scenarios.md#authentication-methods)").

Sympa does not alter signed messages: Decoration (adding "header" and "footer")
and personalization are not applied to messages delivered with standard
reception mode (see also
"[Does Sympa alter messages?](basics-alterations.md#does-sympa-alter-messages)").

### Handling encrypted message

At first, users who want to receive encrypted messages though Sympa have to
send message signed by their private key to Sympa's address.
Sympa obtains user's certificate extracted from these messages.

Once certificate is obtained by Sympa, message encryption is available:

  1. A user sends a message encrypted using Sympa's certificate.

  2. Sympa tries to decrypt incoming message using its private key
     (if decrypted message is signed, it also verifies signature as described
     in previous section).

     If decryption fails, encrypted message is delivered intact.

  3. At last when Sympa delivers message, it tries to encrypt message again
     using certificate of recipient by each, then delivers it.

     If encryption fails (e.g. certificate of the recipient is not found),
     Sympa delivers a message informing failure instead (a mail template
     `mail_tt2/x509-user-cert-missing.tt2` is used).

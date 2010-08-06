PharUtil - Security-oriented utilities for Phar archives
========================================================

Command line utilities for building, signing and verifying Phar archives with OpenSSL public/private key.
Additionally this library contains classes that help you distribute the code (e.g. plugins, skins) to remote
PHP applications over HTTP and, thanks to signing, remove the risk of Arbitraty Remote Code Execution in those
applications.

If you're not interested in remote deployment, you could always use included command line utilities to help you
deal with Phar archives.

Author: [Krzysztof Kotowicz](http://blog.kotowicz.net) - kkotowicz at gmail dot com

License: MIT

Source code: [github](http://github.com/koto/phar-util)

PEAR channel server: [pear.kotowicz.net](http://pear.kotowicz.net)

Command line utilties
---------------------
There are 3 command line utilities in this package:
* `phar-build` for building and signing Phar archives (see `phar-build -h` for help)
* `phar-extract` for extracting/listing contents of Phar archive (see `phar-extract -h` for help)
* `phar-generate-cert` for generating OpenSSL certificates used to sign the Phar archives (OpenSSL installation is required)

Installation
----------
### Dependencies
* OpenSSL compiled into PHP (`--with-openssl`)
* Phar 2.0.0

If you're using PHP >= 5.3.0, Phar is already bundled. For older versions
you must build it from [pecl](http://pecl.php.net).

E.g. under Ubuntu, these steps are required to build and configure the Phar extension:

    $ sudo apt-get install php5-dev
    $ sudo pecl install pecl/phar
    $ echo "extension=phar.so" | sudo tee /etc/php5/conf.d/phar.ini

### Configuring Phar
To be able to build Phar archives, you need to change php.ini setting: `phar.readonly=0`.
Under Ubuntu, this is done by executing:

    $ echo "phar.readonly=0" | sudo tee -a /etc/php5/conf.d/phar.ini

This step is optional (and discouraged) if you will only be reading Phar archives

### Install the package

Install the library through PEAR installer:
    $ sudo pear channel-discover pear.kotowicz.net
    $ sudo pear install kotowicz/PharUtil-beta

Usage
-----
#### Building a Phar archive
* Generate certificates in `cert/` directory (will be put in `priv.pem` and `pub.pem`)
        $ mkdir cert/
        $ cd cert/
        $ phar-generate-cert
* Create `src/` directory and copy all the files to build the archive from there
* Build a signed phar archive
        $ phar-build --phar library.phar
* Copy public key to a client

#### Extracting a Phar archive
        $ phar-extract library.phar output-directory

#### List a Phar archive contents
        $ phar-extract -l library.phar output-directory

#### Using Phar archive

Just use it like a normal Phar archive

    include_once 'phar://path/to/library.phar';

#### Secure remote code deployment with PharUtil
In this package there is a PHP class `PharUtil_RemotePharVerifier` you can use for secure remote code deployment
in your applications.

##### Introduction
[Phar](http://php.net/manual/en/book.phar.php) archives, though they have many superb features for a PHP
developer, have a certain limitation when it comes to their security - although standard Phar file can
be signed, the key used to verify the signature (i.e. the public key) must be stored alongside the archive.
Moreover, the signature verification process is hardcoded and does not allow you to supply the public key
by yourself.

These problems make it difficult to use Phar archives to distribute a trusted code to
clients, because both the key and code are stored on the server - attacker could use e.g.
DNS spoofing to emulate the server and supply the code without any signature or using his own
pair of keys. Because of that, Standard Phar extension does not allow including remote (e.g. HTTP://) Phar
archives to avoid the old security vulnerability of remote code execution (see e.g. `allow_url_fopen`
and `allow_url_include` discussion).

To mitigate this, the `PharUtil` library uses a different method:

* the key used to verify the signature is stored on a client (shared-secret) and is
  never transferred over-the-wire
* all code is downloaded to the local client sandbox and is verifed using the stored code
* after verification the file could be safely included and loaded (no `allow_url_open`
  restrictions)

For a more detailed view of different methods of including remote code see (and why should you care at all),
see [Hardening PHP: How to securely include remote code](http://blog.kotowicz.net/2010/07/hardening-php-how-to-securely-include.html)

##### Usage scenario

Usage scenario is as follows:

On server (publisher):

1. Create public / private keys with Open SSL (`phar-generate-cert`)
2. On a server build a code (`phar-build`) and sign it using your private key.

You may use the built archive on a server like any other Phar archive (see `example/local.php`)

On client (consumer):

3. Prepare a copy of public key (`cert/pub.pem`) for signature verification
4. Download a Phar archive from the server
5. Verify Phar signature using your copy of public key (to be sure that the code has
   been generated by trusted entity)
6. Include and run the Phar archive in your application

4,5,6 - see `remote.php` or the example below

Mentioned files are installed in `example` subdirectory of PEAR package documentation
(e.g. `/usr/share/php/docs/PharUtil/example`).

**Disclaimer**: For the security of given method, it is critical to never disclose the private key!
This method also doesn't protect anyone from looking AT the code - the code is not encrypted,
it is only signed so it cannot be changed by third party.

##### Using PharUtil_RemotePharVerifier on a client

Use `PharUtil_RemotePharVerifier` class to securely check for the Phar signature
before using the archive.

    <?php

    // all verified Phars will be copied to lib/ directory
    $verifier = new PharUtil_RemotePharVerifier('/tmp', './lib', './cert/public.pem');
    try {
      $verified_file = $verifier->fetch("http://example.com/library.phar");
    } catch (Exception $e) {
     // verification failed
     exit();
    }

    // $verified_file contains absolute filepath of a downloaded file
    // with signature verified from './cert/public.pem'
    include_once $verified_file;
    // or
    include_once 'phar://' . $verified_file . '/some/file/within.php';
    // or
    echo file_get_contents('phar://' . $verified_file . '/readme.txt');
    ?>

Contact
-------
[Krzysztof Kotowicz](http://blog.kotowicz.net) - kkotowicz at gmail dot com


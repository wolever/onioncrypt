onioncrypt: encrypt a file with multiple levels of encryption to multiple people
================================================================================

If you get hit by a bus, who'll be able to decrypt your laptop and clear your
browsing history?

``onioncrypt`` will encrypt a file (or directory) to multiple people, using
multiple levels of encryption.

This is useful in situations where a secret (for example, your laptop's
password) should be available to other people, but only if a certain number
agree to work together (for example, because you were hit by a bus).

This n-of-m encryption is accomplished in the most naive way possible: first
encrypting the file to each person, then encrypting the result to each other
person, and so on.


BETA WARNING
------------

**NOTE**: ``onioncrypt`` should still be considered beta quality, as it has not
been throughly or adiquitely reviewed.


Installation
------------

To install ``onioncrypt``::

    $ curl https://github.com/wolever/onioncrypt/raw/master/onioncrypt > ~/bin/onioncrypt
    $ chmod +x ~/bin/onioncrypt

Note: in theory ``onioncrypt`` will work on Linux, but it has not been tested
there (and, obviously, the OS X specific options like ``dmg`` file encryption
will not be available).

``gpg`` is required for ``gpg``-related operations (hint: ``brew install gpg``).


Recommendations
---------------

When using ``onioncrypt`` to distribute important secrets, please consider:

- The security of your secrets depends on both the trustworthiness of the
  recipients (ie, that they won't collude) *and* the trustworthiness of the
  encryption software (ie, gpg and OS X encrypted disk images).

- Strongly consider including all three forms of encryption (gpg to a public
  key, gpg to a password, and encrypted disk image to a password) to make sure
  that no one vulnerability can compromise your secrets.

- Remember that there is no way to revoke access once it's been granted, so
  consider carefully the people you will trust.

- Pick people in disparate social groups (ex, family, friends, and partners)
  and use the ``include`` and ``exclude`` options to prevent people in the same
  social group from potentially colluding.


Usage
-----

Create a configuration file::

    $ onioncrypt --sample-config
    ; This example configuration will generate:
    ; readme.txt
    ; alex-password.txt
    ; judy-password.txt
    ; david.zip.gpg:
    ;     README.txt
    ;     alex.zip.gpg: judy.dmg, chris.zip
    ; judy.zip.dmg:
    ;     README.txt
    ;     alex.zip.gpg: chris.zip, alex.zip.gpg, david.zip.gpg
    ; alex.zip.gpg
    ;     README.txt
    ;     chris.zip: judy.dmg, david.zip.gpg
    ;     judy.dmg: chris.zip
    ;     david.zip.gpg: chris.zip
    ; chris.zip
    ;     README.txt
    ;     alex.zip.gpg: judy.dmg, david.zip.gpg
    ;     judy.dmg: alex.zip.gpg
    ;     david.zip.gpg: alex.zip.gpg

    [DEFAULT]
    ; The number of levels of encryption (ie, layers in the onion)
    depth: 3

    [david]
    ; A gpg key files should be encrypted to.
    gpg-key: b230230d

    ; Files encrypted to david will never contain files encrypted to judy or
    ; chris (although the files encrypted to david may contain judy or chris).
    ; Exclusion is one direction. Both chris and judy may get files encrypted to
    ; david.
    exclude: judy, chris

    ; Contact information, for the README.txt file which will appear in each
    ; encrypted bundle.
    email: david@wolever.net
    full-name: David Wolever
    ; The contact-info field will be incldued verbatim
    contact-info:
        Home phone: 416 555 1234
        Cell phone: 416 555 4321
        123 Fake St
        Toronto, ON

    [judy]
    ; A password will be generated for judy and stored in 'judy-password.txt',
    ; then files will be stored in a disk image encrypted with that password.
    enc-method=dmg-symmetric

    ; Files encrypted to judy will only contain one file encrypted to alex.
    ; Multiple names can be included, separated by a comma: alex, david
    include: alex

    [alex]
    ; A password will be generated for alex and stored in 'alex-password.txt',
    ; then files will be stored in a zip file encrypted with 'gpg --symmetric'.
    enc-method: gpg-symmetric

    [chris]
    ; The UNSAFE-zip encryption method simply the input into a zip file. It goes
    ; without saying that this method isn't safe.
    enc-method: UNSAFE-zip
    $ onioncrypt --sample-config > ~/.onioncrypt.ini

Create a secure workspace (encouraged but not necessary)::

    $ onioncrypt --make-encrypted-workspace
    ...
    Encrypted image:
              File: /tmp/onioncrypt-workspace-fd2cb88b416b2123.sparseimage
          Password: /Volumes/OnioncryptWorkspace/disk-image-password.txt
        Mountpoint: /Volumes/OnioncryptWorkspace
    Hint:
        cd '/Volumes/OnioncryptWorkspace'

Save the secrets you want to ``onioncrypt`` to a file in the workspace::

    $ gpg --export-secret-key > /Volumes/OnioncryptWorkspace/my-secret-key.txt

Run ``onioncrypt:``::

    $ cd /Volumes/OnioncryptWorkspace
    $ onioncrypt ~/.onioncrypt.ini my-secret.txt .
    ted/
    | andrey/
    | | zach.gpg
    | | sarah.gpg
    | v
    | andrey.zip.gpg
    | zach/
    | | andrey.gpg
    | | sarah.gpg
    | v
    | zach.zip.gpg
    | sarah/
    | | andrey.gpg
    | | zach.gpg
    | v
    | sarah.zip.gpg
    v
    ted.zip.gpg
    ted-password.txt
    ted-readme.txt
    
    ...
    $ cat ted-password.txt
    juck-disring-conoy-beauty-hough-hayloft-lovable-lathen-hirable-demise-slab-empanel-dampang-youd-flavian


Then find a way to securely distribute the files (and, possibly, passwords) to
their intended recipients.

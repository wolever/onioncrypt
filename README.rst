onioncrypt: encrypt a file with multiple levels of encryption to multiple people
================================================================================

Usage
-----

Create a configuration file::

    $ onioncrypt --sample-config
    [DEFAULT]
    depth: 3

    [david]
    email: david@wolever.net
    ; A gpg key files should be encrypted to.
    gpg-key: b230230d
    ; Files encrypted to david will never contain files encrypted to judy or
    ; chris (although the files encrypted to david may contain judy or chris).
    ; Exclusion is one direction. Both chris and judy may get files encrypted to
    ; david.
    exclude: judy, chris

    [judy]
    email: judy@example.com
    ; A password will be generated for judy and stored in 'judy-password.txt',
    ; then files will be encrypted to that password.
    gen-password
    ; Files encrypted to judy will only contain one file encrypted to alex.
    ; Multiple names can be included, separated by a comma: alex, david
    include: alex

    [alex]
    gen-password

    [chris]
    gpg-key: abcd1234

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

Save the secrets you want to onioncrypt to a file in the workspace::

    $ gpg --export-secret-key > /Volumes/OnioncryptWorkspace/my-secret.txt

Run onioncrypt::

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
    
    ...
    $ cat ted-password.txt
    juck-disring-conoy-beauty-hough-hayloft-lovable-lathen-hirable-demise-slab-empanel-dampang-youd-flavian


Then find a way to securely distribute the files (and, possibly, passwords) to
their intended recipients.

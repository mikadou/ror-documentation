# ror-documentation

This repository is used to host the documentation for [Rigs of Rods](https://github.com/RigsOfRods/rigs-of-rods).
The `master` branch is intentionally left empty; the actual documentation resides in the `gh-pages` branch.
The latest documentation can be viewed at http://mikadou.github.io/ror-documentation.

The documentation is automatically updated for each successfully building commit to the [rigs-of-rods repository](https://github.com/RigsOfRods/rigs-of-rods). For the purpose of documentation the automated deployment process of documentation to this repository using [Travis CI](https://travis-ci.org/) is described in the following.

### Deployment Keys

The Travis CI service needs to be given access to the `ror-documentation` repository in order to push the latest documentation after each successful build. This is done by setting up a deploy key for the repository.
First a private/public keypair is created:

    ssh-keygen -b 2048 -t rsa
    
The file is saved as `deploy_key`. No password is used (i.e. just hit Enter).

Now, using the Github Webinterface navigate to the _settings_ for the `ror-documentation` repository. Select the section _Deploy keys_ and add a new deploy key to the repository. Copy the contents of the public key file `deploy_key.pub` into the provided form and make sure to enable the `Allow write access` option.

The private key which grants the access to the `ror-documentation` repository needs to be provided to Travis. but without exposing it to the rest of the world. 
In order to achieve this the private key is encrypted as follows (replace `<password>` with a strong password):

    openssl aes-256-cbc -pass pass:<password> -in deploy_key -out <path-to-ror>/tools/travis/linux/deploy_key.enc

The encrypted `deploy_key.enc` file is then added and commited to the repository. 
**Caution:** Do not to share the actual private key `deploy_key` file.


### Travis

In order to decrypt the `deploy_key.enc` file within Travis, the password needs to be provided to Travis.
Luckily Travis has built-in support for encrypted environment variables.
Point the Travis Web interface to the settings for the `rigs-of-rods` repository and add a new environment variable named `encrypted_deploy_key_passwd`. Set the value to the password that was used for encryption above. Make sure the option `Display value in build log` is turned off.

Within Travis the password is simply accessed via the `encrypted_deploy_key_passwd` environment variable. It is used to decrypt the private key which grants access to the `ror-documentation` repository. All output is supressed to prevent accidently revealing the password. In addition the file permissions of the key need to be adjusted.

    openssl aes-256-cbc -pass pass:$encrypted_deploy_key_passwd -in tools/travis/linux/deploy_key.enc -out ~/.ssh/id_rsa -d > /dev/null 2>&1
    chmod 600 ~/.ssh/id_rsa
    
Afterwards arbitrary changes to the `ror-documentation` repository can be pushed from within Travis using SSH:

    git push --force --quiet git@github.com:mikadou/ror-documentation.git master:gh-pages > /dev/null 2>&1

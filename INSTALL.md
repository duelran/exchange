HowTo Install
=============

This installs the ```prod``` version of boundless exchange using docker on a single host VM.
It needs ports 80 and 443 to be open on the VM.

1. Install recent version of docker and docker-compose (see http://docker.com). Note the additional steps in the instructions to run docker as non-root and restart docker daemon on system reboot.

1. Create a boundless directory:
    ```bash
    mkdir -p ~/git/boundless ; cd $_
    ```

1. RMIT blocks port 9418 used by git protocol so we use this workaround:
    ```
    git config --global url."https://".insteadOf git://
    ```
1. Install recent npm and nodejs for your platform (needed for Maploom):

   * For Ubuntu 16.04:
       ```
       cd ~
       curl -sL https://deb.nodesource.com/setup_6.x -o nodesource_setup.sh
       sudo bash nodesource_setup.sh
       sudo apt-get install nodejs
       sudo apt-get install build-essential
       ```
       (from https://www.digitalocean.com/community/tutorials/how-to-install-node-js-on-ubuntu-16-04)

    * See http://nodejs.org for other distributions.

1. Now we will checkout and install the prod branch of Maploom:
    ```
    sudo npm config set prefix /usr/local
    git clone https://github.com/ianedwardthomas/MapLoom.git --branch prod # includes all maploop fixes
    cd MapLoom
    sudo npm install -g grunt-cli karma bower
    sudo npm install
    bower install
    grunt watch -v   # hit control-c to escape when it hangs on waiting...
    cd ..
    ```

1. Now install geonode (which includes all fixes):
    ```
    git clone https://github.com/ianedwardthomas/geonode --branch prod
    ```

1. Finally install ```prod2``` version of exchange:
    ```
    git clone https://github.com/ianedwardthomas/exchange.git --branch prod2
    ```

1. Copy the built in version of ```env``` into your own copy and edit as needed:
    ```
    cp exchange/env exchange/.env
    ```

1. Next change the ```GEOIP``` variable in the ```.env``` file to point to external address for your host, then:
    ```
    cd exchange
    docker-compose build
    docker-compose up -d
    ```
1. The server should be up and running at http://hostip, where hostip is host VM external ip address.

1. Run the following command to see all the changes that have happened exchange's prod2 branch since it was started:
    ```
    git diff fdd425a...
    ```

1. The complete data state of the containers are kept in three named data volumes: ```exchange_dbdata```, ```exchange_geodata```, ```exchange_scratch```.  See:
    ```
        docker volume ls
    ```

    If you delete all the containers with ```docker-compose down``` and then recreate ```docker-compose up -d``` then full state will be retained.  Only if you delete the volumes manually using ```docker volume rm``` or use the ```-v``` option of ```docker-compose down``` will the data actually be deleted.

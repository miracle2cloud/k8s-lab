# Ceating Namespace, User & User Credentials
## Install kubectl
    curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
    cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
    deb https://apt.kubernetes.io/ kubernetes-xenial main
    EOF

    sudo apt-get update
    sudo apt-get install -y kubectl=1.19.1-00
    sudo apt-mark hold kubectl

## Create your own user in Ubuntu follow the below steps:
    $ adduser <username>
    #Add the new user to the sudo group 
    usermod -aG sudo <username>

## Switch to newly created user:
    su - <username>

## How to Enable SSH Password Authentication
## To enable SSH password authentication, you must SSH in as root to edit this file:
    sudo vi /etc/ssh/sshd_config
    PasswordAuthentication yes
    sudo service ssh restart

## Generate a private key for user and Certificate Signing Request (CSR) for user
    $ openssl genrsa -out user.key 2048

    $ openssl req -new -key user.key -out user.csr -subj "/CN=user/O=development"
## Generate a self-signed certificate. Use the CA keys for the Kubernetes cluster and set the certificate expiration.
    $ sudo openssl x509 -req -in user.csr  -CA ca.crt -CAkey ca.key -CAcreateserial  -out user.crt -days 45

# Creating a nice Docker pipeline with GitLab and DigitalOcean

1. Get a [Digital Ocean API Access Token](https://cloud.digitalocean.com/settings/api/tokens)
   and save it in a dotfile so it's available as environment variable: `DIGITAL_OCEAN_TOKEN`
   
2. Get a domain name: **myproject.xyz**

3. Create a new docker machine on Digital Ocean:
    ```
    $ docker-machine create \
        --driver digitalocean \
        --digitalocean-access-token $DIGITAL_OCEAN_TOKEN \
        myproject
    ```
    
4. Get the new machine's IP address from [the Digital Ocean GUI](https://cloud.digitalocean.com/droplets).
   Set it to environment variable: `SWARM_MANAGER_IP`
   
5. Set the CDN of the domainname **myproject.xyz** so that the only records are `A` records `@` and `*` pointing to
   `$SWARM_MANAGER_IP`.
   
5. "Log in" to the machine:
    ```
    $ eval $(docker-machine env myproject)
    ```
    
6. Set the machine to Swarm Mode:
    ```
    $ docker swarm init --advertise-addr $SWARM_MANAGER_IP
    ```
    
7. [Create GitLab Repo](https://gitlab.com/projects/new).

8. Go to **Settings** → **CI/CD** → **Secret variables**.
    ```
    $ pbcopy < $DOCKER_CERT_PATH/ca.pem
    ```
    Create a new variable called `TLSCACERT`, and <kbd>⌘V</kbd> in the value.
    ```
    $ pbcopy < $DOCKER_CERT_PATH/cert.pem
    ```
    Create a new variable called `TLSCERT`, and <kbd>⌘V</kbd> in the value.
    ```
    $ pbcopy < $DOCKER_CERT_PATH/key.pem
    ```
    Create a new variable called `TLSKEY`, and <kbd>⌘V</kbd> in the value.

8. Clone this repo:
    ```
    $ git clone https://github.com/emilniklas/gitlab-digitalocean-boilerplate myproject
    $ cd myproject
    $ rm -rf .git
    ```
    
9. Configure with the actual names:
    ```
    $ sed -i .old "s/0.0.0.0/$SWARM_MANAGER_IP/g" .gitlab-ci.yml
    $ rm .gitlab-ci.yml.old
    
    $ sed -i .old 's/myproject.xyz/MYACTUALPROJECTDOMAIN/g' .gitlab-ci.yml
    $ rm .gitlab-ci.yml.old
    
    $ sed -i .old 's/myproject/MYACTUALPROJECTNAME/g' .gitlab-ci.yml
    $ rm .gitlab-ci.yml.old
    
    $ sed -i .old 's/myproject/MYACTUALPROJECTNAME/g' myproject
    $ rm myproject.old
    
    $ mv myproject MYACTUALPROJECTNAME
    ```

10. Init Git, add GitLab remote, and push initial commit:
    ```
    $ git init
    $ git remote add origin git@gitlab.com:myproject/myproject.git
    $ git add .
    $ git commit -m "Initial commit"
    $ git push -u origin master
    ```

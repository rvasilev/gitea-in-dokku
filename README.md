# gitea-in-dokku

A walkthrough for running [Gitea](https://gitea.io/en-us/) in [Dokku](http://dokku.viewdocs.io/dokku/).

---

## Requirements

* working box with Dokku installed and configured
* Postgres or MySQL Dokku Plugin

## Postgres Dokku Plugin

Because Gitea requires a database, we can use the Postgres plugin for Dokku to manage our database container.  From inside dokku box, type the following:

    sudo dokku plugin:install https://github.com/dokku/dokku-postgres.git

## Deploy Gitea

We will run Gitea from the dockerhub.com image.  Inside the box, start by creating the application and our database:

    dokku apps:create gitea
    dokku postgres:create gitea-db
    dokku postgres:link gitea-db gitea

We need to save the Gitea files in a volume so that we don't lose our Git repositories between application restarts:

    sudo mkdir /var/lib/dokku/data/storage/gogs_data
    sudo chown root:root /var/lib/dokku/data/storage/gogs_data
    dokku storage:mount gitea /var/lib/dokku/data/storage/gogs_data:/data

We also need to use a custom proxy port for Gitea, since our system SSH service is already listening on port `22`:

    dokku proxy:ports-add gitea http:2222:22
    dokku proxy:ports-remove gitea http:22:22

Now we can pull and deploy the Docker image from dockerhub:

    docker pull gitea/gitea:latest
    docker tag gitea/gitea:latest dokku/gitea:latest
    dokku tags:deploy gitea latest

## Configure Gitea

***IMPORTANT***: Currently, there is a Dokku bug that does not properly expose the HTTP port. To bypass it we will manualy proxy internal http port:

    dokku proxy:ports-add gitea http:80:3000

To configure Gitea, you need to browse to Gitea' HTTP port and finish the Gitea setup.  All the database configuration can be see by issuing:

    dokku postgres:info gitea-db

Note the `Dsn:` entry, as it contains the username, password, hostname, port, and database name of the Postgres server.

# References

 1. [gogs-in-dokku]
 1. [TUTORIAL: DEPLOYING GOGS TO DOKKU]

[gogs-in-dokku]: https://github.com/cstroe/gogs-in-dokku

[TUTORIAL: DEPLOYING GOGS TO DOKKU]:https://dokku.github.io/tutorials/deploying-gogs-to-dokku

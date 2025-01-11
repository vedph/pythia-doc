---
title: Deployment
layout: default
parent: Home
nav_order: 8
---

# Deployment

The Pythia API and its sample frontend UI are distributed in Docker images. So, all what you need to deploy a Pythia API service and optionally its UI is just downloading and customizing a `docker-compose.yml` script.

>💡 To setup Docker, you can find quick instructions for your OS at <https://vedph.github.io/cadmus-doc/deploy/docker.html>.

## Pythia

You can use this `docker-compose.yml` file (from <https://github.com/vedph/pythia-shell-v2>) as a starting point. This is configured for local development, so you will have to make some changes. Also, it is using the Pythia shell app as a frontend, which usually is not what you'd like for your own website as this is mostly used for developing UI libraries. If you need a frontend, you can easily create your own frontend by using these libraries, and replace the app container with it as you prefer.

```yml
services:
  # PostgreSQL
  pythia-pgsql:
    image: postgres
    container_name: pythia-pgsql
    restart: unless-stopped
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
      - POSTGRES_DB=postgres
    ports:
      - 5432:5432
    networks:
      - pythia-network
    # this volume is for persisting DB data:
    volumes:
      - pgsql-vol:/var/lib/postgresql/data

# API
  pythia-api:
    image: vedph2020/pythia-api:5.0.5
    container_name: pythia-api
    restart: unless-stopped
    ports:
        - 5000:8080
    depends_on:
        - pythia-pgsql
    environment:
        - ASPNETCORE_URLS=http://+:8080
        - CONNECTIONSTRINGS__DEFAULT=User ID=postgres;Password=postgres;Host=pythia-pgsql;Port=5432;Database={0};
        # TODO: add the URL of your frontend app to CORS allowed origins, e.g.:
        # - ALLOWED__ORIGINS__0=http://www.your-pythia-frontend-url-here.edu
        - SEEDDELAY=30
        # TODO: change the default user password:
        - STOCKUSERS__0__PASSWORD=P4ss-W0rd!
        # TODO: change the JWT key:
        - JWT__SECUREKEY=Rh+m(dkh_Rn6DhOD-wKcd;>P=]Q*T}J/MPbnfenDKOL[1y4I_1Oy1JAU./V98Zex
        # TODO: if email is enabled, set the URLs and email as desired
        - MESSAGING__APIROOTURL=https://simpleblob.azurewebsites.net/api/
        - MESSAGING__APPROOTURL=https://fusisoft.it/apps/blob/
        - MESSAGING__SUPPORTEMAIL=webmaster@fusisoft.net
        # TODO: for seeding data uncomment this:
        # - DATA__SOURCEDIR=/opt/dump/
    # TODO: for seeding data uncomment this volume and copy bulk files into /opt/dump:
    # volumes:
    #     - /opt/dump:/opt/dump
    networks:
        - pythia-network

  # app
  pythia-app:
    image: vedph2020/pythia-shell:6.0.0
    container_name: pythia-app
    restart: unless-stopped
    ports:
      - 4200:80
    depends_on:
      - pythia-api
    networks:
      - pythia-network

networks:
  pythia-network:
    driver: bridge

# volume for persisting DB data
volumes:
  pgsql-vol:
```

>💡 For more details, you can look at the [Cadmus deployment pages](https://vedph.github.io/cadmus-doc/deploy), which provide a similar configuration.

In short, the most essential changes you will need to apply to this base script are all limited to the API service (marked with `TODO` in the above script):

1. add the URL of your frontend app to the allowed origins of CORS. This allows your frontend to access the API backend services via CORS.
2. change the default user password.
3. change the default JWT key. Be sure to make this key long enough (48+ characters), or the ASP.NET service will throw an error at startup.
4. if using email messages for account management, change the `MESSAGING__` entries accordingly.
5. if you want to seed your index the first time the API service starts without a database, see the next section.

## Database

The Pythia backend API image also provides an option to automatically seed a prebuilt index into the corresponding PostgreSQL database.

This is the quickest option to get a Pythia service up and running from your own index, as it just requires you to copy some binary files into a directory of your host. Of course, you still can seed your database using any other method (including [Docker compose](https://stackoverflow.com/questions/70879120/how-to-restore-postgresql-in-docker-compose) itself).

The set of PostgreSQL binary files used with this option are generated via bulk table copy (e.g. `COPY table TO STDOUT (FORMAT BINARY);`). You can use the [CLI tool](cli#bulk-write-command) to create them.

So, the typical procedure to seed a database in your host using the binary files is:

▶️ 1. use the [CLI tool](cli#bulk-write-command) to generate the bulk binary files.
▶️ 2. copy the generated files (there is one for each table in the database) in some folder in your host machine (e.g. `/opt/dump`).
▶️ 3. connect this folder to the container API via Docker volume binding, and set the corresponding environment variable (`DATA_SOURCEDIR`) to that volume. In this case, the API will seed data from the dump files on startup when creating the database. For instance:

```yml
  pythia-api:
    # ...
    environment:
        # ...
        - DATA__SOURCEDIR=/opt/dump/
    volumes:
        - /opt/dump:/opt/dump
    networks:
        - pythia-network
```

>⚠️ Note that in _Windows_ hosts you would need to quote a path including colons (e.g. `c:/data:/opt/dump`), which causes syntactic issues. You can use this [alternative syntax](https://www.reddit.com/r/docker/comments/hkx3s0/volume_mount_with_a_colon_in_the_path_with/):

```yml
    volumes:
      - type: bind
        source: 'c:/data'
        target: '/opt/dump'
```

>📖 See also [this SO post](https://stackoverflow.com/questions/46166304/docker-compose-volumes-without-colon).

Now, the first time you start the API container and there is no existing index database, it will create an empty one and seed it with data from the binary files.

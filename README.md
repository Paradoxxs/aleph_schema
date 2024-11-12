
First we have to download the "main" version of Aleph from "https://github.com/alephdata/aleph/tree/main"

copy aleph.env.tmpl to aleph.env

Generate key for ALEPH_SECRET_KEY

````
openssl rand -hex 24
````

if you want to disable authentication set "ALEPH_SINGLE_USER" to true

Once you are happy with your configuration, execute the following command to allow ElasticSearch to map its memory:

```bash
echo "vm.max_map_count=262144" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
```

When it comes to Windows. Here you have to open a wsl terminal and execute the same command.


After that, you can boot the docker-compose environment:

```bash
docker-compose up -d
```

This will run Aleph in detached mode. Before Aleph can process any requests or data, you need to make sure it sets up the database and search index correctly by executing an upgrade:

```bash
docker-compose run --rm shell aleph upgrade
```

While the system is running, it will bind to to port `8080` of its host machine and accept incoming connections. You can check that the system is functional with a curl request:

```bash
curl http://localhost:8080
```

You can shut down the system at any time by issuing the following command:

```bash
docker-compose down
```

#### Creating a users

During development, you will need to run command-line operations for certain tasks. In order to do so, you will first need to enter the docker container of Aleph. To do so, run:

```bash
make shell
# This will result in a root shell inside the container:
aleph --help
```
This will enter a docker container where the `aleph` shell command is available (see [Usage](https://github.com/alephdata/aleph/wiki/Usage) for details). You can also access the host computers file system at `/host`. This means a file stored at `/tmp/bla.txt` on your computer can be found at `/host/tmp/bla.txt` inside the container.

<Callout>
  When you run Aleph in development mode, the default configuration will not run the worker component used to index documents and do other background work. You can start it either via `make worker` or inside an Aleph shell using `aleph worker`.
</Callout>

#### Extend schema 

You might not be satisfied with the schema design of Aleph. The general rule when it comes to changing the schema is only added new entities to the schema. You can of cause still edit the existing entities, just do not remove attributes from them.

Here som general attributes of aleph schema:

- Abstract - Abstract schemata are used for inheritance only and shouldn’t be used directly.
- Generated - Entities using a generated schema shouldn’t be created directly by users.
- Matchable - Entities using a matchable schema can be used for matching and cross-referencing
- Hidden - Entities using a hidden schema shouldn’t be displayed in user interfaces or created by users.

look at requirements.txt to identify the version of followthemoney being used.  
Head over to https://github.com/alephdata/followthemoney/ and download the corresponding version of followthemoney.

add the following to 'aleph.env"

````
FTM_MODEL_PATH=/followthemoney
````

We also need to add additional volume to **api, ingest-file, and worker services**.

````
volumes:
  - ./path/on/your/computer:/followthemoney
````


When it comes to adding new entities after Aleph have been intitated. 
Add the new entities to the schema folder and run: 

````
docker-compose run --rm shell aleph resetindex
````

then take the environment down and back up again

````
docker-compose down
docker-compose up -d
````

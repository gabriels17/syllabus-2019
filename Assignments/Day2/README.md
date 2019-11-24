# Docker

What is Docker?\
[Read this article before starting the assignment](https://opensource.com/resources/what-docker)

## Part 1

### Getting started with docker

- [ ] Install Docker
After the installation you should add your user to the docker user group so that you do not
have to use sudo to run commands (Linux only).

When you are finished you should be able to run:\
`docker run hello-world`\
and get the following:

```
Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
...(snipped)...
```

### How do I know I'm done?

- [ ] Docker is installed.
- [ ] The docker command works without sudo privileges.

### FAQ

#### What Ubuntu do I have?

Run:\
`lsb_release -a`\
Should display (depending on your Ubuntu version):

```
Distributor ID:   Ubuntu
Description:      Ubuntu 18.04.3 LTS
Release:          18.04
Codename:         bionic
```

## Part 2

### Image vs Container

An instance of an image is called a container. You have an image, which is a set
of layers as you describe. If you start this image, you have a running container
of this image. You can have many running containers of the same image.

### Your new development environment

In the past, if you were to start writing a NodeJS app, your first order of
business was to install NodeJS and other dependencies onto your machine. But,
that creates a situation where the environment on your machine has to be just so
in order for your app to run as expected; ditto for the server that runs your
app.

With Docker, you can just grab a portable NodeJS runtime as an image, no
installation necessary. Then, your build can include the base NodeJS image right
alongside your app code, ensuring that your app, its dependencies, and the
runtime, all travel together.

These portable images are defined by something called a Dockerfile.

### Define a container with a Dockerfile

Dockerfile will define what goes on in the environment inside your container.
Access to resources like networking interfaces and disk drives is virtualized
inside this environment, which is isolated from the rest of your system, so you
have to map ports to the outside world, and be specific about what files you
want to “copy in” to that environment. However, after doing that, you can expect
that the build of your app defined in this Dockerfile will behave exactly the
same wherever it runs.

### Dockerfile

Create an empty directory called `item_repository`. Change directories (cd) into the new directory,
create a file called `Dockerfile`, copy-and-paste the following content into that
file, and save it.

```Dockerfile
FROM node:erbium

WORKDIR /code

COPY package.json package.json

RUN npm install

COPY database.js database.js

COPY app.js app.js

CMD node app.js
```

This Dockerfile refers to a couple of files we haven’t created yet, namely
`app.js`, `database.json` and  `package.json`. Let’s create those next.

### The app itself

Create three more files, `app.js`, `database.js` and `package.json`, and put them in the same
folder with the Dockerfile. This completes our app. When the above Dockerfile is built into an image, `app.js`, `database.js` and `package.json` will be present because of that Dockerfile’s `COPY` command.

`app.js`

```javascript
const express = require("express");
const database = require("./database.js");

var app = express();

app.get('/status', (req, res) => {
    res.statusCode = 200;
    res.send('The API is running!\n');
});

//api call to /items which returns a list of the the 10 newest item names.
app.get('/items', (req, res) => {
    database.getItems(function(items) {
        // TODO: order items alphabetically
        var names = items.map(x => x.name);
        res.statusCode = 200;
        res.send(names);
    });
});

//api call to /items/name which inserts an item to database.
app.post('/items/:name', (req, res) => {
    var name = req.params.name;
    database.insertItem(name, new Date(), function() {
        var msg = 'item inserted successfully';
        res.statusCode = 201;
        res.send(msg);
    });
})

app.listen(3000);
```

`database.js`

```javascript
const { Client } = require('pg');

// export available database functions.
module.exports = {
    insertItem: (name, insertDate, onInsert) => {
        var client = getClient();
        client.connect(() => {
            const query = {
                text: 'INSERT INTO Item(Name, InsertDate) VALUES($1, $2);',
                values: [name, insertDate],
            }
            client.query(query, () => {
                onInsert();
                client.end();
            });
        });
        return;
    },
    getItems: (onGet) => {
        var client = getClient();
        client.connect(() => {
            const query = {
                text: 'SELECT ID, Name, InsertDate FROM Item ORDER BY InsertDate DESC LIMIT 10;',
                rowMode: 'array'
            }
            client.query(query, (err, res) => {
                onGet(res.rows.map(row => {
                    return {
                        id: row[0],
                        name: row[1],
                        insertdate: row[2]
                    }
                }));
                client.end();
            });
        });
        return;
    }
}

function getClient() {
    return new Client({
        host: "your container name",
        user: process.env.POSTGRES_USER,
        password: process.env.POSTGRES_PASSWORD,
        database: process.env.POSTGRES_DB
    });
}

// give the database a few seconds to setup before we connect to it.
setTimeout(() => {
    var client = getClient();
    client.connect((err) => {
        if (err) {
            console.log('failed to connect to postgres!');
        } else {
            console.log('successfully connected to postgres!');
            client.query('CREATE TABLE IF NOT EXISTS Item (ID SERIAL PRIMARY KEY, Name VARCHAR(32) NOT NULL, InsertDate TIMESTAMP NOT NULL);', (err) => {
                if (err) {
                    console.log('error creating Item table!')
                } else {
                    console.log('successfully created item table!')
                }
                client.end();
            });
        }
    });
}, 3000);
```

`package.json`

```json
{
  "name": "FILL_IN_WITH_A_NAME_OF_YOUR_CHOICE",
  "version": "1.0.0",
  "description": "",
  "main": "app.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "FILL_IN_WITH_YOUR_NAME",
  "license": "ISC",
  "dependencies": {
    "express": "4.17.1",
    "pg": "7.12.1"
  }
}
```

### Build the app

We are ready to build the app. Make sure you are still at the top level of your
new directory. Here’s what `ls` should show:

```
$ ls
app.js  database.js  Dockerfile  package.json
```

Now run the build command. This creates a Docker image, which we’re going to tag
using -t so it has a friendly name.

```
docker build -t item_repository .
```

Where is your built image? It’s in your machine’s local Docker image registry:

```
$ docker images

REPOSITORY            TAG                 IMAGE ID
item_repository       latest              326387cea398
```

### Run the app

Run the app, mapping your machine’s port 3000 to the container’s published port
3000 using -p:

```
$ docker run -p 3000:3000 item_repository

failed to connect to postgres!
```

Your shell will enter the docker container and won't allow you to ctrl+c
out of the process.

The app is a simple REST API that stores a list of items sent to it. The
app tries to connect to a postgres database but fails because you do not
have a postgres database running.

Open a new shell and use the curl command to view the same content.

```
$ curl http://localhost:3000/status

The API is running!
```

Now finally to exit the docker container, using another shell kill the
docker container by:

```
$ docker container list

CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                  NAMES
931853651e89        item_repository     "/bin/sh -c 'node ap…"   53 seconds ago      Up 51 seconds       0.0.0.0:3000->3000/tcp hopeful_golick

$ docker container stop 931853651e89
```

Now let’s run the app in the background, in detached mode:

```
docker run -d -p 3000:3000 item_repository
```

You get the long container ID for your app and then are kicked back to your
terminal. Your container is running in the background. You can also see the
abbreviated container ID with docker container ls (and both work interchangeably
when running commands):

```
$ docker container list

CONTAINER ID        IMAGE               COMMAND                   CREATED
1fa4ab2cf395        pagecounter        "/bin/sh -c 'node ..."     28 seconds ago
```

You’ll see that that the api is running on http://localhost:3000/status.

Now use docker container stop to end the process, using the CONTAINER ID, like
so:

```
docker container stop 1fa4ab2cf395
```

**Pro Tip** When running commands that require the `CONTAINER ID`, e.g.\
`docker container stop 1fa4ab2cf395`\
it's usually enough to write the first three characters or so, i.e.\
`docker container stop 1fa`

### Share your image

To demonstrate the portability of what we just created, let’s upload our built
image and run it somewhere else. After all, you’ll need to learn how to push to
registries when you want to deploy containers to production.

A registry is a collection of repositories, and a repository is a collection of
images—sort of like a GitHub repository, except the code is already built. An
account on a registry can create many repositories. The docker CLI uses Docker’s
public registry by default.

Note: We’ll be using Docker’s public registry here just because it’s free and
pre-configured, but there are many public ones to choose from, and you can even
set up your own private registry using Docker Trusted Registry.

### Log in with your Docker ID

If you don’t have a Docker account, sign up for one at
[cloud.docker.com](https://cloud.docker.com/). Make note of your username.

Log in to the Docker public registry on your local machine.

```
$ docker login
```

### Tag the image

The notation for associating a local image with a repository on a registry is
username/repository:tag. The tag is optional, but recommended, since it is the
mechanism that registries use to give Docker images a version. Give the
repository and tag meaningful names for the context, such as hgop:part2.

Now, put it all together to tag the image. Run docker tag image with your
username, repository, and tag names so that the image will upload to your
desired destination. The syntax of the command is:

```
docker tag image username/repository:tag
```

For example:

```
docker tag item_repository john/hgop:day2
```

Run docker images to see your newly tagged image. (You can also use docker image
ls.)

```
$ docker images
REPOSITORY               TAG                 IMAGE ID            CREATED             SIZE
pagecounter              latest              d9e555c53008        3 minutes ago       195MB
john/hgop                day2                d9e555c53008        3 minutes ago       195MB
...
```

### Publish the image

Upload your tagged image to the repository:

```
docker push username/repository:tag
```

Once complete, the results of this upload are publicly available. If you log in
to Docker Hub, you will see the new image there, with its pull command.

Pull and run the image from the remote repository From now on, you can use
docker run and run your app on any machine with this command:

```
docker run -p 3000:3000 username/repository:tag
```

If the image isn’t available locally on the machine, Docker will pull it from
the repository.

```
$ docker run -p 3000:3000 john/hgop:day2
Unable to find image 'john/hgop:day2' locally
part2: Pulling from john/hgop
10a267c67f42: Already exists
f68a39a6a5e4: Already exists
9beaffc0cf19: Already exists
3c1fe835fb6b: Already exists
4c9f1fa8fcb8: Already exists
ee7d8f576a14: Already exists
fbccdcced46e: Already exists
Digest: sha256:0601c866aab2adcc6498200efd0f754037e909e5fd42069adeff72d1e2439068
Status: Downloaded newer image for john/hgop:day2
 * Running on http://0.0.0.0:3000/ (Press CTRL+C to quit)
```

Note: If you don’t specify the :tag portion of these commands, the tag of
:latest will be assumed, both when you build and when you run images. Docker
will use the last version of the image that ran without a tag specified (not
necessarily the most recent image). No matter where docker run executes, it
pulls your image, along with Node and runs your code. It all travels together in
a neat little package, and the host machine doesn’t have to install anything but
Docker to run it.

### How do I know I'm done?

- [ ] I've created a docker image
- [ ] I ran an instance of my image (a docker container) and saw the results in
      a browser (curl)
- [ ] I've stored my docker image on Docker Cloud
- [ ] I've committed all my code to GitHub and commented each line in the
      `Dockerfile`, short description on the purpose of each line, e.g.

```
# Set the working directory to /code
WORKDIR /code
```

## Part 3 - Docker compose

Compose is a tool for defining and running multi-container Docker applications.
To learn more about Compose refer to the
[following documentation](https://docs.docker.com/compose/).

Start by installing Docker compose:\
[How to install Docker Compose](https://docs.docker.com/compose/install/)

### Define services in a Compose file

Create a file called `docker-compose.yml` in your root directory and paste the
following:

`docker-compose.yml`

```yml
version: '3'
services:
  my_item_repository:
    image: username/repo:tag
    ports:
    - '3000:3000'
    depends_on:
    - my_postgres_container
    environment:
      POSTGRES_DB: 'my_postgres_database'
      POSTGRES_USER: 'my_postgres_user'
      POSTGRES_PASSWORD: 'my_postgres_password'
  my_postgres_container:
    image: postgres
    environment:
      POSTGRES_DB: 'my_postgres_database'
      POSTGRES_USER: 'my_postgres_user'
      POSTGRES_PASSWORD: 'my_postgres_password'
```

Then take the environment variables you defined for the `my_postgres_container`
and add them to `database.js`. The host value should be the name of the container,
in this case `my_postgres_container`. Docker compose will map the host value
`my_postgres_container` to the postgres docker container behind the scenes
e.g. my_postgres_container:162.242.195.82

### Build and run your app with Compose

From your project directory, start up your application by running\
`docker-compose up`

`docker-compose` will then pull the images for both services `john/hgop:day2`
and `postgres` from Docker Cloud.\
NOTE: you need to push your image to Docker Cloud before you will see
any changes when you run `docker-compose`, it is not enough to change the code
since `docker-compose` does not know about your source code, it only works on
your docker images.

Docker-compose will link the item_repository and postgres containers
together so that they can connect to each other using the name in the
docker-compose.yml file e.g. \
hostname: "my_postgres_container" -> postgres IP.

When the containers have been started you should be seeing something similar to
this in your terminal:

```
Starting solution_my_postgres_container_1_3611b9a77b40 ... done
Recreating solution_my_item_repository_1_4706dd301372  ... done
Attaching to solution_my_postgres_container_1_3611b9a77b40, solution_my_item_repository_1_4706dd301372
my_postgres_container_1_3611b9a77b40 | 2018-11-07 21:51:12.120 UTC [1] LOG:  listening on IPv4 address "0.0.0.0", port 5432
my_postgres_container_1_3611b9a77b40 | 2018-11-07 21:51:12.120 UTC [1] LOG:  listening on IPv6 address "::", port 5432
my_postgres_container_1_3611b9a77b40 | 2018-11-07 21:51:12.136 UTC [1] LOG:  listening on Unix socket "/var/run/postgresql/.s.PGSQL.5432"
my_postgres_container_1_3611b9a77b40 | 2018-11-07 21:51:12.178 UTC [26] LOG:  database system was shut down at 2018-11-07 21:50:20 UTC
my_postgres_container_1_3611b9a77b40 | 2018-11-07 21:51:12.188 UTC [1] LOG:  database system is ready to accept connections
my_item_repository_1_4706dd301372 | successfully connected to postgres!
my_item_repository_1_4706dd301372 | successfully created item table!
```

You should now be able to post items by running:

```
curl -X POST http://localhost:3000/items/duck
curl -X POST http://localhost:3000/items/dog
curl -X POST http://localhost:3000/items/donkey
curl -X POST http://localhost:3000/items/monkey
curl -X POST http://localhost:3000/items/zebra
curl -X POST http://localhost:3000/items/lion
curl -X POST http://localhost:3000/items/elephant
curl -X POST http://localhost:3000/items/chicken
curl -X POST http://localhost:3000/items/cow
curl -X POST http://localhost:3000/items/deer
curl -X POST http://localhost:3000/items/rhino
curl -X POST http://localhost:3000/items/tiger
```

Now there should be 12 items in the database, try running:

```
curl -X GET http://localhost:3000/items
```

If you get these 10 items:

```
["chicken","cow","deer","donkey","elephant","lion","monkey","rhino","tiger","zebra"]
```

You have finished this part.

If want to start over with a new database you can find out online how to
delete a docker container, delete the postgres container. Once the container
is deleted you can run `docker-compose up` again and it will create a new
postgres container.

### How do I know I'm done?

- [ ] I've gotten to know Docker compose
- [ ] I ran `docker-compose up` and saw the expected results
- [ ] I've posted the 12 animals to the API.
- [ ] I've gotten the expected 10 items back.
- [ ] The items are ordered alphabetically.

## Part 4 - Questions

Create an `answers.md` file that describes the following
(only 1-2 sentences each)

```
# Docker Exercise
TODO: What was this assignment about

## What is Docker?
TODO: short description

## What is the difference between:
* Virtual Machine
* Docker Container
* Docker Image
TODO: short comparison

## Web API?
TODO: short description

## Postgres?
TODO: short description

## package.json file dependencies field:
TODO: short description

## npm express package:
TODO: short description

## npm pg package:
TODO: short description

## What is docker-compose:
TODO: short description

## Results
TODO: What was accomplished in this exercise
```

## Handin

You should store all the source files in your repository:

```bash
.
├── itemrepository
│   ├── app.js
│   ├── database.js
│   ├── Dockerfile
│   └── package.json
├── assignments
│   ├── day01
│   │   └── answers.md
│   └── day02
│       └── answers.md
├── scripts
│   └── setup_local_dev_environment.sh
└── docker-compose.yml
```

They must be placed at these location to get full marks.

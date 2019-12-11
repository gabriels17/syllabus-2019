# Day 12

## Database Migration

In this step we will add database migration to our project, keep in mind that since
this project is academic we are destroying and creating an entirely new database each
time we deploy the game. Normally the same database would be used by each deployment
which is when database migrations are useful.

Database migrations should be bi-directional, that is we should be able to migrate
the database from a lower => higher version and back from a higher => lower version
incase we need to rollback a deployment.

We will be using the `db-migrate` npm package to create and run our migration scripts.
The package should normally be installed as a dev dependency but because of how we are
recreating the database using docker-compose, we will install it as a production
dependency for simplicity.

Create file `database.json`:

```json
{
  "pg": {
    "driver": "pg",
    "host": "my_database_container",
    "database": "game_database",
    "user": "my_user",
    "password": "my_password",
    "schema": "postgres",
    "port": 5432,
    "max": 10,
    "idleTimeoutMillis": 30000
  }
}
```

Now run:

```bash
node node_modules/db-migrate/bin/db-migrate --env pg create GameResultTable
```

To create an empty migration script.

It should look like this:\
`20191210110458-GameResultTable.js`:

```javascript
"use strict";

var dbm;
var type;
var seed;

/**
 * We receive the dbmigrate dependency from dbmigrate initially.
 * This enables us to not have to rely on NODE_PATH.
 */
exports.setup = function(options, seedLink) {
  dbm = options.dbmigrate;
  type = dbm.dataType;
  seed = seedLink;
};

exports.up = function(db) {
  return null;
};

exports.down = function(db) {
  return null;
};

exports._meta = {
  version: 1
};
```

The only functions we care about are `up` and `down`, since this is our first migration
script up would be moving from an empty database to a database with a GameResult table
and down would be moving back (removing the GameResult table).

Since this is an academic exercise we will be intentionally leaving out the `InsertDate`
column that is being used in our API, so that we can create another migration script
that modifies our GameResult table by adding it.

Here is the code for the `GameResultTable` migration:

```javascript
"use strict";

var dbm;
var type;
var seed;

/**
 * We receive the dbmigrate dependency from dbmigrate initially.
 * This enables us to not have to rely on NODE_PATH.
 */
exports.setup = function(options, seedLink) {
  dbm = options.dbmigrate;
  type = dbm.dataType;
  seed = seedLink;
};

exports.up = function(db) {
  return db.createTable("GameResult", {
    ID: { type: "int", primaryKey: true, autoIncrement: true },
    Won: { type: "boolean", notNull: true },
    Score: { type: "int", notNull: true },
    Total: { type: "int", notNull: true }
  });
};

exports.down = function(db) {
  return db.dropTable("GameResult");
};

exports._meta = {
  version: 1
};
```

Now remove the initialization part from `database.js` since we will be using `db-migrate`
to create our table.

Add npm script:\
`"migratedb:pg": "db-migrate --verbose --env pg --config ./database.json --migrations-dir ./migrations up",`

Now make sure to copy the migration files to the docker container and change CMD:

```dockerfile
# Give postgres time to setup before we try to migrate.
CMD sleep 5 && npm run migratedb:pg && node app.js
```

Now install the postgres driver npm package for db-migrate, `db-migrate-pg`:\
`npm install db-migrate-pg --save`

Our `pg` package converts our query names (database, table, columns) to lowercase, this
did not cause any problems when we were creating and querying the database with `pg`, but
now that we are using `pg` and `db-migrate` we will run into issues since Postgres is
case sensitive and `db-migrate` will create the table with upper-case characters in both
the table name and column names, but `pg` will convert our queries to use all lower-case.
To avoid this you should add quotes around the names in your queries:\
`INSERT INTO "GameResult" ("Won", "Score", "Total", "InsertDate") VALUES($1, $2, $3, CURRENT_TIMESTAMP);`

Now run API using `docker-compose` locally, you should see postgres complain:\
`ERROR: column "InsertDate" of relation "GameResult" does not exist`

Now create another migration script that adds the `InsertDate` column to the `GameResult`
table.

Your game should now be playable through the API without getting any errors.

There is a bug in the API, when the user starts the game with 21, the game does not get
stored in the database, **you will need to fix it**.

You should also be able to call `/stats` on your API to receive statistics.

To verify:

- Start the API locally
- Configure the capacity test to play 10.000 games
- Run the capacity test against your local API
- Call the `/stats` method
- You should get roughly: \
  `{"totalNumberOfGames":"10000","totalNumberOfWins":"4162","totalNumberOf21":"919"}`

Playing 10.000 games against a local instance took about 60 seconds, so if it takes
longer than 5 minutes there is something wrong.

## Connect UI

In this step we will connect UI to our server.
This way we can visualize the process of the game.

Go to the root of the project and create a new git repository and pull the client code to the folder.

```bash
mkdir game_client
cd game_client
git init
git pull git@github.com:hgop/lucky-21-frontend.git
rm -rf .git
cd ..
git rm --cached game_client
```

Install project dependencies
```bash
npm install
```

Create file `Dockerfile`:
```dockerfile
FROM node:latest

WORKDIR /code
COPY package.json package.json
RUN npm install
COPY . .
CMD ["npm", "start"]
```

Build Docker and push to Dockerhub.

Remember to tag accordingly.

Add the container to you `docker-compose.yml` file at the root of the project.

The Client code is missing connection to game_api.

Note, the Client might need a refresh after pressing New Game.

In game-api, `server.js` needs to be modified to allow cross origin resource sharing.

It is done be adding a header Access-Control-Allow-Origin in the API's express app.

### Hints

Since we can not have a TA class today, due to weather. Here are a few helpful hints:

`scripts/start_local.sh`
~~~bash
#!/bin/bash

set -euxo pipefail

docker build game_api -t ironpeak/game_api:dev
(cd game_client && npm run build)
docker build game_client -t ironpeak/game_client:dev

API_URL=localhost GIT_COMMIT=dev docker-compose up
~~~

`Access-Control-Allow-Origin Header`
~~~javascript
  app.use((req, res, next) => {
    res.header("Access-Control-Allow-Origin", "*");
    next();
  });
~~~

`Game Client`
~~~yaml
  game_client:
    image: username/game_client:${GIT_COMMIT}
    ports:
    - '4000:4000'
    depends_on:
    - game_api
    environment:
      PORT: 4000
      API_PORT: 3000
      API_URL: ${API_URL}
~~~

## Handin

You should store all the source files in your repository:

```bash
├── game_api
│   ├── migrations
│   │   ├── **************-GameResultTable.js
│   │   └── **************-GameResultTableAddInsertDate.js
│   ├── .eslintrc.json
│   ├── app.js
│   ├── config.js
│   ├── context.js
│   ├── database.js
│   ├── database.json
│   ├── dealer.js
│   ├── dealer.unit-test.js
│   ├── deck.js
│   ├── deck.unit-test.js
│   ├── Dockerfile
│   ├── inject.js
│   ├── lucky21.js
│   ├── lucky21.unit-test.js
│   ├── package.json
│   ├── random.js
│   ├── random.unit-test.js
│   ├── server.api-test.js
│   ├── server.capacity-test.js
│   ├── server.js
│   └── server.lib-test.js
├── game_client
│   │   ├── App.js
│   │   └── utils.js
│   ├── Dockerfile
│   └── index.js
├── assignments
│   ├── day01
│   │   └── answers.md
│   ├── day02
│   │   └── answers.md
│   └── day11
│       └── answers.md
├── scripts
│   ├── deploy.sh
│   ├── docker_build.sh
│   ├── docker_compose_up.sh
│   ├── docker_push.sh
│   ├── initialize_game_api_instance.sh
│   ├── jenkins_deploy.sh
│   ├── setup_local_dev_environment.sh
│   └── sync_session.sh
├── docker-compose.yml
├── infrastructure.tf
├── Jenkinsfile
└── README.md
```

They must be placed at these location to get full marks.

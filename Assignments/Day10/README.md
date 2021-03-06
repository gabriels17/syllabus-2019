# Week 2 - Assignment

**Turn in your GitHub repository URL into Canvas solo or in pairs (if the Github
repository is private please add kthorri and ironpeak as
collaborators)**

**Create a Jenkins user for us, and turn it's username and password in as a
comment with the URL**

You should store all the source files in your repository:

```bash
├── game_api
│   ├── .eslintrc.json
│   ├── app.js
│   ├── config.js
│   ├── context.js
│   ├── database.js
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
│   └── server.js
├── assignments
│   ├── day01
│   │   └── answers.md
│   └── day02
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

Add a `README.md` file, it should include:
 - URL to the Jenkins instance running on port 8080.
 - Any additional information about the project you want us to take into account, that could be
 additional things you managed to implement or an explenation about why you didn't manage to
 complete something etc.

The Jenkins deploy job should output the ip the game API is running on to the console.\
We will use this ip to check if the API is running.

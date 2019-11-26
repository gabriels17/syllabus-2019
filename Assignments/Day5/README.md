# Week 1 - Assignment

**Turn in your GitHub repository URL into Canvas solo or in pairs (if the Github
repository is private please add `kthorri`, `fanneyyy` and `ironpeak` as
collaborators)**

You should store all the source files in your repository:

```bash
├── item_repository
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
│   ├── deploy.sh
│   ├── initialize_game_api_instance.sh
│   └── setup_local_dev_environment.sh
├── docker-compose.yml
├── infrastructure.tf
└── README.md
```

They must be placed at these location to get full marks.

Add a `README.md` file, it should include the URL to the instance running
the API and make sure the port 3000 is reachable from anywhere so we can
verify that the API is running. Along with any additional information
about the project that you want us to take into account when grading.

**IMPORTANT** even though the AWS educate session expires every 3 hour
you can still use terraform to deploy your instance and it will remain
up and running even after the session expires.

# Stuff for HR Presentation

1. Install Python & pip & other deps
    1. `$ brew update`
    1. `$ brew install python`
    1. `$ pip install --upgrade distribute`
    1. `$ pip install --upgrade pip`
    1. `$ brew install awscli`
1. Download EB CLI:
    1. `$ pip install --upgrade --user awsebcli`
1. Add Python to path:
    1. `$ python --version`
    1. Then add the following to your `.bash_profile` or `.path`: `PATH=~/Library/Python/2.7/bin:$PATH`
1. Create IAM user and group
1. Configure your AWS profile on the CLI
    1. `$ aws configure --profile [yourprofile]`
    1. Use `us-west-2` for region name and `json` for format
    1. Check out `~/.aws` to see the setup
1. Initialize your EB configuration on the project
    1. `$ eb init --profile [yourprofile]`
    1. App name: todomvc-plusplus
1. Create your EB environment: `$ eb create`
    1. Environment name: todomvc-plusplus-prod
    1. DNS CNAME prefix: todomvc
    1. Classic load balancer, either works
1. Create RDS instance
    1. DB instance identifier: todomvc
    1. Master username/password isn't a big deal (not accessible to public)
    1. Use it's own security group, e.g. `rds-launch-wizard-1`
    1. Database name: todos
1. Configure the EBS environment
    1. Static files: Virtual Path: `/static/` (what the browser hits), Directory: `/public/ (where we look on our server)
    1. Environment properties:
        1. `NODE_ENV=production`
        1. `NPM_CONFIG_PRODUCTION=true`
        1. `RDS_CONNECTION_URL=postgres://[db_user]:[db_password]@[connection_url]/[db_name]`
1. Add PostgreSQL connection to inbound rules of RDS security group `rds-launch-wizard-1`
    1. Choose 5432 as port, allowing connections from the EC2 security group
1. Set up Travis
    1. Sign into Travis, tell it to do your repo
    1. Install Travis CLI
        1. Check Ruby > 1.9.3 `ruby -v`
        1. Download Travis CLI `$ gem install travis -v 1.8.8 --no-rdoc --no-ri`
        1. Verify `$ travis version`
    1. Get an encrypted secret access key
    1. Add .travis.yml

```
language: node_js
node_js:
  - "6"
cache:
  directories:
  - node_modules
env:
- TRAVIS_NODE_VERSION="6.3"
services:
- "postgresql"
install:
- npm install
before_script:
  - psql -c 'create database "todos-test";' -U postgres
  - sequelize db:migrate
script:
- npm test
- "./node_modules/.bin/grunt collect_static"
```

    1. Run `$ travis setup elasticbeanstalk`, which will get you mostly set up for EB.

```
deploy:
  provider: elasticbeanstalk
  access_key_id: <access_key>
  secret_access_key:
    secure: <encrypted_secret_key>
  region: us-west-2
  app: <elb_app>
  env: <elb_env>
  on:
    repo: <your_repo>
  # V--- add these lines yourself ---V
  bucket_name: <bucket_name_generated_by_eb>
  bucket_path: <elb_app>
  skip_cleanup: true
```



# TodoMVC++ : Taking TodoMVC To Production

TodoMVC++ is the companion application for Zero to Production with Node.js.
To learn more about how this application works, check out the video course on
[Frontend Masters](https://www.frontendmasters.com).

This application is based on [TodoMVC](http://todomvc.com/), and specifically
on a [Vue.js implementation](http://todomvc.com/examples/vue/) by Evan You.

## Running Locally

### Installing Node.js and npm

This application has been tested on Node.js 6 and npm 3 - these packages should
be available for download [here](https://nodejs.org/en/) - choose the "Current"
version for download.

### Installing Node.js modules

Once you have Node and npm installed and this repository downloaded, you'll need
to install the application's dependencies. Do this with:

    npm install

For development you'll probably want to install the following modules globally:

    npm install -g grunt-cli sequelize-cli

### Setting up the database

To run this application locally, you'll require a Postgres database. On a Mac,
the easiest path to installing Postgres is [Homebrew](http://brew.sh/). Once
installed, grab Postgres with:

    brew update
    brew install postgres

If Postgres is installed using the method above, you should now have a few
Postgres administrative commands on your system path. Begin by firing up another
Terminal tab and starting the database:

    postgres -D /usr/local/var/postgres

Next, create the development and test databases:

    createdb todos
    createdb todos-test

If Postgres is **not** installed locally, you can setup a free instance as follows:
- visit https://elephantsql.com
- login and setup a *free* tiny turtle instance
- goto "Details" and copy the url. It should look something like this [postgres://abcdefg:icRAC...](https://customer.elephantsql.com/instance)
- in the project, copy `config/test.js` to `config/user.js`
- set `config.databaseUrl` to your copied postgres url
- don't forget to run `sequelize db:migrate`

Apply the database migrations:

    sequelize db:migrate

Copy over static assets:

    grunt collect_static

### Running the application

To run the application in development mode:

    grunt

To run the application simulating production settings:

    NODE_ENV=production
    grunt collect_static
    npm start

## License

MIT


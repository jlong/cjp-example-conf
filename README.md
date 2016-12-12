# CJP Example Configuration DSL

This is an experimental repository that contains example configuration for CJP
written in an imaginary Groovy DSL. Have a look at
[conf/main.conf](conf/main.conf) to view example configuration. None of this is
working code. It is merely the sketch of an idea.

The following themes are explored:

* Is it possible to combine configuration for my entire CJP setup (with
  multiple masters) into one set of files? Configuration can then be pushed
  from CJOC to CJP masters as needed.
* What if Jenkins config as code was built from the ground up with support for
  Docker containers? While support for Docker would almost certainly be
  optional, what would Jenkins configuration look like if Docker support was a
  first-class feature?

In this model a new command line tool would need to be created for Jenkins to
evaluate configuration, bring up docker containers, and sync configuration
across masters. I've detailed the experience with an imaginary `jenkins`
configuration tool below.


## Start containers

After creating your config, you can run:

    $ jenkins start -d -c conf/main.conf

This does the following:

1. Lints and reads configuration from the entry point `conf/main.conf`
2. Detects that the config is using Docker
3. Generates a `docker-compose.yml` in a temp directory
4. Executes `docker-compose up -d` on this configuration to build and start the
   necessary Docker containers in the background
3. Syncs configuration through CJOC across masters


## Stop containers

To stop running containers:

    $ jenkins stop -c conf/main.conf

This does the following:

1. Reads configuration from the entry point `conf/main.conf`
2. Generates a `docker-compose.yml` in a temp directory
3. Executes `docker-compose stop` to stop running containers


## Lint config

To lint your configuration for syntax errors, run:

    $ jenkins lint -c conf/main.conf

This does the following:

1. Reads configuration from the entry point `conf/main.conf`
2. Outputs parse errors with file and line number


## Generate a docker-compose file

For debugging purposes or integration with other scripts it is helpful to be
able to generate a `docker-compose` configuration file. This is accomplished
with:

    $ jenkins compose -c conf/main.conf > my-docker-compose.yml

This does the following:

1. Reads configuration from `conf/main.conf`
2. Outputs docker-compose compatible YAML
3. Pipes the output into `my-docker-compose.yml`


## Sync configuration

Once our containers are up and running, we can sync configuration updates with:

    $ jenkins sync -c conf/main.conf

This does the following:

1. Lints and reads configuration from the entry point `conf/main.conf`
2. Ensures that containers are present for all masters present in
   configuration, if not the command fails
3. Syncs configuration through CJOC across masters

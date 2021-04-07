

### Back-end

- [PostgreSQL](https://www.postgresql.org/)
- [Redis](https://redis.io/)
- [Celery](https://github.com/celery/celery)

### Front-end

- [Webpack](https://webpack.js.org/)
- [TailwindCSS](https://tailwindcss.com/)
- [Heroicons](https://heroicons.com/)


- **Packages and extensions**:
    - *[gunicorn](https://gunicorn.org/)* for an app server in both development and production
    - *[whitenoise](https://github.com/evansd/whitenoise)* for serving static files
- **Linting and testing**:
    - *[flake8](https://github.com/PyCQA/flake8)* is used to lint the code base
- **Django apps**:
    - Add `pages` app with a `/` page and `/up` health check endpoint
- **Config**:
    - Log to STDOUT so that Docker can consume and deal with log output 
    - Extract a bunch of configuration settings into environment variables
    - Rename project directory from its custom name to `config`
    - `src/config/settings.py` and the `.env` file handles configuration in all environments
- **Front-end assets**:
    - `assets/` contains all your CSS, JS, images, fonts, etc. and is managed by Webpack
    - Custom `502.html` and `maintenance.html` pages
    - Generate favicons using modern best practices
- **Django defaults that are changed**:
    - Use Redis as the default Cache back-end
    - Use signed cookies as the session back-end
    - `public/` is the static directory where Django will serve static files from
    - `public_collected/` is where `collectstatic` will write its files to

Besides the Django app itself:

- Docker support has been added which would be any files having `*docker*` in
  its name
- GitHub Actions have been set up
- A `requirements-lock.txt` file has been introduced using `pip3`. The
  management of this file is fully automated by the commands found in the `run`
  file. We'll cover this in more detail when we talk about [updating
  dependencies](#updating-dependencies).

## Running this app

You'll need to have [Docker installed](https://docs.docker.com/get-docker/).
It's available on Windows, macOS and most distros of Linux. If you're new to
Docker and want to learn it in detail check out the [additional resources
links](#learn-more-about-docker-and-django) near the bottom of this README.


#### Clone this repo anywhere you want and move into the directory:

```sh
git clone https://github.com/rryanburton/docker-django-example hellodjango
cd hellodjango

# Optionally checkout a specific tag, such as: git checkout 0.3.1
```

#### Copy a few example files because the real files are git ignored:

```sh
cp .env.example .env
cp docker-compose.override.yml.example docker-compose.override.yml
```

#### Build everything:

```sh
docker-compose up --build
```


#### Setup the initial database:

```sh
# You can run this from a 2nd terminal.
./run manage migrate 
```


#### Check it out in a browser:

Visit <http://localhost:8000> 


#### Linting the code base:

```sh
# You should get no output (that means everything is operational).
./run flake8
```

#### Running the test suite:

```sh
# You should see all passing tests. Warnings are typically ok.
./run manage test
```

#### Stopping everything:

```sh
# Stop the containers and remove a few Docker related resources associated to this project.
docker-compose down
```

You can start things up again with `docker-compose up` 


## Files of interest

I recommend checking out most files and searching the code base for `TODO:`,
but please review the `.env` and `run` files

### `.env`

This file is ignored from version control so it will never be commit. 


### `run`

You can run `./run` to get a list of commands and each command has
documentation in the `run` file itself.

It's a shell script that has a number of functions defined to help you interact
with this project. It's basically a `Makefile` except with less limitations.
For example as a shell script it allows us to pass any arguments to another
program.

This comes in handy to run various Docker commands because sometimes these
commands can be a bit long to type. Feel free to add as many convenience
functions as you want. This file's purpose is to make your experience better!

*If you get tired of typing `./run` you can always create a shell alias with
`alias run=./run` in your `~/.bash_aliases` or equivalent file. Then you'll be
able to run `run` instead of `./run`.*

## Making this app your own

The app is named `config` right now but chances are your app will be a different
name. Since the app is already created we'll need to do a find / replace on a
few variants of the string "config" and update a few Docker related resources.

You can do that from the command line without needing to install any extra
dependencies by following the instructions below.

#### Remove the original Docker resources:


```sh
# This removes the PostgreSQL database volume along with a few other Docker
# resources that were created and are associated to the hello app.
docker-compose down -v
```

#### Run these commands from the same directory as this git repo:

```sh
# Change config to be whatever you want your app name to be.
#
# After renaming this value, paste this variable into your terminal.
lower=myapp

# If you wanted a multi-word app name you could do:
# lower=my_app
#
# or:
#
# lower=myapp
#
# The choice is yours!

# Recursively replace config using the values defined above.
#
# You don't need to edit this command before running it in your terminal.
find . ! -name 'README.md' -type f -exec perl -i -pe "s/(config)/${lower}/g" {} +
```

#### Verify everything was changed successfully:

```sh
grep -ER --exclude-dir public/ --exclude-dir public_collected/ config .
```

You should get back no output. That means all occurrences of hello were
replaced.

```sh
grep -ER --exclude README.md --exclude-dir .git/ --exclude-dir \
  --exclude-dir assets/node_modules/ \
  --exclude-dir public/ --exclude-dir public_collected/ "${lower}" .
```

You should get back a bunch of output showing you where your app name is
referenced within the project.

#### Remove the `.git` directory and init a new git repo:

This project is not meant to be a long running fork. It's your freshly minted
project that is ready for you to start modifying based on whatever cool app you
want to build.

```sh
rm -rf .git/
git init

# Optionally use main as a branch instead of master. CI is configured to work
# with both main and master btw.
git symbolic-ref HEAD refs/heads/main
```

#### Start and setup the project:

We don't need to rebuild anything yet. Upping it is enough for Docker to
re-create new resources with the new name. We'll also need to setup our
database since a new one will be created for us by Docker.

```sh
docker-compose up

# Then in a 2nd terminal once it's up and ready.
./run manage migrate
```

#### Sanity check to make sure the tests still pass:

It's always a good idea to make sure things are in a working state before
adding custom changes.

```sh
# You can run this from the same terminal as before.
./run flake8
./run manage test
```

If everything passes now you can optionally `git add -A && git commit -m
"Initial commit"` and start customizing your app. Alternatively you can wait
until you develop more of your app before committing anything. It's up to you!

#### Tying up a few loose ends:

You'll probably want to create a fresh `CHANGELOG.md` file for your project. I
like following the style guide at <https://keepachangelog.com/> but feel free
to use whichever style you prefer.

Since this project is MIT licensed you should keep my name and email address in
the `LICENSE` file to adhere to that license's agreement, but you can also add
your name and email on a new line.

If you happen to base your app off this example app or write about any of the
code in this project it would be rad if you could credit this repo by linking
to it. If you want to reference me directly please link to my site at
<https://nickjanetakis.com>. You don't have to do this, but it would be very
much appreciated!

## Updating dependencies

Let's say you've customized your app and it's time to make a change to your
`requirements.txt` or `package.json` file.

Without Docker you'd normally run `pip3 install -r requirements.txt` or `yarn
install`. With Docker it's basically the same thing and since these commands
are in our `Dockerfile` we can get away with doing a `docker-compose build` but
don't run that just yet.

#### In development:

You can run `./run pip3:outdated` or `./run yarn:outdated` to get a list of
outdated dependencies based on what you currently have installed. Once you've
figured out what you want to update, go make those updates in your
`requirements.txt` and / or `assets/package.json` file.

Then to update your dependencies you can run `./run pip3:install` or `./run
yarn:install`. That'll make sure any lock files get copied from Docker's image
(thanks to volumes) into your code repo and now you can commit those files to
version control like usual.

You can check out the
[run](https://github.com/nickjj/docker-django-example/blob/main/run) file to see
what these commands do in more detail.

As for the requirements' lock file, this ensures that the same exact versions
of every package you have (including dependencies of dependencies) get used the
next time you build the project. This file is the output of running `pip3
freeze`. You can check how it works by looking at
[bin/pip3-install](https://github.com/nickjj/docker-django-example/blob/main/bin/pip3-install).

You should never modify the lock files by hand. Add your top level Python
dependencies to `requirements.txt` and your top level JavaScript dependencies
to `assets/package.json`, then run the `./run` command(s) mentioned earlier.

#### In CI:

You'll want to run `docker-compose build` since it will use any existing lock
files if they exist. You can also check out the complete CI test pipeline in
the [run](https://github.com/nickjj/docker-django-example/blob/main/run) file
under the `ci:test` function.

#### In production:

This is usually a non-issue since you'll be pulling down pre-built images from
a Docker registry but if you decide to build your Docker images directly on
your server you could run `docker-compose build` as part of your deploy
pipeline.


## Additional resources

Now that you have your app ready to go, it's time to build something cool! If
you want to learn more about Docker, Django and deploying a Django app here's a
couple of free and paid resources. There's Google too!

### Learn more about Docker and Django

#### Official documentation 

- <https://docs.docker.com/>
- <https://docs.djangoproject.com/en/3.2/>

#### Courses / books

- William Vincent has a bunch of [beginner and advanced Django
  books](https://gumroad.com/a/139727987). He also co-hosts the [Django
  Chat](https://djangochat.com/) podcast

### Deploy to production





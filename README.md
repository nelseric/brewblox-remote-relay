# Boilerplate code for Brewblox service implementations

There is some boilerplate code involved when creating a Brewblox service. This repository can be forked to avoid having to do the boring configuration.

You're free to use whatever editor or IDE you like, but we preconfigured some useful settings for Visual Studio Code.

Everything listed under **Required Changes** must be done before the package works as intended.

## How to use

* Make sure Python3.8 is installed
  * `sudo add-apt-repository ppa:deadsnakes/ppa`
  * `sudo apt install python3.8 python3.8-dev`
* Fork this repository to your own Github account or project.
* Follow all steps outlined under the various **Required Changes**.
* Start coding your service =)
    * To test, run `pipenv run pytest`


## Files

---
### [setup.py](./setup.py)
Used to create a distributable and installable Python package. See https://docs.python.org/3.8/distutils/setupscript.html for more information.

**Required Changes:**
* Change the `name` variable to your project name. This is generally the same as the repository name. This name is used when installing the package through Pip. </br> It is common for this name to equal the package name, but using "`-`" as separator instead of "`_`".
* Change the `url` parameter to the url of your repository.
* Change the `author` parameter to your name.
* Change the `author_email` parameter to your email.


---
### [tox.ini](./tox.ini)
Developer tools such as [Pytest](https://docs.pytest.org/en/latest/), [Flake8](http://flake8.pycqa.org/en/latest/), and [Autopep8](https://github.com/hhatto/autopep8) use this file to find configuration options.

**Required Changes:**
* Change `--cov=YOUR_PACKAGE` to refer to your module name.
* The `--cov-fail-under=100` makes the build fail if code coverage is less than 100%. It is optional, but recommended. Remove the `#` comment character to enable it.


---
### [.env](./.env)
Project-specific environment variables can be stored here. `Pipenv` will automatically load it when executing a command in `pipenv run`.

By default, the names of the Docker and Github repositories are stored here. They are read during the CI build.

**Required Changes:**
* Change `DOCKER_REPO=you/your-package` to match the name of your docker image.


---
### [Pipfile](./Pipfile)
[Pipenv](https://docs.pipenv.org/) is used to streamline development. It manages dependencies and virtual environments. It also automatically loads environment variables declared in `.env`.

`Pipfile` lists all dependencies. Everything under [packages] is needed for the package to run, while everything under [dev-packages] is needed to run the tests.

You can use `pipenv install <package name>` or `pipenv install --dev <package name>` to add dependencies.

**Note:** There is overlap between the [packages] section in `Pipfile`, and the `install_requires=[]` list in `setup.py`. The rule of thumb is that if you need an external package to run, you should add it to both.

**Required Changes:**
* Install pipenv (run `sudo pip3 install pipenv`)
* Update the `Pipfile.lock` file (run `pipenv lock`)
* Install all packages (run `pipenv sync -d`)


---
### [MANIFEST.in](./MANIFEST.in)
This file lists all non-code files that should be part of the package.
See https://docs.python.org/3.8/distutils/sourcedist.html#specifying-the-files-to-distribute for more info.

For a basic service, you do not need to change anything in this file.


---
### [.editorconfig](./.editorconfig)
This file contains [EditorConfig](https://editorconfig.org/) configuration for this project. </br>
Among other things, it describes per file type whether it uses tabs or spaces.

For a basic service, you do not need to change anything in this file.
However, it is recommended to use an editor that recognizes and uses `.editorconfig` files.


---
### [README.md](./README.md)
Your module readme (this file). It will be the package description on Pypi.org, and automatically be displayed in Github.

**Required Changes:**
* Add all important info about your package here. What does your package do? How do you use it? What is your favorite color?


---
### [YOUR_PACKAGE/](./YOUR_PACKAGE/)
[\_\_main\_\_.py](./YOUR_PACKAGE/__main__.py),
[events_example.py](./YOUR_PACKAGE/events_example.py),
[http_example.py](./YOUR_PACKAGE/http_example.py),
[poll_example.py](./YOUR_PACKAGE/poll_example.py)

Your module. The directory name is used when importing your code in Python.

You can find examples for common service actions here.

**Required Changes:**
* Rename to the desired module name. This name can't include "`-`" characters. </br>
It is common for single-module projects to use "`-`" as a separator for the project name, and "`_`" for the module. </br>
For example: `your-package` and `your_package`.
* Change the import statements in .py files from `YOUR_PACKAGE` to your package name.

---
### [test/conftest.py](./test/conftest.py)
Project-level pytest fixtures. Some useful fixtures for testing any brewblox_service implementation are defined here. See tests in https://github.com/BrewBlox/brewblox-service/tree/develop/test for examples on how to use.


**Required Changes:**
* Change the import from `YOUR_PACKAGE` to your package name.


---
### [test/test_http_example.py](./test/test_http_example.py)
An example on how to test aiohttp endpoints you added. Feel free to remove this once you no longer need it.


---
### [docker/Dockerfile](./docker/Dockerfile)
A docker file for running your package. To build the image for both desktop computers (AMD), and Raspberry Pi (ARM):


``` sh
REPO=you/your-package
TAG=local

# Buildx is an experimental feature
export DOCKER_CLI_EXPERIMENTAL=enabled

# Enable the QEMU emulator, required for building ARM images on an AMD computer
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

# Remove previous builder
docker buildx rm bricklayer || true

# Create and use a new builder
docker buildx create --use --name bricklayer

# Bootstrap the newly created builder
docker buildx inspect --bootstrap

# Build the image for amd and arm
# Give the image a tag
# Push the image to the docker registry
docker buildx build \
    --push \
    --platform linux/amd64,linux/arm/v7 \
    --tag "$REPO":"$TAG" \
    docker
```

While you are in the same shell, you don't need to repeat all the commands before the actual build.

If you want to use the image locally, run the build command like this:

``` sh
docker buildx build \
    --load \
    --platform linux/amd64 \
    --tag "$REPO":"$TAG" \
    docker
```


**Required Changes:**
* Rename instances of `YOUR-PACKAGE` and `YOUR_PACKAGE` in the docker file to desired project and package names.

---
### [azure-pipelines.yml](./azure-pipelines.yml)
[Azure](https://dev.azure.com) can automatically test and deploy all commits you push to GitHub. If you haven't enabled Azure Pipelines for your repository: don't worry, it won't do anything.

To deploy your software, you will also need to create a [Docker Hub](https://hub.docker.com/) account, and register your image as a new repository.

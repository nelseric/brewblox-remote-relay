# Boilerplate code for Brewblox service implementations

There is some boilerplate code involved when creating a Brewblox service.
This repository can be forked to avoid having to do the boring configuration.

You're free to use whatever editor or IDE you like, but we preconfigured some useful settings for [Visual Studio Code](https://code.visualstudio.com/).

Everything listed under **Required Changes** must be done before the package works as intended.

## How to use

- Install required dependencies (see below)
- Fork this repository to your own Github account or project.
- Follow all steps outlined under the various **Required Changes**.
- Start coding your service =)
  - To test, run `poetry run pytest`
  - To lint, run `poetry run flake8`

## Install

Install [Pyenv](https://github.com/pyenv/pyenv):

```bash
sudo apt-get update -y && sudo apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev \
xz-utils tk-dev libffi-dev liblzma-dev git python3-venv

curl https://pyenv.run | bash
```

**Bash on Debian / Ubuntu**: to initialize pyenv on startup, press Ctrl-X, Ctrl-E in your terminal to open the batch command editor.
Paste the code block below, and press Ctrl-X to save and run.

```bash
sed -Ei -e '/^([^#]|$)/ {a \
export PYENV_ROOT="$HOME/.pyenv"
a \
export PATH="$PYENV_ROOT/bin:$PATH"
a \
' -e ':a' -e '$!{n;ba};}' ~/.profile
echo 'eval "$(pyenv init --path)"' >>~/.profile

echo 'eval "$(pyenv init -)"' >> ~/.bashrc
```

If you use a different shell, see the relevant instructions at <https://github.com/pyenv/pyenv>.

To apply the changes, run:

```bash
exec $SHELL --login
```

Install Python 3.9:

```bash
pyenv install 3.9.9
```

Install [Poetry](https://python-poetry.org/):

```bash
curl -sSL https://install.python-poetry.org | python3 -

exec $SHELL --login
```

Configure and install the environment used for this project. \
**Run in the root of your cloned project**

```bash
pyenv shell 3.9.9
poetry env use 3.9.9
poetry install
```

During development, you need to have your environment activated.
When it is activated, your terminal prompt is prefixed with `(.venv)`.

Visual Studio code with suggested settings does this automatically whenever you open a .py file.
If you prefer using a different editor, you can do it manually by running:

```bash
poetry shell
```

Install [Docker](https://www.docker.com/101-tutorial):

```bash
curl -sL get.docker.com | sh

sudo usermod -aG docker $USER

reboot
```

## Files

---

### [docker/Dockerfile](./docker/Dockerfile)

A docker file for running brewblox remote relay. To build the image for both desktop computers (AMD64), Raspberry Pi (ARM32), and Raspberry Pi 64-bit (ARM64):

Prepare the builder (run once per shell):

```bash
# Enable the QEMU emulator, required for building ARM images on an AMD computer
docker run --rm --privileged multiarch/qemu-user-static --reset -p yes

# Remove previous builder
docker buildx rm bricklayer || true

# Create and use a new builder
docker buildx create --use --name bricklayer

# Bootstrap the newly created builder
docker buildx inspect --bootstrap
```

Build:

```bash
REPO=ghcr.io/you/brewblox-remote-relay
TAG=local

# Will build your Python package, and copy the results to the docker/ directory
bash docker/before_build.sh

# Set image name
# Build the image for multiple architectures
# - AMD64 -> linux/amd64
# - ARM32 -> linux/arm/v7
# - ARM64 -> linux/arm64/v8
# Push the image to the docker registry
docker buildx build \
    --tag $REPO:$TAG \
    --platform linux/amd64,linux/arm/v7,linux/arm64/v8 \
    --push \
    docker
```

While you are in the same shell, you don't need to repeat the build preparation.

If you only want to use the image locally, run the build commands like this:

``` sh
REPO=ghcr.io/you/brewblox-remote-relay
TAG=local

# Will build your Python package, and copy the results to the docker/ directory
bash docker/before_build.sh

# Set image name
# Load image for local use
# This only builds for the current architecture (AMD64)
docker buildx build \
    --tag $REPO:$TAG \
    --load \
    docker
```

**Required Changes:**

- Rename instances of `brewblox-remote-relay` and `brewblox_remote_relay` in the docker file to desired project and package names.

---

### [build.yml](./.github/workflows/build.yml)

Github can automatically test, build, and deploy all commits you push.
When enabled, this configuration will run tests, and then build a Docker image.

By default, the image is pushed to the Github Container Registry (ghcr.io).
If you want to use an alternative registry like Docker Hub, you can do this by changing the login step,
and then omitting the `ghcr.io/` prefix to your image.

When first pushed, Github sets the visibility of the image to `Internal`.
To make it publicly available:

- Navigate to the Github page of your repository.
- Click on the image name under "Packages" on the right-hand side of the repository page.
- Click on "Package settings" on the right-hand side of the package page.
- Scroll down to the "Danger Zone", and click "Change package visibility".
- Set visibility to "Public", and type the name of the image to confirm.

**Required Changes:**

- Remove the `if: false` line in the `build` job to enable CI.
- Set the `DOCKER_IMAGE` variable to the desired Docker image name.

That's it. Happy coding!

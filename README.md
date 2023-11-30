# Poetry Showcase
This README will incorporate the most important concepts of Poetry

## Installation

Follow the installation procedure in the Docs. I use this command:

```sh
curl -sSL https://install.python-poetry.org | python3 -
```

## Initialisation

### New Project

To create a new folder which has the most minimal structure for a poetry project use:

```sh
poetry new <python-module-name>
```

This will create a structure like in the `./my-normal-package` folder. You can play around with some options that you can find with `poetry new --help` and, for example, change the name of the resulting module, resulting in the `custom-dir-name` folder which was created with this:

```shell
poetry new custom-name --name real-module
```

> Note: This will use the current python as a minimum requirement. You can change that in the `pyproject.toml`

### Current Project
If you want to use poetry in an existing python repo use `poetry init` to start an interactive declaration of the `pyproject.toml` file.

> Make sure that the project name matches the python package name (dashes are translated to underscores)

#### Migrate from requirements.txt

You can just manually install all dependencies in the requirements. A quick Shell script for this is:
```sh
poetry add $( cat requirements.txt )
```

## Usecases

### Package Manager
Poetry manages you dependencies in that you can install and update them. The versions installed will be checked against each other and the python version allowed for compatibility.

To add a package dependency use `poetry add <package>`. This will add the latest compatible version of that package to the `pyproject.toml` and update the `poetry.lock`.

If you pull the repo a simple `poetry install` will install all dependencies that are in the `poetry.lock`. No need to use the `add`-command. This won't update the `poetry.lock` file.

If you want to update all the packages use `poetry update`. This will update the packages to the latest possible compatible version and update the `poetry.lock` file.

`poetry show` will list all current packages installed.

#### Semver
The versions in the `pyproject.toml` file should be seen as constraints for the actual versions. A simple version like `1.2.3` means that only this version will be installed. A caret (^) in front of the version means that the first non-zero version will not be updated (`^1.2.3` will be updated to less than `2.0.0`, version like `1.8.3` are compatible). A tilde (~) means that the second non-zero version cannot be altered.

The install version higher than the ones in the `pyproject.toml`, you can use `poetry add <package>@<new-constraint>` or change the `pyproject.toml` and do `poetry install`.

### Virtual Environment
Poetry always installs your dependencies in a seperate virtual environment. On the first `poetry install` a new virtual environment for this repository is created in the poetry installation folder. `poetry env info` will list all important informations about the environment.

There is an option to tell poetry that it should create the virtual environment in a `./.venv` folder in the repository. `poetry config virtualenvs.in-project true --local` will enable this option for this repo. Without `--local` it will enabled for all repositories. You may need to `poetry env remove <python-version>` before doing a fresh `poetry install`

To bootstrap a new environment in a another python version change the constraint in the `pyproject.toml`, remove the current environment and do a `poetry install`. Make sure that `python[version]` executable is available in the `PATH`. I use `pyenv` to install multiple python versions and do `pyenv global 3.11 3.10 3.9 ...` to enable all version executables globally.

To run a python command with this virtual environment use `poetry run [command]` for example `poetry run python -m mypackage.main`.

## Modularisation

With a dependency line `my_package = {path='../my-package'}` you can install a local image in another poetry project. you can then use `import my_package` in python to use it.

### Multi-Package Poetry Package 
You can customize what will be included in the poetry package. Normally it's just the package with the same name as the name of the poetry project. If you have another structure or want to add multiple packages use the `packages` config in the `pyproject.toml` like in folder `./multi-package`. If you use multiple packages you can use both packages as if they each are in a separate project.

```toml
packages = [
    {include = "mypackage_extra_module", from = "extra_package"}, 
    {include = "mypackage"}
]
```

### Gitlab
To use a private Gitlab as a source of poetry packages you can use this script to upload to the package in the CI:

```yaml
image: python:latest

stages:
    - publish

publish:
    stage: publish
    script:
        - pip install poetry
        - poetry build
        - poetry config repositories.gitlab "${CI_API_V4_URL}/projects/${CI_PROJECT_ID}/packages/pypi"
        - poetry config http-basic.gitlab gitlab-ci-token "$CI_JOB_TOKEN"
        - poetry publish --repository gitlab
```

To use that package in a project you have to declare an extra source in you `pyproject.toml`:

```toml
[[tool.poetry.source]]
name = "gitlab"
priority = "supplemental"
# either use the group-id of the parent group of all python projects or exact package urls
url = "<gitlab-url>/api/v4/groups/<group-id>/-/packages/pypi/simple"
url = "<gitlab-url>/api/v4/projects/<project-id>/packages/pypi/simple"
```

Then you can add your package with `poetry add <package>`

## Miscellaneous

### Vscode

You can change the python interpreter of Vscode to the one in the virtual environment. Look into `poetry env info` or use `poetry run which python` to find the correct python version of the virtual environment.

### Network Fixes
If you are using a Company Proxy, set the `PIP_CERT` environment variable
For `poetry` you have to set the `REQUESTS_CA_BUNDLE` env variable.

### Files

#### pyproject.toml
declares the configuration of your package. What dependencies it needs, which modules it exports and what it's name is.

#### poetry.lock
This locks the versions of the depencies in the package. Doing `poetry install` will install the versions present in that file even if there are newer versions available. `poetry update` will update the `poetry.lock` file and install newer versions.

#### poetry.toml
This alters the behaviour of poetry. Configurations like altering the position of the virtual environment, or changing the http request behaviour are stored there. 

poetry will look into all parent directories for `poetry.toml` files. The Option in the files that are closest to the directory will override the ones that are more up the folder hierearchy. 
<!--
.. title: How to Rock Python Packaging with Poetry and Briefcase
.. slug: python-packaging-with-poetry-and-briefcase
.. date: 2019-01-02 23:01:00 UTC-05:00
.. tags: BeeWare, Briefcase, Python, Poetry, packaging
.. category: Python
.. link: 
.. description: Learn the background of Python packaging and walk through packaging your own app with Poetry and Briefcase
.. type: text
-->

NOTE: Briefcase now automatically creates a new project with a `pyproject.toml`
file by running `briefcase new`. I would recommend following the [BeeWare
Tutorial](https://docs.beeware.org/en/latest/tutorial/tutorial-1.html) to setup
a new project if you want to use Briefcase for packaging it.

As part of modernizing [Gaphas](https://github.com/gaphor/gaphas), the
diagramming widget for Python, I took another look at what the best practices
are for packaging and releasing a new version of a Python library or
application. There are new configuration formats and tools to make packaging
and distributing your Python code much easier.

# A Short Background on Packaging

There are two main use cases for packaging:

1. Packaging a Library - software that other programs will make use of.
2. Packaging an Application - software that a user will make use of.

This may not be a completely accurate definition because software does not
always fit cleanly in to one of these bins, but these use cases will help to
keep focus on what exactly we are trying achieve with the packaging.

### The Library

The goal for packaging a library is to place it on the Python Packaging Index
(PyPI), so other projects can `pip install` it. In order to distribute a
library, the standard format is the Wheel. It allows for providing a built

distribution of files and metadata so that pip only needs to extract files out
of the distribution and move them to the correct location on the target system
for the package to be installed. In other words, nothing needs to be built and
re-compiled.

Previously if you wanted to
achieve this, it was common to have four configuration files:

1. setup.py - The setup script for building, distributing and installing
  modules using the Distutils.
2. requirements.txt - Allow easy install of requirements using `pip install -r`
3. setup.cfg - The setup configuration file
4. MANIFEST.in - The manifest template, directs sdist how to generate
  a manifest

### The Application

The goal for packaging an application is get it in the formats where you can
distribute it on the different platforms for easy installation by your users.
For Windows this is often an exe or msi. For macOS this is an app. For Linux
this is a deb, flatpak, appimage, or snap. There is a whole host of tools to
do this like:
[py2exe](https://py2exe.org), [py2app](https://py2app.readthedocs.io/),
[cx_Freeze](https://cx-freeze.readthedocs.io/),
[PyInstaller](https://pyinstaller.readthedocs.io/), and
[rumps](https://github.com/jaredks/rumps).

### pyproject.toml

On the packaging front, in May of 2016,
[PEP 518](https://www.python.org/dev/peps/pep-0518/) was created. The PEP does
a good job of describing all of the shortcoming of the setup script method to
specify build requirements. The PEP also specified a new configuration format
call pyproject.toml. If you aren't familiar with TOML, it is human-usable and is
more simple than YAML.

The pyproject.toml replaced those four configuration files above using two main
sections:

1. `[build-system] ` - The build-system table contains the minimum requirements
  for the build system to execute.
2. `[tool]` - The tool table is where different tools can have users specify
  configuration data.

### The Tools

Making use of this new configuration format, a tool called
[flit](https://github.com/takluyver/flit) has been around since 2015 as a
simple way to put Python Libraries on PyPI.

In 2017,
[Pipenv](https://github.com/pypa/pipenv) was created to solve pain points about
managing virtualenvs and dependencies for Python Applications by using a new
Pipfile to manage dependencies. The other major enhancement was the use of a
lock file. While a Wheel is the important output for a Library, for an
Application, the lock file becomes the important thing created for the project.
The lock file contains the exact version of every dependency so that it can be
repeatably rebuilt.

In 2018, a new project called [Poetry](https://github.com/sdispater/poetry)
combined some of the ideas from flit and Pipenv to create a new tool that aims
to further simplify and improve packaging. Like flit, Poetry makes use of the
pyproject.toml to manage configuration all in one place. Like Pipenv, Poetry
uses a lock file (poetry.lock) and will automatically create a virtualenv
if one does not already exist. It also has other advantages like exhaustive
dependency resolution that we will explore more thoroughly below.
  
For Application distribution, I am going to focus on a single tool called
[Briefcase](https://briefcase.readthedocs.io/) which along with the other set
of [BeeWare](https://pybee.org) tools and libraries allows for you to
distribute your program as a native application to Windows, Linux, macOS, iOS,
Android, and the web.

# Tutorial

With the background information out of the way, lets work through how you can
create a new Python project from scratch, and then package and distribute it.

### Initial Tool Installation

To do that, I am going to introduce one more tool (the last one I promise!)
called [cookiecutter](https://cookiecutter.readthedocs.io/). Cookiecutter
provides Python project templates, so that you can quickly get up to speed
creating a project that can be packaged and distributed without creating a
bunch of files and boilerplate manually.

To install cookiecutter, depending on your setup and operating system, from a
virtualenv you can run:
```bash
$ pip install cookiecutter
```

Next we are going to install Poetry. The recommended way is to run:
```bash
$ curl -sSL https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py | python
```

### TestPyPI Account Sign-Up

As part of this tutorial we will be publishing packages. If you don't already
have an account, please register for an account on
[TestPyPI](https://test.pypi.org/account/register/). TestPyPI allows you to try
distribution tools and processes without affecting the real PyPI.

### Create Your Project

To create the Python project, we are going to use the Briefcase template, so
run `cookiecutter` on this template:
```bash
$ cookiecutter https://github.com/pybee/briefcase-template
```
Cookiecutter will ask you for information about the project like the name,
description, and software licence. Once this is finished, add any additional
code to your project, or just keep it as is for this demo.

Change your directory to the app name you gave (I called mine dantestapp), and
initialize git:
```bash
$ cd dantestapp
$ git init
$ git add .
```

### Create a pyproject.toml Configuration

Poetry comes equipped to create a pyproject.toml file for your project, which
makes it easy to add it to an existing or new project. To initiliaze the
configuration run:

```bash
$ poetry init
```

The command guides you through creating your pyproject.toml config. It
automatically pulls in the configuration values from the briefcase-template
that we created earlier so using the default values by hitting enter after the
first six questions will be fine. This is what it provided for an output:

```bash
Package name [dantestapp]: 
Version [0.1.0]: 
Description []: 
Author [Dan Yeaw <dan@yeaw.me>, n to skip]: 
License []: MIT
Compatible Python versions [^3.7]: 
```

#### Define Dependencies
The configuration generator then asks for you to define your dependencies:
```bash
Would you like to define your dependencies (require) interactively? (yes/no) [yes]
```
Hit enter for yes.

For the next prompt `Search for package:` enter in briefcase. We are setting
briefcase as a dependency for our project to run.
```bash
Enter package # to add, or the complete package name if it is not listed: 
 [0] briefcase
 [1] django-briefcase
```
Type 0 to select the first option. and hit enter to select the latest version.
You now need to repeat this process to also add Toga as a dependency. Toga is
the native cross-platform GUI toolkit. Once you are done, hit enter again to
complete searching for other dependencies.

#### Define Development Dependencies

At the next prompt the config generator is now asking us to define our development
dependencies:
```bash
Would you like to define your dev dependencies (require-dev) interactively (yes/no) [yes]
```
Hit enter to select the default value which is yes.

We are going to make [pytest](https://pytest.org) a development dependency for
the project.

At the prompt `Search for package:` enter in pytest.

```bash
Found 100 packages matching pytest

Enter package # to add, or the complete package name if it is not listed: 
 [ 0] pytest
```
You will get a long list of pytest packages. Type 0 to select the first option.
and hit enter to select the latest version. Then hit enter again to complete
searching for other development dependencies.

#### Complete the Configuration

The final step of the configuration generator summaries the configuration that it created.
Notice that first three sections are tool tables for Poetry, and the final one is
the build-system table.

```bash
[tool.poetry]
name = "dantestapp"
version = "0.1.0"
description = ""
authors = ["Dan Yeaw <dan@yeaw.me>"]
license = "MIT"

[tool.poetry.dependencies]
python = "^3.7"
briefcase = "^0.2.8"
toga = "^0.2.15"

[tool.poetry.dev-dependencies]
pytest = "^4.0"

[build-system]
requires = ["poetry>=0.12"]
build-backend = "poetry.masonry.api"
```
The dependencies use a "caret requirement", like `python = "^3.7"`. This makes
use of [semantic versioning](https://semver.org/). So in this example if Python 3.8
is released, then it will automatically update to this version. But, it won't
update to 4.0 automatically, since that is a major version change. If we put in
our configuration `"^3.7.2"`, then it would automatically update to
3.7.3 which it is released, but not 3.8, since that is a new minor version.

There are also "tilde requirements" that are more restrictive. So if you enter `python = "~3.7"
it will only allow update to the next patch level, like from 3.7.2 to 3.7.3. The combination
of caret and tilde requirements allows you to get updates to your dependencies when they are released, but
puts you in control to ensure that incompatible changes won't break your app. Nice!

The final prompt asks: `Do you confim generation? (yes/no) [yes]`. Go ahead 
and hit enter to confirm. Congrats, you have generated a pyproject.toml
configuration!

### Install Dependencies

OK, the hard work is over, we have created our project and finished the configuration.
Now it is time to see how Poetry and Briefcase really shines.

To install the dependencies that you defined in the pyproject.toml, just run:
```bash
$ poetry install
```

Poetry includes an exhaustive dependency resolver, so it will now resolve all of the
dependencies it needs to install Briefcase, Toga, and pytest. It will also create
a `poetry.lock` file which ensures that anyone using your program would get the
exact same set of dependencies that you used and tested with.

Notice that we also did not create or specify a virtual environment. Poetry automatically
creates one prior to installing packages, if one isn't already activated. If you
would like to see which packages are installed and which virtual environment
Poetry is using you can run:
```bash
$ poetry show -v
or
$ poetry config --list
```

### Bundle and Run your Application for Platform Distribution

For a Python Application, you want to bundle the application and all of its
dependencies into a single package so that it can easily be installed on a users
platform without the user manually install Python and other modules.

Briefcase allows you to package and run your app using your platform:

```bash
(Windows) $ poetry run python setup.py windows -s
(macOS)   $ poetry run python setup.py macos -s
(Linux)   $ poetry run python setup.py linux -s
```
Your app will launch, will just be a blank window at this point.

Also notice that it creates a folder with the platform name that you used
above. Inside this folder, Briefcase has packaged your app for distribution on
your platform. Briefcase also has distribution options for android, ios, and
django.

### Build your Library for Distribution on PyPI


```bash
$ poetry build

Building dantestapp (0.1.0)
 - Building sdist
 - Built dantestapp-0.1.0.tar.gz

 - Building wheel
 - Built dantestapp-0.1.0-py3-none-any.whl
```

The source distribution (sdist) and wheel are now in a new `dist` folder.

### Publish your Library to PyPI

First we are going to add the TestPyPI repository to Poetry, so that it knows
where to publish to. The default location is to the real PyPI.
```bash
$ poetry config repositories.test-pypi https://test.pypi.org/legacy/
```
Now simply run:
```bash
$ poetry publish -r test-pypi
```
The `-r` argument tells Poetry to use the repository that we configured. Poetry
then will ask for your username and password. Congrats! Your package is now
available to be viewed at https://test.pypi.org/project/your-project-name/ and
can be pip installed with `pip install -i https://test.pypi.org/simple/
your-project-name`.

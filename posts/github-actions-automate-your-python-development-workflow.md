<!--
.. title: GitHub Actions: Automate Your Python Development Workflow
.. slug: github-actions-automate-your-python-development-workflow
.. date: 2019-11-28 14:51:33 UTC-05:00
.. tags: Python, GitHub, programming, CI/CD
.. category: Python 
.. link: 
.. description: GitHub Actions makes it easy to automate all your software workflows, now with CI/CD. Build, test, and deploy your code right from GitHub. Make code reviews, branch management, and issue triaging work the way you want. This post tells you how to take advantage of Actions for your Python library or app.
.. type: text
-->

At GitHub Universe 2018, GitHub launched GitHub Actions in beta. Later in
August 2019, GitHub announced the expansion of GitHub Actions to include
Continuous Integration / Continuous Delivery (CI/CD). At Universe 2019,
GitHub
[announced](https://github.blog/2019-11-13-universe-day-one/#github-actions)
that Actions are out of beta and generally available. I spent the last few
days, while I was taking some vacation during Thanksgiving, to explore GitHub
Actions for automation of Python projects.

With my involvement in the [Gaphor](https://gaphor.org) project, we have a GUI
application to [maintain](https://github.com/gaphor/gaphor), as well as two
libraries, a diagramming widget called
[Gaphas](https://github.com/gaphor/gaphas), and we more recently took over
maintenance of a library that enables multidispatch and events called
[Generic](https://github.com/gaphor/generic). It is important to have an
efficient programming workflow to maintain these projects, so we can spend more
of our open source volunteer time focusing on implemented new features and
other enjoyable parts of programming, and less time doing manual and boring
project maintenance.

In this blog post, I am going to give an overview of what CI/CD is, my previous
experience with other CI/CD systems, how to test and deploy Python applications
and libraries using GitHub Actions, and finally highlight some other Actions
that can be used to automate other parts of your Python workflow.

CI is the practice of frequently integrating changes to code with the existing
code repository. CD then extends CI by making sure the software checked in
to the master branch is always in a state to be deployed to users, and
automates the deployment process. For open source projects on GitHub or GitLab,
the workflow often looks like:

1. The latest development is on the mainline branch called master.
2. Contributors create their own copy of the project, called a fork, and then
clone their fork to their local computer and setup a development environment.
3. Contributors create a local branch on their computer for a change they want
to make, add tests for their changes, and make the changes.
4. Once all the unit tests pass locally, they commit the changes and push them
to the new branch on their fork.
5. They open a Pull Request to the original repo.
6. The Pull Request kicks off a build on the CI system automatically, runs
formatting and other lint checks, and runs all the tests.
7. Once all the tests pass, and the maintainers of the project are good with
the updates they merge the changes back to the master branch.

Either in a fixed release cadence, or occasionally, the maintainers then add a
version tag to master, and kickoff the CD system to package and release a new
version to users.

Since most open source projects didn't want the overhead of maintaining their
own local CI server using software like Jenkins, the use of cloud-based or
hosted CI services became very popular over the last 7 years. The most
frequently used of these was Travis CI with Circle CI a close second. Although
both of these services introduced initial support for Windows over the last
year, the majority of users are running tests on Linux and macOS only. It is
common for projects using Travis or Circle to use another service called
AppVeyor if they need to test on Windows.

I think the popularity of Travis CI and the other similar services is based on
how easy they were to get going with. You would login to the service with your
GitHub account, tell the service to test one of your projects, add a YAML
formatted file to your repository using one of the examples, and push to repo
to trigger your first build. Although these services are still hugely popular,
2019 was the year that they started to lose some of their momentum. In January
2019, a company called Idera bought Travis CI. In February Travis CI then
[layed-off](https://twitter.com/alicegoldfuss/status/1098604563664420865) a lot
of their senior engineers and technical staff.

The 800-pound gorilla entered the space in 2018, when Microsoft bought GitHub
in June and then rebranded their Visual Studio Team Services ecosystem and
launched Azure Pipelines as a CI service in September. Like most of the popular
services, it was free for open source projects. The notable features of this
service was that it launched supporting Linux, macOS, and Windows, and it
allowed for 10 parallel jobs. Although the other services offer parallel
builds, on some platforms they are limited for open source projects, and I
would often be waiting for a server called an "agent" to be available with
Travis CI. Following the lay-offs at Travis CI, I was ready to explore other
services to use, and Azure Pipelines was the new hot CI system.

In March 2019, I was getting ready to launch version 1.0.0 of Gaphor after
spending a couple of years helping to update it to Python 3 and PyGObject. We
had been using Travis CI, and we were lacking the ability to test and package
the app on all three major platforms. I used this as an opportunity to learn
Azure Pipelines with the goal of being able to fill this gap we had in our
workflow.

My takeaways from this experience is that Azure Pipelines is lacking much of the
ease of use that Travis CI has, but has other huge positives including build
speed and the flexibility and power to create complex cross-platform workflows.
Developing a complex workflow on any of these CI systems is challenging because
the feedback you receive takes a long time to get back to you. In order to
create a workflow, I normally:

1. Create a branch of the project I am working on
2. Develop a YAML configuration based on the documentation and examples available
3. Push to the branch, to kickoff the CI build
4. Realize that something didn't work as expected after 10 minutes of waiting
for the build to run
5. Go back to step 2 and repeat, over and over again

I also thought the documentation was often lacking good examples of complex
workflows, and was not very clear on how to use each step. This drove even
more trial and error, which requires a lot of patience as you are working on a
solution. After a lot of effort, I was able to complete a configuration that
tested Gaphor on Linux, macOS, and Windows. I also was able to partially get the
CD to work by setting up Pipelines to add the built dmg file for macOS to a
draft release when I push a new version tag. A couple of weeks ago, I was also
able build and upload Python Wheel and source distribution, along with the
Windows binaries built in [MSYS2](https://www.msys2.org).

Despite the challenges getting there, the result was very good! Azure Pipelines
is screaming fast, about twice as fast as Travis CI was for my complex
workflows (25 minutes to 12 minutes). The tight integration that allows testing
on all three major platforms was also just what I was looking for.

With all the background out of the way, now enters GitHub Actions. Although I
was very pleased with how Azure Pipelines performs, I thought it would be nice
to have something that could better mix the ease of use of Travis CI with the
power Azure Pipelines provides. I hadn't made use of any Actions before
trying to replace both Travis and Pipelines on the three Gaphor projects that
I mentioned at the beginning of the post.

I started first with the libraries, in order to give GitHub Actions a try with
some of the more straightforward workflows before jumping in to converting Gaphor
itself. Both Gaphas and Generic were using Travis CI. The workflow was pretty
standard for a Python package:

1. Run lint using [pre-commit](https://pre-commit.com) to run
[Black](https://black.rtd.io) over the code base.
2. Use a matrix build to test the library using Python 2.7, 3.6, 3.7, and 3.8
3. Upload coverage information

To get started with GitHub Actions on a project, go to the Actions tab on the
main repo:
![Actions tab image]

Based on your project being made up of mostly Python, GitHub will suggest three
different workflows that you can use as templates to create your own:

1. Python application - test on a single Python version
2. Python package - test on multiple Python versions
3. Publish Python Package - publish a package to PyPI using Twine

For these libraries, the 2nd workflow was the closest for what I was looking
for, since I wanted to test on multiple versions of Python. I selected the
`Set up this workflow` option. GitHub then creates a new draft YAML file based
on the template that you selected, and places it in the `.github/workflows`
directory in your repo. At the top of the screen you can also change the name
of the YAML file from `pythonpackage.yml` to any filename you choose. I called
mine `build.yml`, since calling this type of workflow a build is the
nomenclature I am familiar with.

As a side note, the online editor that GitHub as implemented in for creating
Actions is quite good. It includes full autocomplete (toggled with Ctrl+Space),
and it actively highlights errors in your YAML file to ensure the correct
syntax for each workflow.

The top of each workflow file are two keywords: `name` and `on`. The `name` sets
what will be displayed in the Actions tab for the workflow you are creating. If
you don't define a name, then the name of the YAML file will be shown as the
Action is running. The `on` keyword defines what will cause the workflow to be
started. You can define multiple events that will start the workflow by adding
them as a comma separated list. The template uses a value of `push`, which means
that the workflow will be kicked off when you push to any branch in the
repo. Here is an example of how I set these settings for my libraries:

```yaml
name: Build
on:
  pull_request:
  push:
    branches: master
```

Instead of running this workflow on any push event, I wanted a build to happen
during two conditions:

1. Any Pull Request
2. Any push to the master branch

You can see how that was configured above. Being able to start a workflow on
any type of event in GitHub is extremely powerful, and it one of the advantages
of the tight integration that GitHub Actions has.

The next section of the YAML file is called `jobs`, this is where each main
block of the workflow will be defined as a job. The jobs will then be further
broken down in to steps, and multiple commands can be executed in each step.
Each job that you define is given a name. In the template the job is named
`build`, but there isn't any special significance of this name. In the
template, they also are running a lint step for each version of Python being
tested against. I decided that I wanted to run lint once as a separate job, and
then once that is complete, all the testing can be kicked off in parallel.

In order to add lint as a separate job, I created a new job called `lint` nested
within the `jobs` keyword. Once again, this job name doesn't have any special
meaning, it is just a unique name that you give it. Below is an example of my
lint job:

```yaml
jobs:
  lint:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v1
      - name: Setup Python
        uses: actions/setup-python@v1
        with:
          python-version: '3.x'
      - name: Install Dependencies
        run: |
          pip install pre-commit
          pre-commit install-hooks
      - name: Lint with pre-commit
        run: pre-commit run --all-files
```

Next comes the `runs-on` keyword which defines which platform GitHub Actions
will run this job on, and in this case I am running on linting on the latest
available version of Ubuntu. The `steps` keyword is where most of the workflow
content will be, since it defines each step that will be taken as it is run.
Each step optionally gets a name, and then either defines an Action to use or a
command to run.

Let's start with the Actions first, since they are he first two steps in my
lint job. The keyword for an Acion is `uses`, and the value is the action repo
name and the version. I think of Actions as a library, a reusable step that I
can use in my CI/CD pipeline without having to reinvent the wheel. GitHub
developed These first two Actions that I am making use of, but you will see
later that you can make use of any Actions posted by users, and even create
your own using the Actions SDK and some TypeScript. I am now convinced that this
is the "secret sauce" of GitHub Actions, and will be what makes this service
truly special. I will discuss more about this later.

These two Actions I am using, clone a copy of the code I am testing from my repo
and sets up Python 3. Actions often use the`with` keyword for the configuration
options, and in this case I am telling the `setup-python` action to use a newer
version from Python 3.

The last two steps of the linting job are using the `run` keyword. Here I am
defining commands to execute that aren't covered by an Action. As I mentioned
earlier, I am using pre-commit to run Black over the project and check that the
code formatting is correct. I have this broken up in to two steps:

1. Install pre-commit, and install the pre-commit hook environments
2. Run Black against all the files in the repo






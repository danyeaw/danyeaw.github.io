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
[Generic](https://github.com/gaphor/generic). It is important that we have
an efficient programming workflow to maintain these projects so we can spend
more of our open source volunteer time focusing on implemented new features
and other enjoyable parts of programming, and less time doing manual and boring
project maintenance.

In this blog post, I am going to give an overview of what CI/CD is, my previous
experience with other CI/CD systems, how to test and deploy Python applications
and libraries using GitHub Actions, and finally highlight some other Actions
that can be used to automate other parts of your Python workflow.

CI is the practice of frequently integrating changes to code with the existing code
repository. CD then extends CI by making sure that the software checked in to the
master branch is always in a state that can be deployed to users, and automates the
deployment process. For open source projects on GitHub or GitLab, the workflow often
looks like:

1. The latest development is on the mainline branch called master.
2. Contributors create their own copy of the project, called a fork, and then clone their
fork to their local computer and setup a development environment.
3. Contributors create a local branch on their computer for a change they want to make,
add tests for their changes, and make the changes.
4. Once all the unit tests pass locally, they commit the changes and push them to the new
branch on their fork.
5. They open a Pull Request to the original repo.
6. The Pull Request kicks off a build on the CI system automatically, runs formatting
and other lint checks, and runs all of the tests.
7. Once all the tests pass, and the maintainers of the project are good with the updates
they merge the changes back to the master branch.

Either in a fixed release cadence, or occasionally, the maintainers then add a version
tag to master, and kickoff the CD system to package and release a new version to users.

Since most open source projects didn't want the overhead of maintaining their own local
CI server using software like Jenkins, the use of cloud-based or hosted CI services became
very popular. The most frequently used of these was Travis CI with 
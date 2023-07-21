---
title: "Harnessing the best software tools for reproducible analysis"
date: 2023-07-16T13:45:24-04:00
draft: false
tags: ["analytical-integrity", "dag"]
weight: 1
---

Among those who create and use data analyses, it is commonly understood that a trustworthy analysis must be reproducible. In other words, given the same problem statement, an independent analyst should be able to gather their own data and analyze it to reach the same conclusions. If this isn’t true, it raises strong suspicions about the accuracy of the original analysis.

In practice, there are rarely sufficient resources available to fully replicate an analysis from scratch, except for the most controversial, significant, or risky decisions. Consequently, most practical analyses are never subjected to such scrutiny.

However, drawing on advances in software engineering tools and techniques can dramatically lower the cost and difficulty of performing an analysis reproducibly.  In this post, I offer a practical approach to reproducible analysis with examples that could easily be adopted by any Python-centric Data Science team.  I'll also explain why making an investment in reproducibility is more important than you might think based on my experience in managing model risk at a large bank.

## The Peer Reviewer's Perspective

The term "reproducible" is frequently used with a weaker criterion in mind: an independent analyst should be able to achieve the same results using the original analysts' documentation, data, and code. Although this does not guarantee the original analysis is correct, as you may simply be repeating the original mistakes, it is a meaningful step towards establishing trust. It ensures that the data and analysis were conducted with sufficient transparency and clarity to allow the results to be verified.  For the rest of the post, I will be speaking about reproducibility in this sense.

For analysts racing against deadlines and business leaders not involved in the low-level processes of analysis, these concerns may seem theoretical. When time is limited and there's a pressing question, it's natural to focus on getting an answer rather than the methods used to obtain it.  In the long term, however, better methods don’t just improve accuracy and reliability.  They also make the short-term sprints more efficient and manageable.

As a Model Risk Officer (MRO) at a large bank, I experienced this in a very tangible way.  I was responsible for challenging Data Science teams across the organization to identify, quantify, and effectively manage the risks of using Machine Learning models through independent validation of their development process.  The ultimate goal of model validation was to ensure that the model’s strengths and weaknesses were well understood, operational monitoring was adequate, and material analytical errors were identified and corrected.  Above a moderate risk threshold, our validation process always included reproducing some or all of the business team’s analytical results.

As modelers ourselves, we would have loved to focus our effort on debating interesting theoretical and methodological choices or testing innovative new algorithmic approaches.  In practice, we spent most of our time coaching Data Science teams to provide sufficient code, data, and documentation for us to clearly understand what they did.  Essentially, we were forced into the role of “reproducibility coach”, helping teams build the foundations to manage risk more than actually managing it.

This was all the more frustrating because I observed wide variability in the effort required to reproduce a team’s modeling results.  With the least prepared teams, it literally took months to years of meetings to get the information I needed to do my job.  The most prepared teams knew how to hand over a model development package that I could reproduce in an afternoon.  These same teams could also turn around an iteration on their analysis more quickly, as their processes were more systematic.

The variability wasn’t random either.  Teams of junior modelers who had never managed a model through its operational life cycle and siloed teams that developed models but were not responsible for managing them operationally tended to require the most coaching.  Experienced modelers who had experience using their models operationally and being accountable for their failures were most likely to have invested in a reproducible development process because they knew from experience that it made their lives easier in the long run, independent of the need to complete my validation process.  Over time, team members come and go and even a model’s original developers shift their focus to new projects, and at this stage early investments in transparency greatly reduce the cognitive load on the team and rates of human error.

The big takeaways from my time as an MRO were:

* Material errors in complex analysis are way more common than we would like to believe, [even when the team working on it is stellar](https://www.hashadatascience.com/analytical_integrity/#hiring-smart-people-isnt-enough)
* A reproducible development process is table stakes for managing analytical risk effectively
* **With the right tools and techniques, a reproducible development process doesn’t need to be expensive or difficult.**

## Benefiting from Innovations in Software Engineering

Driven by the exponential increase in size and complexity of modern software projects and software teams, software engineers have built tools to automate or streamline every stage of the software lifecycle, from installation to code editing, testing, and deployment.

“Data Scientist” is more of a marketing term than a precisely defined role.  But if I had to choose one skill set that differentiates a Data Scientist from the larger population of analysts and scientists, it’s that they write code to perform analysis.  Additionally, the trend in the past decade has been away from writing code in specialized analytical languages like Matlab and R towards the use of general-purpose programming languages that are widely used by the software engineering community, such as Python and Java.  This growing overlap in tools presents a major opportunity for Data Scientists to benefit from software engineers’ extraordinary progress developing tools and methods to manage the complexity of large software projects.

At a high level, to reproduce a computational task you need to:

1. Build and install the same code and software libraries
2. Gather the same input data
3. Re-run the process steps in the right order with the same configuration settings

Additionally, to manage the cognitive load on the humans overseeing these tasks, you need to:

4. Systematically track the revision history of your project and which revisions were used to produce results
5. Keep code and documentation in sync
6. Maintain tidy, well-organized code and data
7. Automate complex repetitive steps (especially testing)

The good news is that there are literally dozens of software tools available to help with every one of these needs.  By using established tools and patterns, we can push routine issues that create complexity into the background, and save our precious attention to grapple with the problems that are unique to the problem at hand.

## The Tools I Recommend

Many of the tools in my reproducible analysis Python stack are standard choices that I’ve seen used on almost every team I’ve worked with.  For others, there are many competing tools available.  The important thing is to adopt some tool in each of these categories that meets your needs, and develop a workflow that integrates them all.

**1) Build and install the same code and software libraries**

[poetry](https://python-poetry.org/docs/) - Poetry is a tool for dependency management and packaging in Python. It allows you to declare the libraries your project depends on and it will manage (install/update) them for you.  Unlike most Python environment management tools, Poetry ensures repeatable installs using a “lockfile” pattern.

**2) Gather the same input data, and 3) re-run the same process steps**

[DVC](https://dvc.org/doc/user-guide) - To me, DVC was the last missing piece enabling a truly reproducible Data Science workflow.  While software version control tools are almost universally adopted, they are not designed to work well with large data sets.  This leads to Data Scientists being urged to version their code but, paradoxically, not their data.  Before DVC, most data version control tools were hosted services tightly coupled to particular platforms for data storage and compute, which made them difficult to adopt in the context of a large corporation where hosted infrastructure must go through layers of architectural, budget, and security approvals.

In contrast, DVC is a free, open source Python package that can be downloaded and installed in seconds.  It stores large files and datasets outside of git, while storing lightweight metadata in git linking code versions to data versions.  It is extremely flexible, integrating with most common data storage solutions.  This means it can be adopted without disrupting existing data storage patterns in their organization.  It can track data stored on the user’s computer or hosted on any major cloud storage provider, such as AWS S3, Google Cloud Storage, or Microsoft Azure Blob Storage.  DVC users can create a remote repository to give stakeholders access to project data as easily as “git pull” gives access to code.

In addition to data versioning, DVC also provides a lightweight way to automate complex workflows by defining them as a Directed Acyclic Graph (DAG) of tasks and their dependencies.  If this is an unfamiliar concept for you, I strongly recommend checking out [my post on DAGs](https://www.hashadatascience.com/dags/). As with data versioning solutions, many more popular DAG schedulers rely on hosted infrastructure with steep learning curves, so DVC’s solution is refreshingly simple to use.

**4) Track the revision history of your project**

* [git](https://git-scm.com/book/en/v2) - Source code revision management (SCM) tools are basically table stakes for reproducibility.  Luckily, git is more or less universally adopted by both software engineering and Data Science teams at this point. It has been over a decade since I worked with a team that was using anything other than `git` for this purpose.  It’s a flexible and powerful tool, and many of the other tools in my stack are designed to integrate with it.
* [GitHub](https://github.com) - It’s hard for a team that collaborates on code to survive without a hosted source control repository, so GitHub is the one exception to my “no hosted infrastructure” rule.  It helps that it’s free for individuals and wildly common in the corporate world.  I’ve only worked on one team that used something else.  Their UX is category-defining, and they’ve kept up a steady stream of valuable new features like GitHub Copilot and GitHub Actions.

**5) Keep code and documentation in sync**

* [Sphinx](https://www.sphinx-doc.org/en/master/) - Sphinx allows you to write your documentation in relatively simple markup formats, then render it into beautifully formatted websites.  Combined with build automation, it can ensure that your documentation is always based on your latest code and data.
* [GitHub actions](https://docs.github.com/en/actions) - I’m no DevOps expert, but GitHub actions is the easiest build automation tool I’ve used.

**6) Tidy, well-organized code**

Writing code in a consistent, standard style makes your code easier for others to to read, debug, and maintain.  While there will always be a few holdouts who continue to argue over whether to indent with spaces or tabs, the Python community has aligned on the [PEP8 style conventions](https://peps.python.org/pep-0008/).  When I first learned Python, you actually had to know these conventions in detail to survive a code review with an experienced developer.  Now, adding the following tools to your project will automate 98% of the effort needed to follow the style standard, and will gently prompt you through the rest.

* [black](https://black.readthedocs.io/en/stable/)
* [isort](https://pycqa.github.io/isort/)
* [flake8](https://flake8.pycqa.org/en/latest/)
* [pre-commit](https://pre-commit.com/)

**7) Test automation**

* [pytest](https://docs.pytest.org/en/latest/) - The pytest framework makes it easy to write small, readable tests, and can scale to support complex functional testing for applications and libraries.  The majority of python projects use either `pytest` or `unittest`, and in my opinion pytest is more modern and flexible and easier to use.
* [GitHub actions](https://docs.github.com/en/actions) - Build automation tools make it simple to ensure that your automated tests run every time someone tries to update the project’s code, so that the code found there is always guaranteed to pass your tests.

## Bring it all together with a project template

Learning these tools from scratch is daunting, but I have developed a Data Science project template that takes care of the hard work of configuring and integrating the tools and lets a user start benefiting from them with just a few standard setup commands. It’s much easier to use them working within a project where they’ve already been integrated. In fact, I learned most of these tools that way, by contributing to existing projects where they were already being used. To see an example of how this works in practice, check out the [Getting Started](https://miltonmaps.hashadatascience.com/getting-started.html) page of my  [Milton Maps](https://miltonmaps.hashadatascience.com/) project.  In five standard commands, you get the code, data, and software necessary to run my analysis on your machine.  Behind the scenes, I set up this project with a [cookiecutter template](https://cookiecutter.readthedocs.io/en/stable/) that sets up a standard, functional skeleton project with all my standard tools pre-configured.

Using project templates like this is fairly common practice in other corners of the software engineering world, though it hasn’t caught on broadly with Data Scientists.  Several of the major web development frameworks like Django or Ruby on Rails are good examples. Nobody sits around before creating a new Rails project to figure out where they want to put their views; they just run `rails new` to get a standard project skeleton like everybody else. Because that default project structure is logical and reasonably standard across most projects, it is much easier for somebody who has never seen a particular project to figure out where they would find the various moving parts.  Ideally, that's how it should be when a colleague opens up your data science project.

A well-defined, standard project structure lets a newcomer dive into an analysis without having to spend a lot of time getting oriented. They don't have to read all of the code before knowing where to look for very specific things.  Well-organized code is self-documenting in that the organization itself indicates the function of specific components without requiring additional explanation.

In an organization with well-established standards,

* Managers have more flexibility to move team members between projects and domains, because less "tribal knowledge" is required
* Colleagues can quickly focus on understanding the substance of your analysis, without getting hung up on your “creative” approach to connecting to the database and organizing data.
* Conventional problems are solved in conventional ways, leaving more energy to focus on the unique challenges of each project
* New best practices and design patterns can quickly be projected to the team by updating the template.

Do you have a project or a team that might benefit from these best practices?  If so, maybe I can help?

<br>
<button class="contact-button" role="button" onclick="window.location.href='/contact/';">Get in touch!</button>











---
title: "What's a DAG and why do Data Scientists need them?"
date: 2023-02-07T08:18:03-05:00
draft: false
weight: 3
tags: ["dag"]
---

One of the themes that has motivated my career is helping Data Scientists be more productive by adopting best practices
from software engineering.  Data Scientists are often professional-quality data analysts with self-taught programming
skills.  They're smart enough to create a lot of complexity without always having the training to manage it efficiently. "DAGs"
are a key tool that should be in every Data Science Team's toolbox.

### My cautionary tale

{{< admonition tip "Not in the mood for a motivating personal story?" true >}}
Skip to the ["so what"](#dag-answer)
{{< /admonition >}}

When I started working at my first startup, [Bundle](https://techcrunch.com/2012/11/30/capital-one-acquires-bundle-a-data-driven-local-business-directory/), my skill set and approach to problem-solving was typical of
many early-career Data Scientists.  I had a lot of training in mathematics and an eagerness to solve real-word
problems with data. I was a competent amateur at writing code, having used a variety of scripting languages to
transform data and build models throughout my academic career.  I enjoyed learning and applying new languages
and tools, and was more than willing to pick up any new tool that would help me solve whatever problem was in
front of me.

Bundle was a company formed around a unique proprietary data set. Its CEO, Jaidev Shergill, was formerly a
managing director of Venture Investing at Citigroup. In that role, he developed a business concept to mine
customer transaction data from Citibank’s credit cards to generate insights that would help banking customers get
more value from their money.  Ultimately he felt so passionate about the idea that he decided to join the fledgling
company as its CEO.   It was launched with a two year license to access anonymized credit card transaction data
for Citibank’s 20+ million  credit card customers.

It had a clearly defined mission to help people “spend well and save well”, but when I joined they were still
actively experimenting to find product-market fit.  Their first product concept had just been launched: a personal
budgeting platform similar to Mint.com, but with budget guidance mined from the transactions of “people like you.”
So, my initial data science problems focused on the needs of that product: we had to be able to categorize each
transaction into the correct budget category based only on the description that appears on the statement. (We
weren’t getting a category code from the aggregation services that scraped the user’s transactions from
their banks).  We built a series of machine learning classifiers with progressively better categorization
accuracy, and found that we could be much more accurate if we first matched the transactions to a merchant
database, then categorized based on the merchant’s category.  This meant we needed to acquire merchant directory
data, and build an ETL pipeline to load it.  Around this point, we found the budgeting platform wasn’t getting
the traction we’d hoped for with users, so we pivoted to build a merchant review website, similar to Yelp but
based on “real customer behavior”.  For this use case, our first merchant database had significant gaps, so we
acquired another merchant database, and built another ETL pipeline to clean and load that.  We also needed to
dedupe merchant listings across the two databases, so we built a ML matcher and associated pipeline for that.

By this point, our data science team had grown to about 10 people, and we were still preparing each component
of this process as a separate manual process.  A data scientist would run the series of scripts that they had
personally developed and hand off the outputs to the person who needed them as inputs for the next phase.  This
was getting time-consuming and error-prone.  I wanted to be expanding and improving the product, and instead I
felt like I spent all my time babysitting overnight data loads.  We also wanted to be refreshing our product’s
data weekly instead of monthly, but the manual process was simply too time-consuming.  So we decided to automate
the process.  We spent a month or two writing scripts to run our scripts, and got the whole thing (sort of)
working, but I still felt like if one or two of the Data Scientists decided to leave the whole thing would fall
apart.

<center><img src="/images/Bundle Pipeline overview.png" alt="Complex manual pipeline"></center>

It was around this time that we started the due-diligence process to be acquired by Capital One, and the fact
that none of this complexity was documented started to get embarrassing.  My team spent a few days furiously
diagramming the system.  The image above served as a sort of “table of contents” of many pages of job
workflows, all of which had to be prepared by hand.

My story is pretty typical of the challenges that Data Science teams run into when they are lucky enough
to develop a data product that people want.  Software Engineers have developed standard tools for managing
complex workflows efficiently, but because Data Scientists’ training tends to focus on mathematics rather than
algorithms, the are often unaware of the tools.  Since they also tend to be very smart, they are prone to manage
the complexity in their heads until the system becomes absolutely unmanageable for anyone but the original authors.

### <a name="dag-answer"></a> DAGs are the answer

The established software engineering solution to this problem are workflow schedulers based on [Directed
Acyclic Graphs](https://en.wikipedia.org/wiki/Directed_acyclic_graph) (DAGs). A DAG is a graph data structure in which the graph nodes are individual tasks in the
workflow, and graph edges indicate the dependencies. “Directed” means the connections between tasks have a
direction; they point from the upstream task that must run first to the downstream task that depends on it.
“Acyclic” means there are no loops, or “cycles” in the graph.  While many practical a workflow to have
iterative cycles, it’s always possible to represent these as an acyclic graph as long as the cycle eventually
terminates.

{{< mermaid >}}
stateDiagram-v2
[*] --> a
a --> b
a --> c
b --> d
a --> d
a --> e
c --> e
d --> e
e --> [*]
{{< /mermaid >}}

Figure 2 shows an example DAG: here, tasks `b` and `c` can only run after `a` completes, task `d` depends on
the output of `a`,`b`, and `c`, and task `e` depends on the output of `a`,`c`, and `d`.

There are many benefits to using DAGs to model a complex workflow:

* **Consistency and Reproducibility**: once the workflow’s structure has been encoded as a DAG, a workflow
  scheduler will ensure that every step is executed in the correct order every time (unlike a tired Data Scientist
  working late to finish an analysis).
* **Parallelism**: if two or more tasks can be logically run in parallel (such as tasks b and c in the diagram),
  a workflow scheduler will usually do so, yielding big efficiency gains.
* **Clarity**: DAGs can be visualized automatically, and these visualizations are intuitive even for
  non-technical audiences.  The diagram of the Bundle process I shared above was essentially a DAG, but took me a
  day or two to create by hand.  Once we had migrated to a DAG workflow tool, we could automatically generate the
  workflow diagram every time the code was updated.

Indirectly, DAGs facilitate team growth and separation of duties.  If one team’s workflow output is the input to
another team’s workflow, you’ll be sending emails and scheduling meetings to arrange hand-offs every time the
process runs.  With a shared DAG framework, once the two teams agree to add the dependency to the workflow, it will
run automatically from then on.

### Data Scientists need DAGs too

DAGs are a no-brainer from a system’s engineering perspective, and you would have a hard time finding a Data
Engineering team that does not have a DAG workflow scheduler of some kind at the heart of their development
process.  Data Analyst and Data Science teams, on the other hand, are often not taking advantage of this powerful
tool in their day to day work because they simply lack the software engineering background to know how to apply
them.  This can lead to a number of problems:

* **Slow iterations**: A fully-developed model training process has a lot of steps in it.  You have to pull the data, run data quality checks, generate training and validation samples, transform raw data into features, perform hyperparameter tuning, train the candidate model, evaluate its performance, and generate any number of evaluation metrics and charts associated with each stage.  When you’re in the fine-tuning stage of the project, any small error or stakeholder suggestion can require a full re-run.  When the process is manual, each iteration can cost hours or days of your best Data Scientists’ time (read $100s or $1000s), whereas with DAGs they can make the change and let the computers orchestrate the process.  Expensive and slow iterations inevitably pushes the team to shortchange QA.
* **Long waits to test promising ideas in-market**: if the Data Science team’s development framework doesn’t integrate with the engineering team’s, then their prototypes need to be re-implemented from scratch before they can be tested in-market.  When you raise the costs of testing new ideas substantially, you decrease the number of ideas you can test, and lower the probability of hitting on a winner.
* **"Shadow Engineering Environments”** it’s common for these teams to have a number of critical dashboards, tables, or reports that need to be updated regularly, but have never been prioritized and implemented by the official engineering teams.  These data update processes can become critical to business operations without ever going through the formal governance, QA, and business continuity controls that are typically baked into an engineering team’s requirements.

The benefits of adopting DAGs on the Data Science and Data Analyst workflows are the opposite: faster iterations,
shorter time-to-market, and transparency across all critical data processes.  But how do you overcome the software
engineering knowledge gap that has prevented use of these tools in the past?

### It's not as hard as it used to be

Luckily, with the growth of [ML-Ops](https://en.wikipedia.org/wiki/MLOps) as a separate discipline over the last few years, an increasing number of
DAG tools aimed squarely at the Data Scientist and Analyst population have been developed.  With a relatively
small investment, you can create template projects that give your analysts a concrete example of how to work
within a DAG framework.  Because DAGs are very intuitive, they won’t need much training to continue working this
way once they’ve been shown how.  As an example, my [python cookiecutter](https://github.com/ahasha/cookiecutter-pypackage) is a ready-to-use python project template
that incorporates a lightweight DAG and data-versioning tool (DVC).

If you’re looking for a DAG tool that is accessible to analysts, I’d recommend taking a look at these three:

* [DVC](https://dvc.org/doc) is a lightweight but powerful tool that enables Data Scientists to build DAG workflows and transparently track their data versions without any hosted infrastructure.  It’s a strong option for Data Science teams with limited engineering or infrastructure support.  My [Milton Maps](https://miltonmaps.hashadatascience.com/) project gives an example of how it can be integrated into a data science project.
* [Apache Airflow](https://airflow.apache.org/docs/apache-airflow/stable/) is an open-source platform for developing, scheduling, and monitoring batch-oriented workflows that is popular with many Data Science and Data Engineering teams. Airflow’s extensible Python framework enables you to build workflows connecting with virtually any technology. A web interface helps manage the state of your workflows. Airflow is deployable in many ways, varying from a single process on your laptop to a distributed setup to support even the biggest workflows.
* [dbt](https://www.getdbt.com/blog/on-dags-hierarchies-and-ides/) is a SQL-first transformation workflow that lets teams quickly and collaboratively deploy analytics code following software engineering best practices like DAGs, modularity, portability, CI/CD, and auto-generated documentation.

Would you like help exploring the potential benefits of DAGs for your Data Scientist and Analyst teams?  [Get in touch](https://forms.gle/ocDywDpj3HAy87Pc8) and let’s start the discussion?

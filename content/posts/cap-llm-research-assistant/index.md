---
title: "Case Study: Using AI as a research assistant to develop Milton's Climate Action Plan"
date: 2024-07-13T20:36:51-04:00
tags: ["generative-ai", "case-study", "climate"]
draft: false
weight: 1
---

As a member of Milton's [Climate Action Planning Committee](https://www.townofmilton.org/584/Climate-Action-Planning-Committee), I'm working with a team of committed, passionate volunteers and town staff to develop the roadmap for Milton to rapidly reduce its town-wide fossil fuel pollution and prevent the most damaging impacts of climate change.

Massachusetts's climate goal to achieve net-zero emissions by 2050 is at once breathtakingly ambitious and barely adequate to prevent dire consequences for future generations. Writing a climate action plan is challenging because it necessarily involves bold, unprecedented action across every aspect of the local economy, as almost everything we do uses energy from fossil fuels in one way or another. Since their goals are unprecedented, no one has ever successfully implemented one and there is no well-blazed path to follow.  But, there are also plenty of solutions and strategies for us to consider.  At least 20 other communities in Massachusetts that we are aware of have adopted Climate Action Plans, with many more currently developing one.

Even for volunteers who are passionate about this topic, reading and digesting dozens of other towns' plans is not an easy task.  Generally, we want to focus on one topic area at a time (e.g. reducing residential building emissions).  We have questions like, "What are towns near us doing to increase the supply of renewable electricity for their residents?", but the way the plans are published makes it time-consuming to extract answers to such questions.  They are typically available as slickly formatted PDFs to be downloaded from the town website.  I was desparate for a single spreadsheet with all the towns' actions, strategies, and goals that I could filter and search quickly, but I quickly ran out of patience trying to put one together by hand.

After the release of ChatGPT last year and the rapid development of new programming frameworks to integrate them into software, I saw an opportunity to automate my way out of this difficulty. These models can extract structured data from prose documents like climate action plans, making it easier to review and evaluate them systematically.  As a Data Scientist, I was also looking for an opportunity to get my hands dirty with the latest developments in AI, and this seemed like a good use case for learning.

Long story short, this was indeed a good task for AI, and I was able to extract [a dataset of 617 goals and 2437 action items](https://docs.google.com/spreadsheets/d/1R_4A5cVGfJsiWXK0LWYIsIMSKgblCb8xY3XvXvKdU9I/edit?usp=sharing)) from 21 climate action plans published by municipalities in the Northeast.  Feel free to use it!  Compared to spending tiring hours to do this for one document by hand, the pipeline took at most a few minutes per document and had a total cost of under $20.   I had to overcome a few common technical challenges which are discussed in more detail below. The full project code is available on [GitHub](https://github.com/ahasha/llm_cap_research/blob/main/llm_cap_research/analyze.py) under an MIT license.

## Developing the Solution

My goal was to build a data pipeline that would process a collection of PDF documents into a table of goals and actions.

As I do for all my projects, I used the [reproducible data science project structure](https://hashadatascience.com/reproducible_data_science/) I've written about [elsewhere](https://medium.com/mission-lane-tech-blog/automate-model-development-part-1-ed22b1760ca9).  I used  [LangChain](https://python.langchain.com/v0.1/docs/introduction/) to integrate LLMs into my pipeline.  It is a general-purpose programming framework that integrates with a variety of LLM model providers (e.g. OpenAI, HuggingFace) and provides a range of enhanced functionality to integrate LLM results into an application.

### The Prompt

Much of the introductory content about LLMs focuses on chatbots or Q&A workloads, where the LLM is expected to produce a brief, relevant answer to a statement from the user.  This information extraction use case is a little different, in that I wanted the LLM to exhaustively review lengthy source material and provide a comprehensive list of all the relevant details it contains (in this case, goals and actions).

I expressed my use case to the LLM with the following prompt:

```
You are an expert municipal climate action planner.

Read the provided Climate Action Plan document and extract the following information
described in the text:

1. Goals;

2. Action items to achieve the goals;

If the text does not explicitly describe a goal or action, do not make up an answer. Only
extract relevant information from the text. However, be exhaustive, and extract all goals
and actions that are explicitly mentioned in the text.

There will often be multiple goals and actions mentioned on each page.

If the value of an attribute you are asked to extract is not present in the text, return
null for the attribute's value.
```

The instructions to "extract all goals and actions that are explicitly mentioned in the text; there will often be multiple goals and actions mentioned on each page" guided the LLM to produce an exhaustive list of goals and actions in its output.  I further reinforced this through the output data structure definitions described below.

### Structured Information Extraction

Langchain's [information extraction capability](https://python.langchain.com/v0.1/docs/use_cases/extraction/) seemed perfectly suited to my task and was my key reason for selecting it.  It transforms user-defined pydantic data structures into LLM prompts that guide the model to return results in structured formats like JSON or YAML that can be analyzed systematically. This LangChain feature allows you to define detailed attributes of any entity information you want to extract and also provide semantic guidance to the LLM on what to look for.  For example, [here](https://github.com/ahasha/llm_cap_research/blob/7e90ecd12b6c0634f394823808091e78cd73cb6b/llm_cap_research/analyze.py#L93-L117) is how I directed the LLM to identify a climate action:

{{< gist ahasha f2693f20d2f0e05c14e216e21b1b6e9f actions.py >}}

LangChain uses this pydantic class to instruct the LLM which attributes to record about each action, but it also passes your descriptive comments to the LLM as instructions about what to look for.  It felt similar to the kind of guidance you'd give to a human research assistant.  I defined a similar class for [goals](https://github.com/ahasha/llm_cap_research/blob/b4cbb5fa2fefb96f159bac9fe96ec6d94f11d9b4/llm_cap_research/analyze.py#L58-L90) emphasizing the distinction from actions: they define outcomes rather than methods (a distinction that was not always followed in our source documents!), and preferably include a quantitative metric and a timeline.

 To reinforce my prompt request for an exhaustive list of goals and actions, I define a `Result` output object that included a list of goals and a list of actions:

{{< gist ahasha f2693f20d2f0e05c14e216e21b1b6e9f results.py >}}

### Minimizing Hallucinations

With LLMs, Hallucination is a key risk.  When a model "hallucinates", it makes up false answers that are not present in the source material.  To mitigate the possibility of hallucination, I did two things.  First, my prompts explicitly included "If the text does not explicitly describe a goal or action, do not make up an answer", and "only extract relevant information from the text."

Second, via the pydantic entity definition above, I instructed the LLM to also include the verbatim quote and page number from the source document that its response is based on.  This citation allows users to verify the information and ensures transparency. In the final output, each goal and action item is linked to a document URL, making it easy for human researchers to quickly look up the source material for follow up if desired.

### Managing Cost

Since I had not built an application using commercial LLM APIs before and this was a self-funded learning project, I was concerned about the costs I might incur simply by experimenting.  To begin development with the OpenAI APIs, you create an account and give them your credit card information to receive an API key.  API calls are charged by "token" of input and output (roughly per word), so it isn't trivial to predict up front what a particular workload will cost.  I needed to monitor my token usage to ensure the project costs would be acceptable.  I decided I wanted to integrate right from the start with a telemetry solution to monitor token usage and costs during development. This also turned out to be valuable for diagnosing issues with the output token limits discussed below.

 A lot of new startups are offering solutions in this category, but I found them to be fairly immature given how new and fast moving this space is.  I tried several tracing tools and found that their tutorials didn't work with the version of LangChain I was using.  Ultimately, I chose to use [LangSmith](https://smith.langchain.com/) because it is developed by the same company as LangChain and, not surprisingly, worked seamlessly with it.  All I had to do was add `langsmith` to my project environment and add [a single line of code](https://github.com/ahasha/llm_cap_research/blob/7e90ecd12b6c0634f394823808091e78cd73cb6b/llm_cap_research/analyze.py#L26-L27) to get a usage dashboard with visibility into my cost per API call and total costs for the project.

{{< figure src="langsmith_screencap.png" title="Langsmith Screen Capture" >}}

Another thing I did to manage cost was to use an inexpensive, less capable model (`gpt-3.5turbo`) for testing until I was certain I had a functional pipeline.  Only when I knew I had everything hooked up correctly did I switch to a more capable model (`gpt-4o`) and focus on tuning my prompt and parameters to get meaingful results.

### Loading the PDF files

Another important consideration was the optimal amount of content to send to the LLM in each API call. The context window of an LLM model is the amount of text the model can consider at one time when generating a response.  The context window of cutting edge LLMs are huge.  gpt-4o's input context limit is 128,000 tokens, or about the length of a 350 page novel.

However, for my use case, the LLM's output token limit was the limiting constraint, not its context window.  The output token limit is the maximum length of the model's response.  GPT-4o has an output token limit of 4089, far smaller than its input context window.  Because I was asking for an exhaustive list of goals and actions described in the text, the ratio of input to output length varied widely depending on the content.  Sometimes the plan authors spend pages describing their town, their planning process, and thanking all the people who helped without mentioning a single goal or action item.  Other pages consisted of bulleted lists of goals and actions in small type.  Because I'd asked for the LLM to structure its output in a  machine-readable format (JSON) and include verbatim quotes, such pages could easily result in outputs double the length of the provided input.  I quickly discovered that if the model's response exceeds the 4096 token limit, it will simply stop when it hits the token limit and return malformed JSON, resulting in an error and halting my processing pipeline.

On the other hand, I found that feeding short segments of the document resulted in unacceptable level of duplication of actions and goals that were mentioned multiple times in the document, and the model would miss items that were truncated by a segment boundary.

This issue ultimately determined the chunk size I used. Submitting one page at a time led to excessive duplication of action items mentioned on multiple pages. My goal was to find a trade-off that kept queries and responses within the model's context window while minimizing duplication.

#### Managing output length while chunking PDFs

LangChain offers a wide selection of [Document Loaders](https://python.langchain.com/v0.1/docs/modules/data_connection/document_loaders/) for integrating data in all kinds of formats.  In particular, they offer multiple PDF parsers with varying capabilities to handle images and other difficult formatting.  Understanding images was not central to my use ase, so I selected `PyMuPDFLoader` for its speed and limited dependencies.

By default, `PyMuPDFLoader` chunks a PDF into documents one page at a time, but the amount of information on a page varied wildly depending on formatting, font sizes, etc.  Usually, a page was too short, leading to excessive duplication as described above. To deal with this, I implemented a [PDF document generator](https://github.com/ahasha/llm_cap_research/blob/7e90ecd12b6c0634f394823808091e78cd73cb6b/llm_cap_research/analyze.py#L161-L212) that processed the document page by page, building up chunks until they reached a desired token count. I used `tiktoken`, a library provided by OpenAI, to calculate the number of tokens in each block of text. This approach allowed me to experiment with different maximum chunk sizes to balance staying within the context window limits and reducing duplication in the output.

However, for some worst-case pages with high density of goals and actions, even a very short document chunk could produce output that exceeded the model's output token limit.  To avoid having to constrain my typical input chunk size to these worst-case scenarios, I implemented [a back-off mechanism](https://github.com/ahasha/llm_cap_research/blob/b4cbb5fa2fefb96f159bac9fe96ec6d94f11d9b4/llm_cap_research/analyze.py#L290-L305) where API calls that resulted in an output parsing failure were divided in half recursively and resubmitted until a successful response was received.  Since I had to pay for an API call whether the output could be parsed successfully or not, I wanted to use an input chunk size that would succeed the majority of the time, while using the back off mechanism to avoid having the pipeline fail on those worst-case pages.  Ultimately, I used an input chunk length equal to `gpt-4o`'s output token limit (4096) with good results.

## Conclusion

Overall, I was pleased with the results of this exercise.  I was impressed by how capable the model was at identifying the kind of information I was looking for and how little it cost to process my entire collection of documents.

This approach could be easily modified for similar use cases where you want to extract exhaustive lists of relevant items from lengthy source material.  It's a good use case for LLMs because it frees humans from a tedious and mind-numbing task: having to reach everything and assemble a complete list before you can focus in on the specific items that interest you that may be spread across multiple documents.  But, the results will still be subject to human review before we act on them, limiting the risk from hallucinations.

If you'd like some help with a similar use case, don't hesitate to reach out!

<br>
<button class="contact-button" role="button" onclick="window.location.href='https://forms.gle/ocDywDpj3HAy87Pc8';">Schedule a consultation!</button>
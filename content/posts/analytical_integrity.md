---
title: "Operationalizing Analytical Integrity"
date: 2023-01-28T16:26:01-05:00
draft: false
toc:
tags: ["analytical-integrity"]
---

## If you're in the data business, analytical integrity is critical

Selling data and analytics is a bit like practicing medicine.  Your product is complex and error-prone, and if it’s not 
delivered with quality and accuracy you can do more harm than good.  But your customer, while desperate to get a quality 
product from you, is in no position to judge the quality of your work directly.  So instead they judge you on shallow 
proxies that are easier to see: how fast they can get an appointment, how professional your office decor looks, your 
bedside manner, etc.  For a Data Scientist, those proxies might be attractive website design, data size, what 
university you graduated from, or the number of equations in your pitch deck.

Any company that sells products based on data and analytics must grapple with the reality that accuracy and transparency
are in tension with speed to market, and customers are unlikely to reward you for making the extra investment required 
to deliver true quality as opposed to those shallow proxies, at least in the short run.  However, if you fail to invest
in the analytical integrity of your product, your customers will inevitably find out.  Your reputation may be 
irretrievably damaged when they do.

## Hiring smart people isn't enough

Typically, the first thing business leaders do when they want to build a great data product is hire exceptional talent. 
If your team has the right experience, and the right credentials, the thinking goes, they can hardly fail to produce a 
great product.  

Certainly, a talented team is a necessary ingredient for a great product.  The experience and integrity of
the individuals on your team can carry you a certain distance.  Individual expertise alone can be sufficient when
the team is small, the data experts have full visibility to how their work creates value end to end, and they are 
empowered to meet their own quality standards.

A smart and capable team isn't sufficient to avoid significant errors if any of these conditions aren't met. 
That is:

* When the end-to-end value chain of the analytics is too complex or varied for an individual to appreciate
* When the data team becomes too large to avoid having a range of skill and experience
* When the data team is under pressure to deliver shallow proxies of quality, but not held accountable for true quality
* When the timescale for getting feedback on true quality is longer than a performance management cycle.

The financial crisis of 2008 is a classic global-scale example of how this dynamic can play out. 
Everyone knows that the financial crisis was triggered by the collapse of the mortgage market, but 
in a very real sense it was also a story about failures of analytical integrity in the banking industry.

### Analytical integrity and the financial crisis

Housing prices had fallen before, but they triggered a financial collapse in 2008 because falling 
prices drove losses at levels that banking executives had thought would be impossible.  Why 
did they think such losses were impossible?  Because they were relying on complex pricing and risk models
that estimated the probability of large losses, and those models said the probability 
was extremely low.  These models enabled the creation and marketing of ultra-low-risk bonds (so-called
triple-A rated bonds) backed by high-risk mortgage loans.  The triple-A rating on these bonds enabled
banks to buy them with money they were not allowed to put at risk, but earn returns typically associated with
riskier assets.  

[James G. Rickard](https://en.wikipedia.org/wiki/James_Rickards) made this point forcefully in his [testimony before the House Committee on 
Science and Technology](https://www.govinfo.gov/content/pkg/CHRG-111hhrg51925/pdf/CHRG-111hhrg51925.pdf) in September 2009: 
> The world is now two years into the worst financial crisis since the Great Depression. 
The IMF has estimated that the total lost wealth in this crisis so far exceeds
$60 Trillion dollars, more than the cost of all of the wars of the 20th century combined. 
The list of causes and culprits is long including mortgage brokers making
loans borrowers could not afford, investment bankers selling securities while anticipating 
their default, rating agencies granting triple-A ratings to bonds which soon
suffered catastrophic losses, managers and traders focused on short-term profits and
bonuses at the expense of their institutions, regulators acting complacently in the
face of growing leverage and imprudence and consumers spending and borrowing at
non-sustainable rates based on a housing bubble which was certain to burst at some
point. This story, sadly, is by now well known.

> What is less well-known is that behind all of these phenomena were quantitative
risk management models which told interested parties that all was well even as the
bus was driving over a cliff. Mortgage brokers could not have made unscrupulous
loans unless Wall Street was willing to buy them. Wall Street would not have
bought the loans unless they could package them into securities which their risk
models told them had a low risk of loss. Investors would not have bought the securities 
> unless they had triple-A ratings. The rating agencies would not have given
those ratings unless their models told them the securities were almost certain to
perform as expected. Transaction volumes would not have reached the levels they
did without leverage in financial institutions. Regulators would not have approved
that leverage unless they had confidence in the risk models being used by the regulated entities. 
> In short, the entire financial edifice, from borrower to broker to banker to investor to rating 
> agency to regulator, was supported by a belief in the power 
> and accuracy of quantitative financial risk models. Therefore an investigation into
the origins, accuracy and performance of those models is not ancillary to the financial crisis; 
> it is not a footnote; it is the heart of the matter. Nothing is more important to our 
> understanding of this crisis and nothing is more important to the task
of avoiding a recurrence of the crisis we are still living through.

I was a summer intern for the Mortgage Backed Securities modeling team at Citigroup in 2006 and 2007, and
the problem was not a shortage of stellar minds working on the problem.  The analytics leaders I worked under were 
brilliant, and they understood the shortcomings of their models very well.  There was a weekly lunch seminar where 
summer interns were given an introductory tutorial on the different markets and asset types from an experienced trader.  
I vividly remember the Collatoralized Mortgage Obligation (CMO) trader saying (in 2006), "This market is headed for a 
train wreck eventually.  But there's nothing we can do about it but keep meeting the market's demands while it lasts."

I don't think anyone on that team foresaw the scale of damage the market's collapse would cause, but they 
knew it was unsustainable.  The problem wasn't a lack of analytical ability, it was a broken system of incentives.  
If the paycheck of everyone involved depends on analytics giving the answer people want, and not the truth, then
individuals feel like their only choice is to ride the gravy train until it crashes, or take a principled
stand and leave.  And you have to keep in mind, it's rarely obvious when analytics are flawed, or to what extent.
Just as the best writers need editors to help them perfect and refine their work, it takes time, energy, and a critical frame
of mind to identify errors in an analysis.  If analysts aren't rewarded or held accountable for this effort, the path of 
least resistance is to skip it.  If the analytics are complex enough, doubt about the scope of the flaws and lack
of investment to understand them deeply provides plenty of moral cover to stick around until the gravy train crashes.

## Operationalizing analytical integrity

The government's response to the role of flawed analytics in the financial crisis was surprisingly
balanced and effective. In 2011, the Federal Reserve and the OCC release [regulatory guidance on model risk
management](https://www.occ.treas.gov/news-issuances/bulletins/2011/bulletin-2011-12a.pdf), and 
required banks to set up systems of oversight to prevent the kind of systematic failures that 
had put the banking system at risk.  As AI and Machine Learning became increasingly popular over the next decade,
they were included in this regulatory framework.  At Capital One, I led the team responsible for model risk 
governance of innovative uses of Machine Learning outside of core lending activities, such as chatbots and cybersecurity. 
I learned a lot about how to tackle this problem pragmatically.

Investing in analytical integrity isn't a binary choice between perfection and garbage.
Perfectionism leads to analysis paralysis.  Data and models are never perfect. At the other extreme, tunnel vision on 
speed to market or conforming to customer expectations leads to flawed analytics that, at best, erode 
customer trust.  Beyond the startup stage, leaders need to build a management and incentive framework that 
allows their team to strike an effective balance between these two extremes, and that converts analytical integrity
from a personal virtue of individual team members to a necessary consequence of how the team operates.

The exact framework for this depends on what your company does, but here are six key principles to think about
as you set out to operationalize analytical integrity:

* **Effective challenge through peer review**: Peer review is the gold standard process for ensuring the integrity of complex
    analysis.  To be effective, it must be performed by individuals with appropriate subject-matter expertise who are
    incentivized to identify material flaws of the analysis.
* **Define a policy framework for required validation activities**: under delivery pressure, the temptation to skimp on 
  testing is powerful.  Detail-oriented analysts will often want to spend more time checking their work than can truly 
  be justified from a business perspective.  A clear policy framework removes the need for constant negotiation and 
  conflict between these two points of view, and is needed to drive consistency and predictability.
* **Know whether your analytics have short or long feedback loops**: the quality of certain types of analytics can
  be judged at a glance.  Everyone knew ChatGPT was a game changer the moment it was released.  Other kinds, like 
  bank lending models, can take years (and multiple compensation cycles) before their true performance is known. Analytics
  with long feedback loops need a larger investment in analytical integrity.
* **Adjust the scope and timing of validation activities based on risk and complexity**: A simple calculation driving a 
  low stakes business decision may need only minimal review prior to release, while a complex, heavily-used analytical 
  feature could cause significant reputational harm and operational cost if an error is discovered by a customer.  
  Validation policies should define metrics and thresholds to distinguish these scenarios, and scale the scope and 
  timing of validation activities accordingly.
* **Senior leader incentives combine delivery and quality metrics**: It is rare for senior business leaders to have the 
  skill set or time necessary to evaluate the quality of analytics that drive their business strategies.  Particularly 
  when slow feedback loops delay the consequences of flawed analysis well beyond the rewards of successful delivery, 
  leaders who are judged solely on their ability to deliver face irresistible pressure to deprioritize non-obvious
  quality issues.  Leadership performance must be evaluated both on what and how they deliver, or the validation process 
  will not be effective or credible.
* **Use automation and standardization to make integrity the "happy path"**: There are many difficult tasks involved on
  the road to analytical integrity.  You need to be transparent about what data was used, and how it was transformed. It
  needs to be made accessible to others to enable peer review.  Countless small validation criteria need to be checked
  every time the analysis is altered.  Starting from scratch and working manually, it is nearly impossible to do all these
  things.  Luckily, there has been a revolution in the automation of software testing in recent decades, and data 
  scientists have developed patterns and standards for applying these tools to analytical work.  Learning these tools,
  and baking them into organizational standards (e.g., through a project template) can make working in a transparent
  and reproducible manner the easiest way forward for your team.

If this sounds like something your organization needs to work on, maybe I can help! Please don't hesitate to 
[get in touch](/contact/).



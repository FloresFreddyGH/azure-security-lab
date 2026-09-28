# Lab Journal

## September 25, 2026 — Getting started

I verified my Azure student offer, which showed $100 in available
credit. I also created the GitHub repository and cloned it onto
my Ubuntu computer.

I'm starting with the documentation so I can keep track of what
I build and explain my decisions later.

### What I completed
- Verified my Azure student offer.
- Created the project repository on GitHub.
- Cloned the repository onto Ubuntu.
- Started this journal.

### Next
Review Azure costs and plan the lab resources before deploying them.

### GitHub setup troubleshooting

My first commit failed because I hadn't configured my Git author
name and email. After setting those, I could commit locally.

I then signed in through GitHub CLI to push from Ubuntu. GitHub
blocked the push because the commit contained my private email
address. I switched to my GitHub no-reply email and amended the
commit. The next push worked.

I learned that making a commit saves changes locally, while pushing
uploads them to GitHub. I also learned that changing Git's email
setting doesn't update commits I've already made.
little advancement... (it took like 10 minutes dont know why exactly)
The workspace was successfully created in Sweden Central.
No log sources are connected yet.


### Finding an allowed deployment region

The East US workspace setup was blocked by an Azure policy.
I checked the policy parameters and found that my subscription
allows Sweden Central, Belgium Central, Denmark East,
Germany West Central, and France Central.

I chose Sweden Central for the workspace after checking that
it supports Azure Monitor Logs and Microsoft Sentinel.
The resource group remains in East US because its location
doesn't have to match the resources inside it.


### Creating the lab resource group

I created rg-azure-security-lab in East US to keep the lab
resources together. I added Project and Environment tags
so its purpose is easy to identify.

The group is empty for now. Its region specifies where Azure
stores the group's metadata; it doesn't force every resource
I add later to use that same region.

### Region restriction during workspace setup

I tried creating the Log Analytics workspace in East US, but it was basically 
blocked for Azure Students subscriptions which i didnt know before, after ten 
minutes finally found why i couldnt access.

At least this helped me understand the difference between a subscription
policy restriction and a problem with the resource configuration.


### Reviewing workspace pricing

I checked the workspace's Usage and estimated costs page.
It's using Pay-as-you-go, and the displayed Analytics Logs
ingestion rate is $2.99 per GB.

The page notes that Microsoft Sentinel costs aren't included
in these estimates. I'll account for that before enabling it.

### Checking log retention

I checked the workspace's default retention and left it at
30 days. The portal says 31 days are included in the current
pricing plan.

Individual tables can have different retention settings, so
I'll check those when I start collecting logs. I'll also save
the evidence used in my reports rather than relying on logs
being available for the whole semester.

### September 27, 2026 — Enabling Microsoft Sentinel

I enabled Microsoft Sentinel on law-azure-security-lab.
Azure confirmed that it was added successfully.

The trial runs from September 27 through October 28, 2026,
at 23:59:59 UTC. The activation banner says it covers up to
10 GB per day for Sentinel and Log Analytics, with additional
data billed beyond that allowance.

Sentinel is enabled, but I haven't connected a log source or
created detection rules yet. My next step is to collect Azure
Activity logs so I can investigate changes in the subscription.

### Installing the Azure Activity solution

I installed the Azure Activity solution from Sentinel's Content
hub. It provides a data connector and templates for working
with Azure subscription activity.

Installing the solution and connecting the logs are separate
steps. I still need to configure the connector and verify
that events reach my workspace.

### Learning KQL with AI assistance

I used ChatGPT/Codex to help write my first KQL query and
explain what each part does. I'm still learning KQL, so this
gave me a starting point to work through.

I ran the query in my own Azure workspace and checked the
results. It returned three Activity log records, confirming
that logs had reached the workspace.

### September 28, 2026 — Querying and investigating Activity logs

I confirmed that Azure Activity logs reached my workspace by
running a KQL query that returned three records.

I followed their shared correlation ID to inspect a
diagnostic-settings write starting and succeeding, followed
by a successful deployment record. The setting involved was
subscriptionToLa at the subscription level.

The successful write showed Log Analytics Contributor in its
authorization evidence. Its identity claims identified an
application identity and pointed to a policy assignment's
managed identity. This fits the logging setup I configured,
but I still need to compare the assignment ID with my policy.

I used Codex to help write and explain the queries.
I ran them myself and inspected the returned events while
learning how filtering, sorting, and selecting fields work.

Next: verify the exact policy identity and generate a
controlled test event to investigate.

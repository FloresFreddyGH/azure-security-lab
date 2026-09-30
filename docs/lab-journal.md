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

### September 28, 2026 — Verifying the caller identity

I compared the policy's Principal ID with the caller in the
successful diagnostic-settings event. They matched exactly.
The policy assignment path also matched the event's xms_mirid
claim.

This confirmed that my logging policy's managed identity
performed the change to subscriptionToLa.

### Tracking a controlled tag change

I added LabTest=activity-tracking-01 to my lab resource group
at about 23:27 New York time on September 28.

The portal kept loading, but after refreshing I confirmed
the tag was saved. I then found its Start and Success records
in Log Analytics at 03:27:48 UTC on September 29.

The caller matched my account, and the records shared a
correlation ID. This verified that I could make a known change
and find the corresponding events in my monitoring workspace.

### September 28, 2026 — Testing changes and creating a detection

I confirmed that the logging policy's Principal ID matched
the caller in the diagnostic-settings event. Its assignment
path also matched the event's identity claims.

I added LabTest=activity-tracking-01 to my resource group.
I found its Start and Success records in Log Analytics,
with my account as the caller. I then narrowed the KQL query
to successful tag writes.

I created an informational Sentinel rule in the Defender
portal using this query. It runs every 5 minutes and checks
the previous 15 minutes. It creates an alert when at least
one event matches and groups alerts from this rule into
incidents within a one-hour window.

After confirming the rule was enabled, I changed the tag
to activity-tracking-02 at approximately 23:53 New York time
(03:53 UTC on September 29).

I stopped before verifying whether an alert or incident was
created. That check is my next step. This is a practice rule;
a tag change alone does not indicate malicious activity.

### September 30, 2026 — Investigating and resolving the Sentinel incident

The scheduled Sentinel rule generated an incident after my
controlled LabTest tag change. The incident contained three
alerts from the same rule because the rule ran every five minutes
and looked back over the previous fifteen minutes.

I opened the alert and verified the source event. It showed a
successful MICROSOFT.RESOURCES/TAGS/WRITE operation on
rg-azure-security-lab. The caller was my own account, and the
event time matched the controlled test.

The alert query returned one matching event in Advanced hunting.
I classified the incident as a false alert caused by expected lab
activity and resolved it. No malicious activity was identified.

This completed an end-to-end test of log collection, KQL filtering,
scheduled detection, alert generation, incident investigation,
classification, and resolution.


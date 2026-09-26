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


### Checking log retention

I checked the workspace's default retention and left it at
30 days. The portal says 31 days are included in the current
pricing plan.

Individual tables can have different retention settings, so
I'll check those when I start collecting logs. I'll also save
the evidence used in my reports rather than relying on logs
being available for the whole semester.

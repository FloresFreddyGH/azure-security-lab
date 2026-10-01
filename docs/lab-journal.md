# Lab Journal

I recently passed AZ-900 and wanted to try building something in Azure myself. This is my first cloud security lab, so I'm learning as I go. Here's how it went, including the little things I had to figure out along the way.

## September 25, 2026 — Getting started

First things first: I checked my Azure student offer and confirmed the $100 credit. I created this repository and cloned it onto my Ubuntu computer. I wanted somewhere to keep my notes and screenshots before getting too far into the lab.

### Getting GitHub working

Even the first push gave me something to learn. My first commit failed because I hadn't configured my Git author name and email. Once I fixed that, the commit worked locally, but GitHub blocked the push because it contained my private email address.

I signed in with GitHub CLI, switched to my GitHub no-reply email, and amended the commit. Then the push worked. Small lesson already: committing saves changes locally, pushing sends them to GitHub, and changing my Git email doesn't automatically fix an old commit.

### Setting up the resource group and workspace

I created `rg-azure-security-lab` in East US and added Project and Environment tags so I could tell what it was for.

The Log Analytics workspace took a little more figuring out. I tried East US, but a policy on my subscription blocked that region. It took me about ten minutes to work out what was going on. The policy listed Sweden Central, Belgium Central, Denmark East, Germany West Central, and France Central as allowed regions.

I went with Sweden Central after checking support for Azure Monitor Logs and Sentinel. The workspace was created successfully there. The resource group stayed in East US; I learned that its location is for the group's metadata and doesn't mean every resource inside it has to be in the same region.

At this point I had a workspace, but no connected logs yet.

### Checking costs before going further

I checked Usage and estimated costs. The workspace showed Pay-as-you-go and an Analytics Logs ingestion rate of $2.99 per GB at the time. Sentinel costs weren't included in that estimate, so that was another thing to keep track of.

I left the default retention at 30 days. The portal showed 31 days included in the pricing plan, but individual tables can have their own settings. I'm keeping evidence in this repo too, so I don't have to rely on the logs staying available all semester.

## September 27, 2026 — Turning on Sentinel

I enabled Microsoft Sentinel on `law-azure-security-lab`, and Azure confirmed it was added successfully. One more part of the lab in place!

The activation banner showed a trial from September 27 through October 28, 2026, at 23:59:59 UTC. It listed up to 10 GB per day for Sentinel and Log Analytics, with additional data billed beyond that allowance. Those were the details shown during my setup.

### Getting some logs to work with

I installed the Azure Activity solution from the Content hub. One thing I learned here: installing the solution and actually connecting the logs are separate steps. I still had to configure the connector and check that records arrived.

I used ChatGPT/Codex to help with my first KQL query and explain the parts I didn't understand yet. After working through the logging setup, I ran the query in my workspace and got three Activity log records back. It was nice to finally have actual results to look through.

## September 28, 2026 — Following the events

With those records available, I started looking at what they meant. I followed their shared correlation ID and found a diagnostic-settings write that started and succeeded, followed by a successful deployment record. The setting was `subscriptionToLa` at the subscription level.

The successful write showed Log Analytics Contributor in the authorization details. Its identity claims pointed to an application identity and a policy assignment's managed identity. That seemed to fit my logging setup, but I wanted to compare the actual IDs before calling it confirmed.

I checked the policy's Principal ID against the event's caller, and they matched. The policy assignment path also matched the `xms_mirid` claim. That connected the change back to the managed identity used by my logging policy.

Codex helped me write and understand the queries while I practiced filtering, sorting, and choosing fields to display. I ran them in Azure and checked the records myself.

### Making a change I could track

I added `LabTest=activity-tracking-01` to the resource group at about 23:27 New York time. The portal kept loading, but refreshing showed that the tag had saved.

Then I found the Start and Success records in Log Analytics at 03:27:48 UTC on September 29. My account was the caller, and the records shared a correlation ID. This was a useful little test: make a known change, then find its trail in the logs.

### Building my first detection

I narrowed the query to successful tag writes and created an informational Sentinel rule in the Defender portal. It runs every five minutes, looks back fifteen minutes, and alerts when it finds more than zero matching records. Alerts from this rule can be grouped into an incident within a one-hour window.

Once the rule was enabled, I changed the tag to `activity-tracking-02` at approximately 23:53 New York time, or 03:53 UTC on September 29.

I stopped there for the session. Next up was checking whether the rule had created an alert and incident. The tag change was just a safe way to test the workflow; it wasn't malicious activity.

## September 30, 2026 — My first incident investigation

The rule had generated an incident from my controlled tag change. It contained three alerts from the same rule. Since the rule ran every five minutes with a fifteen-minute lookback, the same event could show up in more than one run.

I opened the alert and checked the source record: a successful `MICROSOFT.RESOURCES/TAGS/WRITE` operation on `rg-azure-security-lab`. The caller was my account, and the time matched my test. The alert query also returned one matching event in Advanced hunting.

I resolved the incident as a false alert caused by expected lab activity. That was the classification I used for this first incident; no malicious activity was identified.

At this point I had followed a change from the Azure logs to an alert, an incident, and a resolution. Pretty cool to see those pieces working together. I still wanted to understand the repeated alerts, though.

### Working on the repeated alerts

I compared the three source records. Same timestamp, correlation ID, caller, operation, and resource. They referred to the same event.

I added `ingestion_time() > ago(5m)` to the rule. I also mapped `CallerIpAddress` as an IP entity and `_ResourceId` as an Azure resource entity, keeping the five-minute schedule and fifteen-minute lookback.

For the next test, I used `LabTest=activity-tracking-03`. The successful write happened at 21:31:33 EDT, but it didn't reach Log Analytics until about 21:41:01. That's roughly nine minutes and twenty-eight seconds of ingestion delay. The empty query results earlier in the test made more sense once I compared those times.

Incident 2 was created at 21:45:57 EDT. This time the graph showed the source IP and resource group, which made the entity mapping easier to understand. I checked the source event and saved an investigation comment on the alert.

At first there was one alert. By 21:57 there were two, both for the same test event. So I couldn't call the repeated-alert issue fixed yet.

### Looking closer at the time window

The two alert queries had `query_now` values of 01:40:39 and 01:45:39 UTC on October 1. Both used the ingestion-time lower bound, but neither had an upper bound. The event's ingestion timestamp was later than the earlier query's reference time.

With Codex's help, I added `ingestion_time() <= now()` as well, so the filter had both a beginning and an end. Then I prepared another tag change to test it.

I also caught a small mix-up: the correlation ID I initially copied for the fourth test belonged to `activity-tracking-03`. I needed the new test's own ID to follow the right event. The next entry records that check and the result.

### Fourth test and wrapping up the incident

I changed the tag to `LabTest=activity-tracking-04`. The successful write happened at 22:05:28 EDT and reached Log Analytics at 22:13:46, about eight minutes and eighteen seconds later.

This test's correlation ID was `932c7419-ac40-456e-8797-0de4173d01fd`.

I checked the alert query and confirmed it used both `ingestion_time() > ago(5m)` and `ingestion_time() <= now()`. Its query reference time was October 1 at 02:15:04 UTC.

At 22:28 EDT, Incident 2 had three alerts total: two from the earlier test and one from this fourth test. I observed just one alert for the fourth test during that period. That was a good result for this test, although it doesn't prove duplicates can never happen.

After checking the source records and the mapped IP and resource, I resolved Incident 2 at about 22:34 EDT. The portal showed **Benign Positive**, which described this intentional lab activity.

### What I'm taking away from this lab

Overall, this was a fun way to get more comfortable with Azure after AZ-900. The setup, queries, and screenshots gave me something concrete to work through, and the troubleshooting ended up being a big part of the learning.

The timestamps were probably the biggest lesson: event time, ingestion time, query reference time, and incident creation time tell different parts of the story. I also learned that putting alerts into one incident doesn't get rid of duplicate alerts.

There's still more I could test. Events that arrive outside the fifteen-minute lookback can be missed, and I haven't tested scheduler failures, retries, or high volumes. For this lab, I kept the scope to successful tag writes in my resource group.

ChatGPT/Codex helped me understand the queries, troubleshoot, and organize these notes. I made the Azure changes, ran the tests, and checked the evidence myself. I'm still learning, but now I have a whole monitoring and investigation workflow that I've actually worked through.

---
layout: page
title: Ordering matters - the case of the slow RDS snapshots
permalink: /post/ordering-matters-the-case-of-the-slow-rds-snapshots/
excerpt: A story of trying to perform maintenance against a fleet of RDS instances, and meeting some unexpected friction due to doing things in the wrong order.
---

## Introduction
At Gearset, we have a modest fleet of PostgreSQL ("Postgres") database instances running in Amazon Web Services's (AWS's) Relational Database Service (RDS) offering. These database instances hold data for the variety of services that comprise our product and internal systems and they run across several geographic regions, in our production, staging, and other environments.

We run all of our databases in a configuration known as multi-AZ, where AZ stands for "Availability Zone". Essentially, a database instance actually has two instances behind the scenes with replication between them that allows us to fail over from one to the other manually or automatically in an attempt to increase availability during upgrades or hardware failure, etc.

We regularly have to perform maintenance on these database instances to install operating system updates, upgrade the database engine, or change certain aspects of their configuration which cannot be done without a reboot.

Some parts of Gearset can have their databases upgraded during working hours, either because they back internal-facing systems (where downtime is less of a concern), or because they back services which, despite being customer-facing, are resilient to storage-layer outages.

Naturally, we began this maintenance cycle by upgrading databases backing the less finicky parts of Gearset.

It's a good job we did, because it has turned up an interesting problem in our approach to maintenance this time around, which, now understood, should save us a great deal of time when we move on to the more critical components - the ones where we have to do it out of hours in order to reduce the impact on our customers.

This isn't the first time we've performed maintenance on our RDS Postgres instances where it required downtime. We've been through several minor and major upgrade cycles, we've installed OS patches, we've seen it all before - or so we thought. Outside of those maintenance windows, due to the less resilient parts of the platform, we opt out of anything that could trigger otherwise-avoidable reboots or failovers of some of our database instances.

This maintenance window, we're doing something we've never done before; we're both installing OS patches *and* doing a minor engine upgrade.

## The plan
The plan that we've been following this time around was as follows:
 - take a manual snapshot of the target instance (with the understanding that this would expedite the automatic snapshot(s) that we knew would later occur as part of the upgrade process, since RDS snapshots are essentially incremental)
 - kick off the OS patching process and wait until that was done
 - kick off the minor version upgrade from one version of Postgres to another

If at this point you're screaming at this blog post about what's going to go wrong, [we're hiring](https://gearset.com/careers).

## The outcome
For one of a our medium-sized instances, with only a few terrabytes of allocated storage, the initial manual snapshot took 15 minutes. This was a little longer than we expected - the automated snapshot for that instance only took a couple of minutes earlier that morning, and it had received far less traffic than unusual in the meantime - but we didn't think this was a problem because we didn't have to take the internal applications that relied on this database instance offline yet.

Once the manual snapshot was finished, we took the dependent applications offline and applied the OS patches. This only took a few minutes.

So far, so good.

Then we requested the minor database engine version upgrade.

A minor version upgrade involves automatic snapshots before and after the actual engine upgrade - we knew this, it was the reason we'd taken manual snapshots earlier - so that it would be quick.

The pre-upgrade snapshot took 4 hours.

Whilst this was an internal-facing application, its availablility was still fairly important to our sales team. The "one hour" of expected downtime that we promised them stretched into nearly five hours of unexpected downtime, and impacted their ability to do their job.

Oops.

## Let's do it again
We chalked this up to an idiosyncracy of RDS - it's a managed service, you don't know what quotas or burst balances might have impacted us. We promised that we wouldn't take down that application for that long again in the future, and moved on after a fruitless discussion about what might have gone wrong.

Then another team in Gearset followed the same process for a *much* larger database instance, backing a customer-facing service. Same approach, worse consequences. The upgrade took nearly 12 hours, most of which was once again spent taking that automatic pre-upgrade snapshot.

## What's going on?
We are a few days away from having to take Gearset offline in order to upgrade the databases which are on the critical path. Early start on a Saturday morning, usually takes a few hours under the best of circumstances, and we've got the snapshot demon taunting us. At this point, it feels like we've accepted that we may never know the cause of The Snapshots From Hell.

I'm on a call with my teammate today, Barry Leonard. We're chatting off-hand about the incidents behind us, the risk ahead of us, and what options we have at our disposal on the day if it doesn't go our way.

And a very fortuitous conversation it was.

AZ this, snapshot that, instance here, replication there...

And then epiphany struck; a set of facts:

 - multi-AZ databases run in two AZ in a given region: a primary and a secondary which get failed over between (the primary becomes the secondary, and vice versa, after a failover)
 - we don't do multi-AZ failovers very often, if we can avoid it
 - snapshots, both manual and automatic, are taken against the secondary
 - snapshots live in a single AZ: the "speed" advantage of having a previous snapshot is reliant on topology
 - OS patches cause a failover from the primary to the secondary, but not back again afterwards
 - minor version upgrades take a snapshot of the secondary, and upgrade both the primary and secondary, then take another snapshot of the secondary, no failing over.

What had been happening was this: a given multi-AZ instance had the primary running in a certain AZ for a long time; we kicked off a snapshot for that instance which lived in the zone of the secondary instances (let's say, AZ1); we kicked off OS upgrades and caused a failover to the instance in AZ2; then when we kicked off the minor version upgrade it triggered a snapshot of the instance in AZ2, which had no recent snapshots to be "quickly incremental" against.

What makes this even more confusing is that AWS's documentation says that the snapshots happen against the secondary instance (and their support confirmed this to us whilst we were trying to understand what had happened with our first two upgrades), but the AWS Console shows the AZ for the primary and the AZ of the snapshots as being all in the primary region - I'm not sure if this is a case of poor documentation or a case of the Console trying to show a "less confusing" view.

## The solution
So what are we going to do on Saturday?

Well, now that we know that ordering matters, for each of the database instances we are performing maintenance on, after the OS patch we're going to trigger a reboot with failover before applying the minor database upgrade. This should ensure the automated snapshots happen in an AZ where they have a recent snapshot to derive from, which should hopefully result in a much quicker process and less downtime for our customers. Wish us luck! (I will update this post if it all goes terribly wrong...)

## Update (2023-03-27)
We intentionally performed a failover between the OS patches and the minor version upgrades on the big day - it went exactly as we'd hoped! Very little time spent on the automated snapshots, and they were taken in the same region that the previous several automated snapshots had happened in. This saved us a great deal of time and meant that our customers could get back to doing their critical Salesforce deployments over the weekend as they had originally planned.

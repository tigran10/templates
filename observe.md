# Observability and Monitoring: We Take It Seriously

## Table of Contents
- [We Take It Seriously](#we-take-it-seriously)
- [Where to Start?](#where-to-start)
- [Complexity](#complexity)
- [Tools vs Solutions](#tools-vs-solutions)
- [Dashboards, Instrumentation, Logging, Databases, Alerts](#dashboards-instrumentation-logging-databases-alerts)
	- [Instrumentation](#instrumentation)
		- [Instrument Logs](#instrument-logs)
		- [Instrument Metrics](#instrument-metrics)
		- [Data Format](#data-format)
	- [Logo Chaos](#logo-chaos)	
		- [Logs aren't solutions](#logos-arent-solutions)     
		- [Elasticsearch](#elasticsearch)
		- [Kibana](#kibana)
		- [Prometheus](#prometheus) 
		- [Alerts](#alerts) 
		- [Grafana](#grafana) 
		- [Open telemetry](#opentelemetry) 
	- [Why We Did What We Did](#why-we-did-what-we-did)
		- [Instrumentation](#instrumentation-1)     
		- [Logs](#logs)         
		- [Metrics](#metrics)
		- [Dashboards](#dashboards) 
		- [Alerts](#alerts-1) 
	- [The Multi-Cloud Question](#the-multi-cloud-question)
	- [Final Words](#final-words)

---

## We Take It Seriously

When we write software in our teams, we assume it will be running on production. Yes, surprise surprise, unusual assumption. So from day one, you must consider concerns about logging, monitoring, and observability.

Surprisingly, we saw this being a weak point in engineering, and we think we know why. We observed when we interviewed people, that in many organisations when they were shipping their solutions, they never saw it running, they never did troubleshooting on those systems, they never saw observability dashboards, they did not even know where to start from to do that, because for each of these things there were central teams promising it will be done for them.

Obviously if a developer is not putting the right amount of logs, or instrumentation, or engineering self-healing capabilities, those things cannot be injected from the side. It's not clear how that promise is being made — that someone else, who has zero context about your code, or access to the code, can support your code in production, hoping you have provided logging context, traceability, and observability. That's fundamentally wrong.

Hence, another topic where we are very opinionated. If you write code, make sure it works in production. And if it does not for some reason, make sure you write a test to replicate the issue, and on top of that, observe and trace down the problem.

**You write it, you own it.**

---

## Where to Start?

If you have been there and done that, you can skip this section.

---

## Complexity

When we write software, we want to make sure what we write is right. So we write tests. Then we package the software and start promoting it to production via some kind of a path to production.

The closer it is to production, the less access you have to your software, and the more interest you are developing about the usage of the software at the same time. And that’s where things are becoming tricky.

You get more questions about real-life usage when the software is running in real life, which means in a controlled environment, where suddenly you can't trace logs and do `cmd+F` to search. So you must have had all the necessary sensors, tools, engineered, so that you can operate remotely.

### Like Engineering a Space Shuttle

Almost like with engineering a space shuttle, the command center takes probably the same amount or more of engineering as the shuttle itself. But to add to the complexity, it’s different when you deploy to production because it suddenly assumes scale.

And with scale comes the underlying design — how you scale your application. Usually, you run the copy of it to help the traffic. More traffic, more copies are running. Now those copies can be orchestrated by platforms like Kubernetes, or serverless instances, or your own engineered solution.

But logic is always the same: you make your software able to run as a single instance as well as N instances. But then it means you log about the same software from multiple copies.

### The Instance Shuffle

To add more complexity, now imagine we don’t have a concept of sticky sessions, so 10 different requests from the same user now land not on the same instance of your app but on 10 different instances of the application. Suddenly all our logs and metrics are now shredded. Not only do I need to collect and order them in one place, I also need to actually query and combine relevant logs together.

If you think this was the complex example, you will be disappointed — as it's the easy part. Usually it's not just a request and response — it’s a request to some service that then connects to another service that then connects to something else. All those different services are scaled, so they all have different copies of themselves to cope with traffic.

So now, all of them are logging total randomness unless you find a way to order them in a way that tells the correct story. That was about logging. Assuming we actually write logs.

### And Metrics Too

But at the same time, we want to observe each of these "scalable" instances, understand what is being used inside and how. What can cause the problem. And ideally — predict problems. And we can call this part: metrics.

We need lots of metrics coming from everywhere, that we can then group and make dashboards, alerts that tell interesting stories. And when we get bored of dashboards, maybe we start engineering solutions that try to self-heal when they detect a potential problem.

---

## Tools vs Solutions

Now, when you know why it’s complex, let’s talk tools.

You might have heard from techie-jargon people: "Let’s use `Prometheus`", or `ELK stack`, or `Datadog`. And of course, all those tools can be useful.

However, they will **not** be useful if you think it’s a matter of using the tool. As you will get very basic dashboards and UIs, with information inside that does not make any sense.

In reality, all those tools assume extra engineering. In large corps, you will always meet lots of people talking tools as solutions. That’s exactly what we don’t want.

We need to make sure problems are articulated as engineering problems. And solutions are engineered — not installed. **BIG** difference there.

Now let’s go back to all these tools and see what they have in common.

---

## Dashboards, Instrumentation, Logging, Databases, Alerts

The usual suspects we are looking at:

- Instrumentation
- Storage for logs
- UI with query mechanism and query language
- Alert rules
- Dashboards
- Time-series database

Then you have a spectrum of tools that try to cover all aspects, or provide solutions for one or two aspects. Some tools like Datadog have done great custom engineering of everything, so you get opinionated but amazing tools to engineer your solutions. But you can also pick and choose each of those things from available open-source projects and have a blend of tools.

### What Prometheus Is and Isn’t

Prometheus is one of them. You may think it’s a dashboard — as people who talk tools-language usually refer to it as a dashboard. However, in reality, it’s a time-series database, and query language.

So you use their database and query language, and their instrumentation to collect all data that you can then query with time-series language.

Or Elasticsearch is usually a preferred open-source solution to save logs, in JSON format. But each of these tools are not solutions. They are more like tools that you need to use to engineer solutions as they are very technical.

### Instrumentation

#### Instrument Logs

Remember we talked about logging being hard, as it comes from different places, in different order. We need to start putting extra information into logs so that we can reconstruct the correct story later by grouping correct things together.

To not do that manually — as it would be insane — we use instrumentation tools. Instrumentation means: add extra code to your code, without writing the code.

One of the ways: mix in a Java agent, that will push this logic to relevant parts of your application.

For example, instrumentation can be:
- All our functions must have logging when they were started and ended.
- Try to resolve `USER_ID`, and add it to every log so that we can then group all logs coming from the same user and sort them with timestamp.
- Or even better, add also `REQUEST_ID`, to group logs by request and response.

So your logs are the number one thing to be instrumented.

#### Instrument Metrics

Second part of observability after logs: metrics.

Metrics can be from things like:
- "How many times within the last 24 hours your users logged in"
- Memory and CPU consumption of each application instance
- Number of messages inside your queues
- Rate and number of messages being consumed from those queues

So it’s a really wide spectrum, and the more sensors you put, the better data-driven decisions you can then get.

Also, you can keep people who like to look at dashboards happy — because you will have lots of data to play with.

While custom metrics like number of logged in users must be written inside your code, crosscutting things can be instrumented. So you usually get Java agents that get metrics for CPU or memory and ship them somewhere.

Or you also get libraries that connect to your cloud to observe your cloud services, topics, queues and so on. You collect and store this data somewhere like Prometheus, to query it later.

Now tools that provide solutions around metric stores usually come with instrumentation libs to help with data collection.

#### Data Format

Next question. If all those tools are providing their own instrumentation and backends to save this data, maybe we need to agree on a data format, so that we don’t sell our souls to a specific implementation.

Sounds like a good idea.

OpenTelemetry is the de facto agreed format, so you can move between solutions if needed. So guess what? OpenTelemetry also comes with its own instrumentation.

---
### Logo Chaos

It usually starts with a slide — full of logos. Prometheus. Elasticsearch. Kibana. Grafana. OpenTelemetry. All lined up neatly with arrows, looking like a solution.

#### Logos aren't solutions

Every tool on that slide comes with assumptions — about scale, uptime, deployment models, HA, observability pipelines, and operational ownership. What seems like an easy “stack” is often a set of distributed systems that need to be engineered, maintained, secured, and evolved.

Let’s look at what this really means.

#### Elasticsearch

You’re not just “using Elasticsearch.” You’re hosting and scaling a distributed log engine. Stateful storage. Index management. Query performance tuning. Backup and restore strategies. Retention policies.

You’ll need to monitor it too. And probably scale it. And maybe worry about when it becomes your biggest production bottleneck.

#### Kibana

The UI looks simple — until you realise it needs SSO, role management, and manual dashboard creation. And that someone needs to understand its query language well enough to troubleshoot real production issues.

Dashboards don’t build themselves. And they don’t stay up-to-date either.

#### Prometheus

It’s not just a “metrics database.” It’s a pull-based TSDB that needs to discover your services, scrape them, store data reliably, and expose it for queries.

Add high availability and remote write to the mix, and now you’re running something that expects engineering support. It drops data silently if overloaded. It grows rapidly. And it doesn’t work well out-of-the-box in federated setups.

#### Alerts

Once you figure out Prometheus alerting rules and build a few expressions, you’ll quickly notice: alerts don’t mean anything unless they’re routed properly. Slack, email, escalation. Who owns what. What’s noisy. What’s actionable.

This needs infrastructure around it — not just YAML.

#### Grafana

It’s flexible, powerful, and visual — but also configuration-heavy. Someone needs to hook up data sources, manage provisioning, sync state, and update dashboards as systems evolve.

Otherwise you’ll end up with beautiful dashboards that are out-of-sync or abandoned after one use.

#### OpenTelemetry

The spec is solid. The libraries are evolving. The collector is flexible. But it’s not plug-and-play. You still need to decide what to instrument, how to sample, where to send the data, and how to manage the overhead. It shifts complexity — it doesn’t remove it.

---

All of these tools are great — in the right hands, with the right investment, and the right operational thinking behind them.

But if you’re not thinking about how they behave under load, how you deploy them in non-prod environments, how you scale, recover, upgrade, isolate, and observe them — you’re just stacking infrastructure without a plan.

Logos don’t fail gracefully. Systems do.

So every time a tool gets picked, we ask:
- Who runs it?
- Who upgrades it?
- Who owns incidents tied to it?
- How is it tested in staging?
- How do we decommission it?

Because every tool has a cost — and it’s rarely just licensing. It’s people, time, and production pain.

If you don’t plan for that, the logos on your slide will eventually become the reasons in your postmortem.

---

### Why We Did What We Did

Why We Did What We Did

Now you might be wondering — what did we actually do?

Good question. We focused on what matters most. Not tools. Not slides. Just the outcomes we care about — and the engineering needed to get them.

#### Instrumentation

We chose OpenTelemetry and friends to instrument our code and logs.

Why? Because it’s:
- Open standard
- Cloud-agnostic
- Actively evolving
- Easy to plug into whatever backend we choose

This gives us flexibility. We’re not locked into one vendor, one toolchain, or one opinionated pipeline. We can emit telemetry in a standard way and route it wherever we need — now or later.

#### Logs

We instrument our logs — meaning they’re structured, correlated, and consistent.

We ship them as JSON. Portable, queryable, easy to work with.

Where do they go? GCP’s logging backend.

No servers to manage. No storage limits to juggle. No log rotation scripts. It just works — and integrates into the rest of the platform natively.

Can we route logs elsewhere? Yes. But why complicate things before we need to?

We saved ourselves a mountain of operational overhead by not self-hosting a log platform.

#### Metrics

Same philosophy here.

Telemetry is emitted. Backend takes care of ingestion, storage, retention, and querying.

GCP Metrics backend handles scale, cost control, and alerting. It’s optimized for the cloud it runs on. And we don’t need to reinvent time-series infrastructure.

So instead of worrying about Prometheus internals, we’re focused on emitting meaningful signals from our apps.

#### Dashboards

We use `GCP’s` built-in dashboards.

Are they perfect? No. Are they enough? Yes — and more.

Because:
- They work
- They’re tied to `IAM` (`RBAC` is built-in)
- They support Terraform (dashboards-as-code)
- They don’t require a separate platform to host and maintain

That’s a huge win in terms of ownership. Less glue code. Fewer things to break.

#### Alerts

GCP Alerting closes the loop.

We write alerting policies as code, version them, roll them out per environment, and route them to the right destinations.

Email, SMS, Slack, PagerDuty — whatever the workflow is, the platform supports it.

Again, no extra infra to manage. No sidecar projects. No duct tape.

---

### The Multi-Cloud Question


At some point, someone always asks:

	“Aren’t we locking ourselves into a single cloud provider? What about multi-cloud?”

It’s a fair question — and one we’ve already thought through.

The reason we chose this setup isn’t because we’re cloud-loyal. It’s because it works. It gets us observability without spending months building platforms or years maintaining them.

What matters more to us is:
- Standards-based telemetry (OpenTelemetry)
- Structured, portable logs
- Configurable alerting as code
- Infrastructure we don’t have to babysit

Everything we’re emitting — metrics, traces, logs — is cloud-agnostic. If we want to switch backends tomorrow, we can.

And yes — logs could be routed elsewhere. But right now, they’re landing in a system that scales, integrates, and lets us focus on product instead of platform.

If “multi-cloud” becomes a real requirement, not just a slide title — we’re ready.

Until then, we’re shipping value. Not architecture diagrams.

---

### Final Words

That’s the story.

And yes — much of this is opinionated. Intentionally so.

Because our focus is delivering real value:
- To the engineers building and running our systems
- To the Gupa customers using the products we ship

Every decision we’ve made is in service of that.

We’re not here to talk logos, debate tools, or over-architect for slides. We’re here to build systems that:
- Work out of the box
- Scale without drama
- Are observable from day one
- Can be operated by teams, not specialists

We design for simplicity. We automate the boring parts. We make things safe, fast, and attractive for good engineers to own and evolve.

And most importantly — we make them deliver faster.

That’s where the value is.

And that’s what we ship.

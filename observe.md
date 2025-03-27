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
  - [Why We Did What We Did](#why-we-did-what-we-did)

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

You might have heard from techie-jargon people: "Let’s use Prometheus", or "ELK stack", or "Datadog". And of course, all those tools can be useful.

However, they will **not** be useful if you think it’s a matter of using the tool. As you will get very basic dashboards and UIs, with information inside that does not make any sense.

In reality, all those tools assume extra engineering. In large corps, you will always meet lots of people talking tools as solutions. That’s exactly what we don’t want.

We need to make sure problems are articulated as engineering problems. And solutions are engineered — not installed. Big difference there.

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

Now let’s try to blend different tools together, and think about engineering solutions.

Imagine someone — let’s call him Bob — works as a "Head of Logos".

Bob's offer is below (and usually that person will say, "that’s what we have done in my previous job"):

1. Elasticsearch for logs — it's great technology, super fast, amazing for logs. Read this blog.
2. Kibana as a UI to query the crazy logs (remember how complex it can get) — Kibana is the de facto UI for Elasticsearch.
3. Prometheus for metrics database — everyone is doing Prometheus, look at their website. It's really good.
4. Grafana for better dashboards — I always thought Grafana is more than dashboards, I thought it's monitoring. But yes, doesn’t matter. Grafana, let’s use it.
5. Alerts from Prometheus — you know Prometheus also supports alerts, look at their website.
6. Let’s agree that we want OpenTelemetry, and mix in a few instrumentations from Prometheus and friends to get some data for free.

### Back to Reality

Congratulations — you just solved all problems, Bob. Arrows, logos, colors here and there, and landing pages. You found tools that say they do things really well, you put them together on slides and solved the problem.

Now let’s go back to reality and see what your slide actually means.

#### 1. Elasticsearch

We need to host an Elasticsearch cluster. Yes, host it yourself. Make sure it works — always. Grows and scales without problems, as we will be pushing an insane amount of data. Unless you want to save on logs?

So you need some scalable way to host it. Oh, let’s put Kubernetes on our slides — sounds cool.

By the way, have you ever managed a k8s cluster? Super easy — RBAC, secrets, segregation of namespaces and networks, liveness... of course, no problem with that.

Oh by the way, if we put k8s there, we might as well put a Helm chart repo, as of course we will be using Helm with k8s.

Have you done Helm release management? Insane amount of YAML files, crazy Dockerfiles and etc? Very easy and fun.

Also, how many people do you know that have done all of this before?

Now, when that one logo added a huge drama — people, new teams, expenses, and operations — to our life, let’s have a look at the other logos.

But before we move — if we store logs in Elasticsearch, where do we store logs for Elasticsearch itself (aha, gotcha)?

#### 2. Kibana

We assume it’s easy to configure and connect to our Elasticsearch. Maybe.

Have you done it before?

Now when it connects, maybe, just maybe, we also need to think:
- Who can access it?
- SSO?
- Different roles?
- We need to create lots of widgets that are helpful.
- Who knows the query language?

GPT will help nowadays, but the rest is another operational-heavy logo.

#### 3. Prometheus

Remember — it’s a database, right?

So, same as with Elasticsearch:
- Where to host it?
- Scalability?

That’s when we find out that there is a different version of Prometheus that is semi-enterprise — it comes as a scalable instance.

Super high maintenance, as with any manual operational storage that must scale forever.

At this point we are spending a lot of money already — as it’s very likely nobody has done this before.

#### 4. Alerts

Let’s assume we now know how to deploy and configure Prometheus UI and alerting.

It’s actually hard. But let’s assume the hard part is done.

We have this nice UI — that is empty.

We learn the query language, we put something in, we do illusion of alerts. And we find out, we actually need to engineer a solution to deliver these alerts — via email, text, and so on.

Oh. We had not thought about it before.

#### 5. Grafana

Same story:
- Crazy configurations to connect to different data sources.
- Once done, you hope it works.

However, remember — now **you** are hosting this tool.

You put all these operation-heavy tools, each requiring lots of knowledge, just to get logs.

Amazing.

#### 6. OpenTelemetry

This was easy — seems like it works.

But we can’t test it — as everything else will be delivered in 5 years, after 3 transformation programs and many permutations of tools on slides.

### Dessert: The Inevitable Spiral

Imagine — just imagine — you have done all this. You found a way to automatically deploy everything and wire it together.

Guess what? This is part of your software now.

So you need to:
- Test that your logs are working
- Metrics are adequate
- Alerts are working

Ah — would that mean deploy and operate this in many environments to make sure it works?

Aha. So all these crazy clusters now must run in many different environments. Have their own release schedules, upgrades. And people who are not bored to death doing this operation.

At this point, we call it technical debt, and change jobs.

Someone else’s problem.

To be more precise — someone else’s **digital transformation** problem.

---

You just witnessed a simple logo from Bob — who does not talk technology — putting it on a slide and creating this chaos.

Remember the quote: **"that’s what we have done in my previous job"**.

And believe me — most of the time, that’s exactly what Bob did.

And he will do it again.

Our approach is:
**"that’s what we have done in our previous work, and that’s exactly why we will NOT do it again"**

We actually have done it.

And yes, maybe we contributed to those tools in open source, maybe we even contributed to books about how to put these things together.

And honestly, those tools are amazing.

But:
- Context matters
- Skills matter
- Experience matters

Many things actually matter.

There is always more than one way of doing the same thing and engineering.

And sometimes, even if you know exactly what you are doing — it’s **not** the reason to do it.

And **can** be the reason to **NOT** do it.

We belong to the latter.

---

### Why We Did What We Did

Now you might be interested: what have we done?

Good question. We prioritised the usual suspects — what matters to us the most.

#### 1. Instrumentation

We picked **OpenTelemetry and friends** to instrument all our code and logs. 

Pretty agnostic of toolchain — the data can be saved in any telemetry-friendly storage.

#### 2. Logs

We got instrumented logs. We thought, OK, we need a place to save them.

It’s JSON data — so relatively portable anyway.

We wanted to save, and forget about them.

**GCP Logging backend is delivering the promise.**

Can you push logs somewhere else? Yes — if you really need to. It’s JSON. Good luck though.

We just saved **millions of operational craziness** with this one.

#### 3. Metrics

Instrumented metrics are now flying around. Same here — save and forget.

No scalability issue. No upgrades. No maintenance windows. No tickets. No emails.

It just works.

**GCP Metrics backend**, telemetry-friendly. 

We just saved a few millions again.

#### 4. Dashboards

By now, we already sold our soul to GCP — so you won’t be surprised to hear about dashboards in Google.

Is it cool? Absolutely.

Because:
- First — it works.
- Second — you get whole **GCP RBAC control** on users for free.
- Zero maintenance.
- Lots of widgets.
- Surprise — **Terraform**: so you can have dashboards as code.

#### 5. Alerts

**GCP Alerts.** From emails to text messages, rules, escalations — all as code.

#### How Long Did It Take?

Couple of weeks.

We created a **library**, that comes as a dependency, with most things configured — so you save time on glue code.

You mix it into your Spring-like project, you get pretty much everything out.

You learn how to use Terraform, and each time you install your software in GCP, it comes with:
- Its own dashboards
- Its own alerts
- Its own logs

**As one package.**

You build it — you own it.

---

### The Bob Reaction

At this point, Bob sends unhappy emails:

> "Yes, but we don’t want to be locked to the cloud implementation. We want to be multi-cloud..."

And he adds a few other jargon-y words there.

Our answer to Bob:

> Bob, it’s a good point. And of course, you’ve done it before. That’s exactly why we kept the data formats not opinionated, and portable.
>
> Once multi-cloud works for everything apart from logging, we promise — we will also just forward logs to any other logo.
>
> But for now, can we please please please focus on delivering **user value** and stop the logo conversation?
>
> Also, if you need help with multi-cloud — let us know. We might be able to.

---

### Final Words

Story ends here.

And we agree — many things come opinionated.

That’s because we focus on delivering **value** to our users:
- The engineers using our solutions
- The Gupa customers using our digital products

We are pushing for value with every commit.

At the time of writing this README, there is no value in talking logos.

We engineer agnostic and solid solutions — everything:
- Automated
- Safe
- Attractive to good engineers
- Available quicker to Gupa customers


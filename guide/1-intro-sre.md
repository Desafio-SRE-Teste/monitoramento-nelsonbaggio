id:     intro-sre
Title:  SRE Academy - Introdução ao SRE

# SRE Academy - Introdução ao SRE

## O que é SRE?

SRE (Site reliability engineering) é uma série de práticas e
princípios que unificam os problemas de infraestrutura e operações,
com soluções de engenharia de software. Seu foco é
para garantir aplicações altamente escaláveis e estáveis, garantindo funcionalidade e indicadores de alta disponibilidade.

## Práticas SRE

 As práticas de um time de SRE podem variar mas conseguimos ter algumas bastante comuns entre a maioria dos times, sendo elas:
 
    -Testes de stress de aplicações
    -Definir e medir metas de —SLIs, SLOs, e error budgets.
    -Non-Abstract Large Scale Systems Design (NALSD) com foco em disponibilidade.
    -Implementação de observabilidade na aplicação.
    -Gerenciamento de incidentes e problemas.
    -Capacity planning.
    -Alteração ou adequações de processos CI/CD.
    -Chaos engineering.


## Monitoring

Without monitoring, you have no way to tell whether the service is even working; absent a thoughtfully designed monitoring infrastructure, you’re flying blind. Maybe everyone who tries to use the website gets an error, maybe not—but you want to be aware of problems before your users notice them. We discuss tools and philosophy in Practical Alerting from Time-Series Data.

## Incident Response

SREs don’t go on-call merely for the sake of it: rather, on-call support is a tool we use to achieve our larger mission and remain in touch with how distributed computing systems actually work (and fail!). If we could find a way to relieve ourselves of carrying a pager, we would. In Being On-Call, we explain how we balance on-call duties with our other responsibilities.

Once you’re aware that there is a problem, how do you make it go away? That doesn’t necessarily mean fixing it once and for all—maybe you can stop the bleeding by reducing the system’s precision or turning off some features temporarily, allowing it to gracefully degrade, or maybe you can direct traffic to another instance of the service that’s working properly. The details of the solution you choose to implement are necessarily specific to your service and your organization. Responding effectively to incidents, however, is something applicable to all teams.

Figuring out what’s wrong is the first step; we offer a structured approach in Effective Troubleshooting.

During an incident, it’s often tempting to give in to adrenalin and start responding ad hoc. We advise against this temptation in Emergency Response, and counsel in Managing Incidents, that managing incidents effectively should reduce their impact and limit outage-induced anxiety.

## Postmortem and Root-Cause Analysis

We aim to be alerted on and manually solve only new and exciting problems presented by our service; it’s woefully boring to "fix" the same issue over and over. In fact, this mindset is one of the key differentiators between the SRE philosophy and some more traditional operations-focused environments. This theme is explored in two chapters.

Building a blameless postmortem culture is the first step in understanding what went wrong (and what went right!), as described in Postmortem Culture: Learning from Failure.

Related to that discussion, in Tracking Outages, we briefly describe an internal tool, the outage tracker, that allows SRE teams to keep track of recent production incidents, their causes, and actions taken in response to them.

## Testing

Once we understand what tends to go wrong, our next step is attempting to prevent it, because an ounce of prevention is worth a pound of cure. Test suites offer some assurance that our software isn’t making certain classes of errors before it’s released to production; we talk about how best to use these in Testing for Reliability.

## Capacity Planning

In Software Engineering in SRE, we offer a case study of software engineering in SRE with Auxon, a tool for automating capacity planning.

Naturally following capacity planning, load balancing ensures we’re properly using the capacity we’ve built. We discuss how requests to our services get sent to datacenters in Load Balancing at the Frontend. Then we continue the discussion in Load Balancing in the Datacenter and Handling Overload, both of which are essential for ensuring service reliability.

Finally, in Addressing Cascading Failures, we offer advice for addressing cascading failures, both in system design and should your service be caught in a cascading failure.


## Development

One of the key aspects of Google’s approach to Site Reliability Engineering is that we do significant large-scale system design and software engineering work within the organization.

In Managing Critical State: Distributed Consensus for Reliability, we explain distributed consensus, which (in the guise of Paxos) is at the core of many of Google’s distributed systems, including our globally distributed Cron system. In Distributed Periodic Scheduling with Cron, we outline a system that scales to whole datacenters and beyond, which is no easy task.

Data Processing Pipelines, discusses the various forms that data processing pipelines can take: from one-shot MapReduce jobs running periodically to systems that operate in near real-time. Different architectures can lead to surprising and counterintuitive challenges.

Making sure that the data you stored is still there when you want to read it is the heart of data integrity; in Data Integrity: What You Read Is What You Wrote, we explain how to keep data safe.

## Product

Finally, having made our way up the reliability pyramid, we find ourselves at the point of having a workable product. In Reliable Product Launches at Scale, we write about how Google does reliable product launches at scale to try to give users the best possible experience starting from Day Zero.



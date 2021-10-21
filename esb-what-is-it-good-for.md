layout: true
.footer[2021-03-29 / Simon Lundström, IT-avdelningen]

---
class: first-slide
# Enterprise Service Bus

#### What is it good for?

---
# Agenda

1. ESB at SU
1. Message Queue
1. OSGI Blueprint integrations
1. Karaf
1. Future work

---
# ESB at SU

Just a platform provider. We don't create integrations the users do.

---
## ServiceMix
A package of:
* Apache ActiveMQ - Message Queue server
* Activiti - BPM
* Apache Camel - integration framework based on [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/toc.html)
* Apache CXF - services framework, SOAP, REST, CORBA(!) etc
* Apache Karaf - application server for integrations
* etc…

???
Many people talk about ServiceMix. We use it but we also don't use everything.

---
## ~~ServiceMix~~
A package of:
* ~~Apache ActiveMQ - Message Queue server~~
* ~~Activiti - BPM~~
* Apache Camel - integration framework based on [Enterprise Integration Patterns](https://www.enterpriseintegrationpatterns.com/toc.html)
* Apache CXF - services framework, SOAP, REST, CORBA(!) etc
* Apache Karaf - application server for integrations
* ~~etc…~~

???
We run ActiveMQ separately and don't use much of the things included.

---
# ActiveMQ
<pre>
         ┌────────┐          
         │Producer│          
         └────────┘          
┌────────┐    │    ┌────────┐
│Consumer│    │    │Consumer│
└────────┘  STOMP  └────────┘
     │        │         │    
     └──JMS───┼───AMQP──┘    
              ▼              
       ┌────────────┐        
       │esb.it.su.se│        
       └────────────┘        
              │              
              ▼              
      ┌───────────────┐      
      │ Message Queue │      
      └───────────────┘
</pre>

???

Supports every MQ protocol available.

---
# Karaf
<pre>
  ┌─────────┐      ┌──────────┐
  │Producer:│      │ ┌──────┐ │
  │ Github  │      │ │Queue:│ │
  └─────────┘   ┌──┼▶│Github│ │
       │        │  │ │mirror│ │
       ▼        │  │ └──────┘ │
┌────────────┐  │  │        MQ│
│esb.it.su.se│  │  └──────────┘
└────────────┘  │              
┌──────┬────────┼──┐           
│      ▼        │  │           
│┌──────────┐   │  │           
││Blueprint:│   │  │           
││  github  │───┘  │           
│└──────────┘      │           
│Integration server│           
└──────────────────┘
</pre>

???
An integration can do anything.

---
## Comparison Web / Integration platform

<style>table {  width: 70%; } td { width: 50%; } td,table,th { border: 1px solid black;}</style>

| Web | Integration | What? |
| --- | --- | --- |
| pom.xml/build.gradle | feature.xml | Dependency management |

--
| .war | OSGI blueprint.xml / bundle.jar | Application code |

--
| Tomcat | Karaf | Container runtime |

--
| Tomcat EE | ServiceMix | Application server package |

--
| JDBC | JMS | API |

--
| DB (MySQL) | MQ (ActiveMQ) | Persistent Storage |

---
# OSGi Blueprints
### XML-programming(!) with Camel

```xml
<route>
  <from uri="cxfrs:///githubmirror?modelRef=
  file:/local/githubmirror/conf/model.xml&amp;bindingStyle=SimpleConsumer" />
  <convertBodyTo type="java.lang.String" />
  <bean ref="githubWebhookValidator" method="validate"/>
  <inOnly uri="activemq:queue:su.it.githubmirror" />
</route>
```

---
# OSGi Blueprints
### XML-programming(!) with Camel
```xml
<route>
  <from uri="activemq:su.it.ladok3.feed.su.sukat" />
  <choice>
    <when>
      <xpath>si:KontaktuppgifterEvent|si:StudentEvent|[…]</xpath>
      <!-- Convert XML to JSON and "convert" it to our prefered format -->
      <marshal ref="xmljsonWithOptions" />
      <setBody>
        <jsonpath>StudentUID</jsonpath>
      </setBody>
      <setBody>
        <simple>{"studentuid": "${body}"}</simple>
      </setBody>
      <inOnly uri="activemq:queue:su.it.sukat.ladok" />
    </when>
  </choice>
</route>
```

---
### ActiveMQ
What's the difference?
#### Topic
* Every message produced is sent to 0..N subscribers/consumers

#### Queue
* Every message produced is sent to 1 subscriber/consumer
* Can have multiple consumers for e.g. load balancing/redundancy.

<small>https://activemq.apache.org/how-does-a-queue-compare-to-a-topic.html<br>https://docs.microsoft.com/en-us/azure/service-bus-messaging/service-bus-queues-topics-subscriptions</small>

---
# Routing - PubSub via Integration
<pre>
┌────────────────┐       ┌──────────────────────┐
│LADOK ATOM FEED │       │┌────────────────────┐│
└────────────────┘       ││Queue: CourseService││
         ▲               │└────────────────────┘│
       HTTP              │┌───────────────────┐ │
   ┌─────┴────────────┐  ││Queue: SUKAT LADOK │ │
   │┌─────────┐       │  │└───────────────────┘ │
   ││  LADOK  │    ┌──┼─▶│┌───────────────────┐ │
   ││Consumer │────┘  │  ││Queue: TimeEditTool│ │
   │└─────────┘       │  │└───────────────────┘ │
   │Integration server│  │┌───────────────────┐ │
   └──────────────────┘  ││ Queue: SchemaTool │ │
                         │└───────────────────┘ │
                         │            MQ-server │
                         └──────────────────────┘
</pre>

???
Routing messages can be done in many different ways.

---
# Routing - MQ internal with filtering
<pre>
┌────────────────┐                             
│ SUKAT Producer │                             
└────────────────┘                             
         │                                     
┌────────┼────────────────────────────────────┐
│     STOMP             ┌───────────────────┐ │
│        │              │    Queue: Taxi    │ │
│        ▼              └───────────────────┘ │
│┌───────────────┐      ┌───────────────────┐ │
││    Topic:     │   ┌─▶│  Queue: SAS Viya  │ │
││su.sukat.users │   │  └───────────────────┘ │
│└───────────────┘   │  ┌───────────────────┐ │
│        │           │  │     Queue: AD     │ │
│        │           │  └───────────────────┘ │
│        ├──Version 1┘                        │
│        │              ┌───────────────────┐ │
│        │              │Queue: Affiliation │ │
│        └──Version 3──▶│      mailer       │ │
│                       └───────────────────┘ │
│                                   MQ-server │
└─────────────────────────────────────────────┘
</pre>

---
# Statistics
* 26 integrations in Karaf
* ~90 consumers connected to ActiveMQ
* Average message size ~11KB
* ~900 000 messages a week
* 70 queues
* 8 topics

???

---
# Future work

* [PLATTFORM-344](https://jira.it.su.se/browse/PLATTFORM-344)<br>
  Switch authentication backend to Kerberos

--
* [PLATTFORM-555](https://jira.it.su.se/browse/PLATTFORM-555)<br>
  Upgrade ActiveMQ to get slow consumer detection. No more queue depth alerts.

--
* [PLATTFORM-575](https://jira.it.su.se/browse/PLATTFORM-575)<br>
  Migrate away from Karaf and write integrations as "microservices" in Kubernetes

---
# Questions?

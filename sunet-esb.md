layout: true
.footer[2021-11-10 / Simon Lundström, Stockholms Universitet, IT-avdelningen]

---
class: first-slide
# Enterprise Service Bus

#### What is it good for?

---
# Agenda

1. whoami(1)
1. ESB at SU
1. Message Queue
1. OSGI Blueprint integrations
1. Karaf
1. Råd till SUNET

---
# whoami(1)

FIXME

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
# Råd till SUNET

ESB == Integrationsplattform
---
# Nackdelar med ESB
* Dålig isolering

--
  * Alla måste köra samma version av bibliotek

--
  * Alla kan ha sönder din plattform

--
* Oftast är koden extremt specifik för plattformen (OSGi)

--
* Utvecklarna vill skriva koden på sitt sätt i sitt språk

--
* Hur löser man deployment?

--
  * Alla ska sköta kod, CI och CD på samma sätt?

--
Över flera lärosäten?

--
GLHF

---
# Fördelar med ESB
* XML-programmering!

--
  * Allt enkelt är superenkelt!
  * Allt komplicerat är komplicerat

--
* Slipper sköta en massa själv som kund

--
* Kan fokusera på "sin sida"
???
Behöver bara sköta sin egna företagslogik

---
# Vad istället?

--
En ESB!

# 🙄

---
# Vad istället?

* Asynkrona och synkrona integrationer

--
  * En meddelande tjänst (MQ)

--
  * REST API(er)

--


LADOK hade rätt hela tiden!? Tydligen! Eller, nästan.

---
# Asynkron överföring

* Tunna meddelanden (LADOK-style)

--
* Tjocka/feta/rika meddelanden (LIS-style?)

--

![I have no idea what I'm doing](https://memegenerator.net/img/instances/49626302/i-have-no-idea-what-im-doing.jpg)

???
Kan inget om LIS. Har försökt läsa på men...
Out of sync meddelanden? Blir det samma resultat?
Är det riktig Event Sourcing/Command and Query Responsibility Segregation?

---
# Asynkron överföring
#### SU-style!

* ~~Tunna meddelanden (LADOK-style)~~
* ~~Tjocka/feta/rika meddelanden (LIS-style?)~~
* Triggers (supertunna meddelanden ibland med metadata)!
  * Detta objekt bör syncas synkront

--
  * Spelar ingen roll när det sker i tid. Datan blir bara möjligen försenade

--
  * Beroenden mellan objekt blir komplicerat. When in doubt: DLQ!
???
Tänk: en kurs lägger till en student innan studenten finns.

Dead Letter Queue. Konsumenten tycker inte den kan hantera meddelandet, låt någon kolla på det senare.

Bara den personen/objektet blir lidande. Alla andra får sin data uppdaterad.

---
# En "ESB"

SOAP är tillbaka!

???
Allt gammalt är nytt igen!

---
# En "ESB"

* [AsyncAPI](https://www.asyncapi.com/) [Exempel](https://studio.asyncapi.com/?url=https://raw.githubusercontent.com/asyncapi/asyncapi/v2.2.0/examples/simple.yml)

--
* [OpenAPI](https://www.openapis.org/) [Exempel](https://petstore.swagger.io/)

--
* Lärosäten får ratta sin egen konsument på befintlig plattform.

--
  * Ska ni nödvändigtvis hosta? Kubernetes
???
Containers. Eller iaf någon sorts managerad container lösning.


---
# Lösta problem?

| Problem | Löst? |
|--- |--- |
| Isolering | ✅ |
???
Alla kan köra vilka versioner av bibliotek dom vill. Containers kan bli begränsade eller så är det lärosätets problem.
--

| Valfritt språk?* | ✅ |
???
Dom får bara hjälp av
[OpenAPI](https://github.com/OpenAPITools/openapi-generator#user-content-overview)
och
[AsyncAPI](https://github.com/asyncapi/generator#list-of-official-generator-templates)
om språket stöds av generatorerna.
Allt går, det finns docs och det finns också projekt att jämföra sin kod med en
API spec.
--

| Deployment | ✅ |
???
Alla kör sin egen konsument eller kör i en av SUNET hostad containerlösning som inkluderar CI och CD.
--

| Fokusera på "sin sida" | ✅ |
???
Koden fram till affärslogiken är klar

---
# Så vad bidde det då?

* MQ
* REST API(er)
* Specifikationer och dokumentation
* Central AuthN (OpenID Connect/OAuth/API-nycklar)

Typ en ESB

---
# Questions?
--

Slides at https://github.com/simmel/slides via https://github.com/stockholmuniversity/su-remark-template/

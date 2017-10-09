---
layout: page
title: Resume
description: chrislovecnm resume
---

**Chris Love**

Email: [clove@cnmconsulting.net](mailto:clove@cnmconsulting.net)

**Cloud Kubernetes DevOps Architect and Engineer**

More than eighteen years of diverse experience creating custom solutions; Kubernetes Implementations, DevOps solumtions, Big Data Implementations,and scalable micoservices.  Proven ability to solve complex problems with limited resources, on time, and within budget. Retained key clients in high pressure and politically charged situations by identifying and creating unique solutions, thereby maintaining revenue.   Industry leading blend of cutting edge Kubernetes Deployments, Big Data Technologies, and system integration. Kuberentes core contributor, kops owner, and lead for the AWS Kubernetes Special interest groups.

**Skills **

<table>
  <tr>
    <td>Cloud Stacks</td>
    <td>Amazon (EC2), Google Compute, vSphere 6.x</td>
  </tr>
  <tr>
    <td>DevOps Tools</td>
    <td>Kubernetes, Docker, Graphana, Prometheus, Terraform, Knife, Icinga, logstash, collectd</td>
  </tr>
  <tr>
    <td>Big Data Stacks</td>
    <td>Apache Cassandra, Apache Spark, Elastic (ELK), Redis, Hadoop</td>
  </tr>
  <tr>
    <td>Security Tools / Integrations</td>
    <td>OAuth, SSO, SAML, LDAP, HMAC, Rest Based Tokens</td>
  </tr>
  <tr>
    <td>Enterprise Tools</td>
    <td>VMware Virtual Cloud Director 5.5, VMware VRealize Suite, Tomcat, Nginx, Apache ActiveMQ 5, Apache Camel, RabbitMQ, MySQL, Percona</td>
  </tr>
  <tr>
    <td>Operating Systems</td>
    <td>Linux (Gentoo, RHEL, CentOS, Ubuntu, CoreOS), Mac OS X, Microsoft Windows Server</td>
  </tr>
  <tr>
    <td>Languages</td>
    <td>Go, Java, Groovy, Scala, Python, Ruby, C/C++, Lua 5.1 </td>
  </tr>
  <tr>
    <td>Dev Software</td>
    <td>Intellej, Bazel, Git, Gradle, Maven, Hudson, Jenkins</td>
  </tr>
</table>

**Experience**

<table>
  <tr>
    <td>Kubernetes Organization</td>
    <td>04/2016  – Present</td>
  </tr>
</table>

Kubernetes contributor and active community member.  Focus is primarily in three areas;
kops, the AWS special interest group, and mentoring.

kops is easiest way to get a production grade Kubernetes cluster up and running
in EC2 or GCE.  Added support for installing CNI providers, and cluster
validation. Improved kop rolling-update to include support for draining and
validating clusters. Worked on support for air gapping kops intallations and
re-using share security groups and IAM profiles.  Assisted other contributors
with design and reviewed numerous Pull Requests. Implemented `kops` bi-weekly
developer offices hours.

Currently volunteerring as one of the leaders in the AWS special interest group.
Facilitating meetings, and involved in product decisions for AWS cloud support
in the Kuberentes project.  Implemented critical fix [
#39551](https://github.com/kubernetes/kubernetes/pull/39551) for Kubernetes
1.4.9 that enabled changing times on volume attach/detach reconciling sync to
fixing impact to AWS by chrislovecnm


kops owner
* kops
  * manifests
  * cni providers
  * https://github.com/kubernetes/kops/pull/2964
  * documentation
  * security groups, assets, iam profiles
  * validate
  * rolling-update
* Cassandra
* AWS sig
  * https://github.com/kubernetes/kubernetes/pull/39551 Enable changing times on volume attach/detach reconciling sync to fixing impact to AWS by chrislovecnm · Pull Request #39551 · kubernetes/kubernetes
* Kubernetes Docs
  * https://github.com/kubernetes/kubernetes.github.io/pull/1486

<table>
  <tr>
    <td>Cisco</td>
    <td>10/2015  – 12/2015</td>
  </tr>
</table>


Contributed to Cisco Cloud open source project named Mantl.  Mantl is a platform for rapidly deploying globally distributed services.  Mantl platform utilizes Anisble, Ansible plugins, and Terraform to deploy cloud environments.  Once the cloud environment and servers are launched, Mantl can be configured to install Mesos, Kubernetes, Docker, and other technologies such as ELK.  Focused on furthering Mantl, Terraform, and vSphere integration.  Assisted in setting up vSphere 6.0 test cluster, allowing for end to end system tests.  Contributed to Terraform project, written in Go, and assisted in managing various bugs on Github.  Modified and tested Mantl Ansible python plug-ins allowing for use of new terraform provider.  Wrote white paper on running Cassandra on either Mesos or Kubernetes, which contrasted pros and cons.  Completed assessment and diagnosis of Cassandra challenges.

<table>
  <tr>
    <td>Giving Tools - Volunteer Work</td>
    <td>06/2011- Present</td>
  </tr>
</table>

Team implementing platform to help non profits gather online donations and marketing data.  IT infrastructure is completely cloud based hosted in Google. Platform is clustered within a Kubernetes system running  within Dockers.  Puppet management was utilized in the past when hosted on VMware ESX.  Assisted clients with deployments on EC2.Donation collections are handled by a AngularJS / Grails application and various rest services.  Ad Serving components utilize Redis, Lua and Nginx in order to deliver high through-put ad serving.  Various components of Ad Serving also include aspects of semantic analysis of web page content through the usage of machine learning.  Architecture stack include: Spring Java components, Apache Spark, Cassandra, Redis and Lua, Apache Camel, Elastic, and Apache Activemq.  Cassandra is used to store various data elements, and elastic provides aggregations and other data operations.  Analytics are delivered to clients via JavaScript graphing.  Data is analyzed and collected through the use of Elastic and Apache Spark.  Messaging and other EIP patterns are delivered through the use of Apache Camel.

<table>
  <tr>
    <td>CenturyLink Technology Solutions - Formerly Savvis</td>
    <td>01/2013  – 09/2015</td>
  </tr>
</table>

Held multiple Software and DevOps roles within the company.  Projects varied from acting as a subject matter expert for VMware Cloud API, to interacting with top client’s dealing with mission critical issues.  Software projects primarily included Spring based Java, and ruby working on chef and knife.  Involved in the creation of an online market place for virtual machines within CenturyLink’s cloud product.  Lead member of team to allow CenturyLink to sell VMware vRealize Operations to their clients.

Assisted teams to overcome hurdles with virtual machine deployment. Worked with internal and VMware support to resolve software and system issues. Discovered various mission critical bugs before they directly impacted clients, and created SLA fines for CenturyLink.  Introduced the use of knife, a component of chef, for use by the DevOps engineers. Knifed reduced the deployment time in half of a fully managed and configured virtual machines.  Created reports that allowed the client to reduce their costs, and determine low and high use systems. Assisted DevOps teams with various puppet improvements.

<table>
  <tr>
    <td>Smile Brands</td>
    <td>08/2012 – 01/2013</td>
  </tr>
</table>

Leading initiative to process dental insurance claims via integration with various partners.  Utilizing Apache Camel and various web services to process and submit claims to insurance processors.  Running Camel inside of Apache Karaf within an OSGI based classloading system.  Data layer is Oracle based utilizing JPA to communicate with the database.  Web and Apache Camel layers are implemented using various Spring 3.2 components including: REST, MVC, AOP, and dependency injection.

<table>
  <tr>
    <td>Dish Network</td>
    <td>06/2011 – 12/2011</td>
  </tr>
</table>

After initial project served is part time consultant assisting with various technical and implementation challenges.  Worked hand in hand with SecureAuth (SAML identity company) to create SAML SSO environment to pilot security model for Dish Network and Blockbuster Video. Created POC implementation of a secure J2EE web application hosted in WebLogic with user identities and roles secured via SecureAuth SAML IdP.  Assisted SecureAuth with product improvements in order to enable SecureAuth to act as an IdP for WebLogic.

Fully documented POC and trained Dish Network staff on SAML, the implementation of secure WebLogic application, and the implementation of security utilizing SecureAuth. Integrated Tomcat with SecureAuth utilizing Spring Security SAML Extension.

<table>
  <tr>
    <td>AT&amp;T - Spring / vFrabric Consultant</td>
    <td>08/2011 – 7/2012</td>
  </tr>
</table>


Serving as a Senior Consult, working with AT&amp;T, to integrate VMware Virtual Cloud Director with various network and IP provisioning and billing systems. Created a Grails audit tool to track discrepancies between AT&amp;T network provisioning and the usage of those provisions within VMware Virtual Cloud Director.  Grails application utilizes Twitter Bootstrap for look and feel, and jQuery as it’s JavaScript library.   Leveraged groovy to lazy load process XML that was created by a Spring Integration system that aggregates data from Oracle databases into XML reports.

Built integration bridge linking vCD and IP / Network provisioning tools.  Utilized Spring MVC and Spring Rest Client to build a rest bridge allowing data conversion between XML and JSON.  Assisted with the creation of  Spring Integration system that listens for RabbitMQ messages from vCD then routes the messages to AT&amp;T’s billing system. Implemented new continuous integration build system utilizing Jenkins and Maven.  Started test driven development and maven build policies  for release management.  Trained and mentored AT&amp;T staff in Spring, Maven, Unit Testing, and CI best practices and industry standards.

<table>
  <tr>
    <td>Progress Software – ADP and Garden City Group</td>
    <td>05/2011 – 07/2011</td>
  </tr>
</table>


ADP – Expanded functionality of previously implemented Spring based web services.  This service layer provides custom capability to create forecasts of future tasks within a running Savvion BusinessManager process.  Mentored staff in utilizing Spring based JUnit testing in order to save considerable time testing functionality outside of the Savvion container.

Garden City Group – Assisted with the creation of a custom adapter for the integration of web services. Implemented various examples including a web application that hosted CXF web services. This application provides examples of how to integrate an external web application with Savvion BusinessManager via the BizLogic Client API.  Using an external web application allows the creation of new Savvion components while not adding the custom code to the Savvion installation.  This architecture allows for easier future upgrades.  Worked with GCG to create production architecture for SBM 7.8 deployed on RedHat JBoss AS.

<table>
  <tr>
    <td>Accenture – Healthcare Client </td>
    <td>07/2010 – 04/2011</td>
  </tr>
</table>


Contracted as lead architect and portal developer. Brought on to address various WebCenter Interaction needs and PeopleSoft content integration. Discovered challenges with Single Sign On and WSRP solution.  Worked with Oracle WebCenter development group to architect WSRP / SAML solution utilizing the following technology stack: WebCenter Interaction 10g, WebCenter Pagelet Producer 11g, Oracle Virtual Directory 11g, Oracle Access Manager 11g, and PeopleSoft.  The solution allows users to access PeopleSoft content from within a WebCenter Interaction Portal, and the user’s identity is passed to PeopleSoft via Oracle Access Manager and SAML.

Designed and implemented web application to provide user access to Siebel tickets via a WebCenter Interaction portlet.  User Interface utilized Ext JS, Spring, CXF Rest and Webservices.  Stabilized production deployment using WLST and Maven.  Worked with Oracle On Demand to implement and deploy various components and technologies including: WebCenter Interaction, WebCenter Spaces, Single Sign On, WebLogic, Identity Management and SSL Security. Mentored client staff in various technologies including: Maven, Git, Spring, WLST, Ext JS, and other Oracle technologies.

<table>
  <tr>
    <td>ADP</td>
    <td>02/2010 – 06/2010</td>
  </tr>
</table>


Implemented custom web service to provide access to various data points within certain Savvion BusinessManager 7.5 processes.  Utilized BizLogic APIs, BizLogic Query Service, and BizStore Query Service to access and aggregate various data points within processes.  Wrote custom service that allowed a process to be completed while setting a data slot before the process was completed.  Developed a service layer that allowed the setting of various data slot values within a process.  Created JSP front-end to interact with the above services utilizing EXT JS libraries and JSON.  Worked with various BizLogic and BizSolo process flows.

<table>
  <tr>
    <td>Intuit</td>
    <td>03/2009 – 12/2009</td>
  </tr>
</table>


Re-architected legacy daemon process into a web application utilizing Spring, JPA, RestEasy, CXF, Apache Camel and Apache ActiveMQ.  System integrated multiple systems including: databases, VMLogix, proprietary interfaces and source control.  Reduced overall lines of code by approximately 40%, while increasing unit test coverage by approximately 150%.  Increased system performance by 400%, by utilizing ActiveMQ and Apache Camel.  System orchestrated software testing utilizing VMLogix LabManager and virtual images running on VMware ESX servers.  System performance now allows Intuit to run approximately 1100 software tests overnight, testing web based and desktop software products.

Assisted with mission critical project that created various Maven plug-ins used for Intuit’s build system.  Built system utilizes Hudson and Perforce.  Migrated various Savvion 6.0 projects to Savvion 7.0.

<table>
  <tr>
    <td>Camico</td>
    <td>08/2007 – 02/2009</td>
  </tr>
</table>


Led a team creating web based insurance applications that allows clients to purchase or renew professional liability insurance policy(s).  Camico saw positive income from their new business application within a week of the initial launch.  Integrated multiple systems utilizing Oracle AquaLogic BPM.  These systems included: SalesLogix CRM, Camico’s Policy System ‘PLUS’, communicating with various .Net web services, databases, and business analytics.  Co-authored custom .Net WebControl / AJAX library extending .Net controls while utilizing MooTools and Clientcide JavaScript libraries.

Worked with Camico employees to improve unit testing, system testing, source control, installation procedures, and other software best practices.   These practices were instrumental in reducing overall time to market by approximately 33%.  Mentored developers in basic software development, AquaLogic BPM, Java / C# web service interoperability, and general software delivery practices.

<table>
  <tr>
    <td>Warner Bros.</td>
    <td>03/2007 – 05/2007</td>
  </tr>
</table>


Brought on to the Digital End-to-End (DETE) program to provide: architecture health checks, mentoring across various project components, and the delivery of service layer framework to access AquaLogic BPM.  DETE is Warner Bros. vision of the future studio environment wherein content is created, processed, managed and distributed completely digitally.  The project’s cornerstone technologies included AquaLogic BPM, WebCenter Interaction, AquaLogic Enterprise Service Bus, and HP Digital Media Platform.

Collaborated with different development teams in order to stabilize and implement base frameworks for various project components including: data service layer utilizing hibernate with Java 5.0 annotations, automated build environment with maven2, and review of system hardware infrastructure in order to support an HA environment. Delivered Spring aware service layer framework that allowed various software components to access ALBPM via its Process API (PAPI).

<table>
  <tr>
    <td>Option One Mortgage</td>
    <td>02/2007 – 03/2007</td>
  </tr>
</table>


Developed a paperless mortgage AquaLogic BPM application.  Implemented WebLogic PageFlows communicating with AquaLogic BPM via Process API calls. The custom PageFlows replaced the out of the box ALBPM user interactive pages.  Collaborated with the client performance tuning WebLogic and ALBPM Engine running on a VMWare virtual server.

<table>
  <tr>
    <td>Micron Technologies</td>
    <td>07/2006 – 01/2007</td>
  </tr>
</table>


Led the implementation of a Savvion BusinessManager 6.8, on BEA WebLogic 8.1 server.  The rollout included fifteen systems with five systems in production.  Integrated WebLogic applications with SiteMinder agents on both Apache and WebLogic, to provide Single Sign On solution.  Corrected LDAP and Savvion integration issues that reduced client’s login time from over ten minutes to less than four seconds.

Implemented company’s overall BPM strategy and practices, while mentoring various business units with BPM implementations.  BPM projects ranged from manufacturing excursions to providing executive level metrics and score cards.  Created production release procedures that leveraged CruiseControl, Maven and Subversion.

**Military Experience**

<table>
  <tr>
    <td>United States Air Force and DOD</td>
    <td>06/2005 – 08/2005</td>
  </tr>
</table>


Honorably discharged, in 1994, from United States Air Force with the rank of E-4, and continued to work for department of defense.  Decorated veteran with time spent in Desert Storm.

**Education**

**University of Colorado, **Boulder, Colorado

Electrical Computer Engineering

Attended three years of college, before being recruited by an employer for a software position.

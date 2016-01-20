---
layout: post
title: Using Ria Services Soap and Windows Phone 7
date: 2010-09-24 04:44
author: mravinale
comments: true
categories: [Silverlight, Windows Phone 7]
---
I remember last year or so at this time, I began to glimpse <strong>Ria Services</strong>, from that preview so far <strong>Ria Services</strong> grew strongly and became something more just than a framework for creating applications <strong>RAD</strong> (Rapid Application Development) for Silverlight.

<strong>Ria Services</strong> initially was conceived  to expose our business logic as a service, taking as a model our data access layer of EF, and get our domain in a client-side context to work with Silverlight in a dynamic manner, facilitating thus the development and access to data.

However for large scale applications we may have other requirements, such as: Exposing our services in a way that they can be consumed from other systems, or decouple the business logic from the services, to make it interchangeable or use other tools like workflow in the same, and/or perhaps also to decouple the data access layer to use other ORM than EF .

For both problems there are solutions that <strong>Ria Services</strong> team has implemented in order to continue using the facilities that it has given us working on Silverlight.
For now we will focus on the first problem, the interaction with other systems in the next article I`ll deal with the decoupling between layers.

With a friend we’ve share a vision of using <strong>RIA Services</strong> for an application that could access these services from external system in this case <strong>Windows 7 Phone</strong>!!!.

Make sure we have the latest version of Ria Services: <a href="http://www.silverlight.net/getstarted/riaservices/" target="_blank">http://www.silverlight.net/getstarted/riaservices/</a>

1. First of all we need to create a Web application in Asp.Net as Empty.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:06b7b6b9-c70c-4785-a360-17971ad5c635" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/08x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/01.png" border="0" alt="" width="429" height="273" /></a></div>
2. In this case, we can designate it as “SoapRiaService”.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:eea37702-548a-44ad-9c33-9d87de849f2a" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/18x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/18.png" border="0" alt="" width="425" height="253" /></a></div>
3. Add a database where we will test our Employee table .
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:6671904a-8d4e-4329-8eab-8548665343ec" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/38x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/31.png" border="0" alt="" width="425" height="252" /></a></div>
4. Then select Entity Framework 4 new item and write it the database conection name, you could use "SoapTestDbEntities".
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:3c6eae40-957f-4a0d-aa51-6a6cc7c9615d" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/48x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/41.png" border="0" alt="" width="429" height="261" /></a></div>
5. For this test and for now we will take the Employees table to create our mapping
<div class="wlWriterSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/58x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/51.png" border="0" alt="alt" width="420" height="243" /></a></div>
6. Ok, we already have our entity Employee in our ORM. Now we need to do a solution build.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:26cdae18-27a9-490f-815a-7f5660bae081" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/68x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/61.png" border="0" alt="" width="433" height="265" /></a></div>
7. As a next step we need to create our domain service and get the interface for the basic CRUD operations in order to make it accessible to our client.
To achieve this we need to add a new component called Domain Service Class that is our Ria Service. In this case we may call it “EmployeeDomainService”.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:77107263-4a86-456e-83d9-6fbb8903f6cc" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/78x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/72.png" border="0" alt="" width="429" height="256" /></a></div>
8.  When we add a Domain Services class  using the wizard we can choose the entity that we need to apply the editing operations from the context of EF.
Choose properties enabling editing metadata generation, which will allow us to access the properties of validation of the entity, and also choose to expose an endpoint of Odate (REST) <a href="http://www.odata.org/">http://www.odata.org/</a>
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:b9af8fa5-21f5-4e26-86ba-0154e469655e" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/88x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/82.png" border="0" alt="" width="426" height="320" /></a></div>
9. For each method we add the decorator [invoke] in that way we generate the corresponding proxy methods in the client.

10) The Namespace and DomainServiceNameClass, give us a clue to find out the generated WCF service. The service address is:
1. http:// [host: port] / Services / [namespace] - [DomainServiceNameClass]. svc (the '.' are replaced by '–' )
2. Example: <a href="http://localhost:37390/Services/SoapRiaService-EmployeeDomainService.svc">http://localhost:37390/Services/SoapRiaService-EmployeeDomainService.svc</a>
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:9c321bbb-4377-49aa-a995-dc1e0d9e7a4e" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/98x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/92.png" border="0" alt="" width="425" height="257" /></a></div>
11. Now we need to add the endpoint to access the SOAP services, if we do not want a REST enpoint , just remove the OData endpoint.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:3723a062-a5a4-4fdb-9ceb-cf39faf242fd" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/9-a8x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/9-a.png" border="0" alt="" width="426" height="183" /></a></div>
12. Having all the logic and configuration ready in our application we need to make sure to add the library Microsoft.ServiceModel.DomainServices.Hosting.dll address shown below.
But you have this path, be sure to install or reinstall WCF Silverlight RIA Services V1.0 for 4 and Visual Studio 2010 <a href="http://www.microsoft.com/downloads/en/details.aspx?FamilyID=cd3191a1-def4-4caa-8120-1f0bbcf4bb05&amp;displaylang=en">http://www.microsoft.com/downloads/en/details.aspx?FamilyID=cd3191a1-def4-4caa-8120-1f0bbcf4bb05&amp;displaylang=en</a>
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:0d4fbbf5-ed61-44e7-82fa-329508bd7f7b" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/108x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/102.png" border="0" alt="" width="424" height="225" /></a></div>
13. After adding the library and compiled the project, we just need to try it (F5) to be host in the IIS test server and then we can see our service at the following address:

<a href="http://localhost:37390/Services/SoapRiaService-EmployeeDomainService.svc">http://localhost:37390/Services/SoapRiaService-EmployeeDomainService.svc</a>
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:b948e845-30f7-4313-ab8d-3029dcebc02f" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/118x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/112.png" border="0" alt="" width="425" height="272" /></a></div>
14. If we want to see the WSDL Contract <a href="http://localhost:37390/Services/SoapRiaService-EmployeeDomainService.svc?wsdl">http://localhost:37390/Services/SoapRiaService-EmployeeDomainService.svc?wsdl</a>
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:65d0b6b5-bf40-4931-b891-3e5e12bd28a7" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/128x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/122.png" border="0" alt="" width="425" height="289" /></a></div>
15. Now we’ve ready our service, to create our application in Windows Phone 7, first download the developer tools <a href="http://developer.windowsphone.com/">http://developer.windowsphone.com/</a> once installed create a new Windows project Panorama Phone Application (COOL!). use "WP7SoapRiaService" to name it, if you want.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:a7804c3b-881f-4b93-81e9-98568a3f78a5" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/138x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/132.png" border="0" alt="" width="425" height="251" /></a></div>
16. Once created the application we’ll need to add a service reference as an address setting <a href="http://localhost:37390/Services/SoapRiaService-EmployeeDomainService.svc">http://localhost:37390/Services/SoapRiaService-EmployeeDomainService.svc</a>, then click on GO,at this time you should now see our service and operations exposed by Ria Services, rename the namespace to "EmployeeService" for example.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:49fbba7a-9224-426f-b793-1e25f8c6498e" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/148x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/141.png" border="0" alt="" width="420" height="246" /></a></div>
17. Let see the proxy classes that were created after giving Ok double clicking on the service reference.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:1ca42a00-a2e8-4171-95e4-a40784bc4427" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/158x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/152.png" border="0" alt="" width="425" height="263" /></a></div>
18. Well now as a next step we could add the Reactive eXtensions(RX) reference ,
to control the asynchronous call to our Web Service, yap Reactive Extensions now comes within the Windows SDK Phone 7!!!.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:2ff74cd1-bd6a-4017-9a66-eafdfa9ecb27" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/168x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/162.png" border="0" alt="" width="425" height="266" /></a></div>
19. Once you add the reference add the following code, on my last post I’ve explained RX, and I just added a spinner property to control the visibility:
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:f975f42c-d97d-448f-b012-79edf87e9d0c" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/178x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/171.png" border="0" alt="" width="420" height="279" /></a></div>
20. Add a Spinner and bind it to the Visibility property, to control its visibility depending the data in our collection.
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:44ae961a-895b-47f9-9af9-6e379da4c847" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/188x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/181.png" border="0" alt="" width="425" height="246" /></a></div>
21. Run the application, and done you have the application ready!!
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:cabaac04-d37c-405e-838e-2e05b6c80215" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/198x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/19.png" border="0" alt="" width="211" height="335" /></a></div>
<div id="scid:8747F07C-CDE8-481f-B0DF-C6CFD074BF67:e5b5828a-8520-4ff6-b1e7-d91837f7176e" class="wlWriterEditableSmartContent" style="display:inline;float:none;margin:0;padding:0;"><a rel="thumbnail" href="http://mravinale.files.wordpress.com/2010/09/208x6.png"><img src="http://mravinale.files.wordpress.com/2010/09/20.png" border="0" alt="" width="207" height="335" /></a></div>
Note:
The first time when you execute this test you should be patience, because it could take a little longer time until you get the information from the server.

I Hope this post could be useful :D

<a title="Project Download" href="http://riaserviceswp7comm.codeplex.com/">Project Download.</a>

May the Code be with you!!!

<a rel="license" href="http://creativecommons.org/licenses/by/3.0/"><img style="border-width:0;" src="http://creativecommons.org/images/public/somerights20.png" alt="Creative Commons License" /></a>
This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a>

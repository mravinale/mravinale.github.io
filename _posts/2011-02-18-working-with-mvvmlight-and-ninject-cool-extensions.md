---
layout: post
title: Working with MvvmLight and Ninject cool extensions
date: 2011-02-18 20:15
author: mravinale
comments: true
categories: [Silverlight]
---
<p>Today I would like to share some cool stuff that you can do with MvvmLight and Ninject extensions,</p>  <p>Other objective of this article is to show how to implement in a very quick way(easy steps) the <a href="http://en.wikipedia.org/wiki/Aspect-oriented_programming" target="_blank">AOP</a> IANPC Interception (actions to be executed before or after the execution of <strong>every</strong> call to a method in our code) provided by <a href="http://ninject.org/" target="_blank">Ninject</a> <a href="http://ninject.org/extensions" target="_blank">Extensions</a>(<a href="http://en.wikipedia.org/wiki/Dependency_injection" target="_blank">DI/Ioc</a>) applied in the implementation of the interface INotifyPropertyChange as a simple atribute.</p>  <p>in other words :</p>  <p>We are going to change this code:</p>  <pre class="code"><span style="color:blue;">      public class </span><span style="color:#2b91af;">SomeViewModel </span>: <span style="color:#2b91af;">ViewModelBase </span>{

           <span style="color:blue;">private </span><span style="color:#2b91af;">ObservableCollection</span>&lt;<span style="color:#2b91af;">Item</span>&gt; _listCollection;
           <span style="color:blue;">public </span><span style="color:#2b91af;">ObservableCollection</span>&lt;<span style="color:#2b91af;">Item</span>&gt; ListCollection {
               <span style="color:blue;">get </span>{
                   <span style="color:blue;">return </span>_listCollection;
               }
               <span style="color:blue;">set </span>{
                   _listCollection = <span style="color:blue;">value</span>;
                   OnPropertyChanged(<span style="color:#a31515;">&quot;ListCollection&quot;</span>);
               }
           }
       }</pre>

<p><a href="http://11011.net/software/vspaste"></a><a href="http://11011.net/software/vspaste"></a><a href="http://11011.net/software/vspaste"></a></p>

<p>To this code:</p>

<pre class="code"><span style="color:blue;">       public class </span><span style="color:#2b91af;">SomeViewModel </span>: <span style="color:#2b91af;">AutoNotifyViewModelBase </span>{

           [<span style="color:#2b91af;">NotifyOfChanges</span>]
           <span style="color:blue;">public virtual </span><span style="color:#2b91af;">ObservableCollection</span>&lt;<span style="color:#2b91af;">Item</span>&gt; ListCollection { <span style="color:blue;">get</span>; <span style="color:blue;">set</span>; }</pre>

<p>
  <br /></p>

<pre class="code">       }</pre>

<p>A little of history and motivation first:</p>

<p>I’ve been doing some experiments with AOP, and then I saw a really nice <a href="http://www.codeproject.com/KB/library/Aspects.aspx" target="_blank">article in code project</a> about using <a href="http://sachabarber.net/?p=849" target="_blank">Aspects applied to INotifyPropertyChanged</a><em> </em>(INPC), but I didn’t see a Ninject implementation so as always I’ve started to google about it.</p>

<p>What I saw was a gret <a href="http://jonas.follesoe.no/2010/01/17/automatic-inpc-using-dynamic-proxy-and-ninject/" target="_blank">article</a> of <a href="http://twitter.com/follesoe" target="_blank">Jonas Follesoe</a> talking about using dynamic proxy for automatic INPC with Ninject I really recomend to read that post and download the project.</p>

<p>Reading article you can see that <a href="http://twitter.com/innovatian" target="_blank">Ian Davis</a>, the owner of interseption-extension for Ninject, contacted Jonas, and then he made his own adaptation of automatic INPC(IANPC) another <a href="http://innovatian.com/2010/01/ninject-extensions-interception-and-iautonotifypropertychanged-ianpc/" target="_blank">great article</a></p>

<p>After couple week I had some free time and I wanted to see the source, of Ian davis project, and realised that the code of the IANPC was already inside, so I’ve started to make it work for share with you.</p>

<p><a href="http://mravinale.files.wordpress.com/2011/02/image.png"></a></p>

<p>Let’s start to see the implementation:</p>

<p>First step is download the assemblies that we need:</p>

<p>Get Silverlight <a href="https://github.com/downloads/ninject/ninject.extensions.interception/Ninject.Extensions.Interception-2.2.0.0-release-silverlight-4.0.zip" target="_blank">Ninject asemblies</a> from <a href="https://github.com/ninject/ninject.extensions.interception/downloads" target="_blank">GitHub</a>.</p>

<p>Get <a href="http://commonservicelocator.codeplex.com/releases/view/17694#DownloadId=54958" target="_blank">Common Service locator</a> from <a href="http://commonservicelocator.codeplex.com/" target="_blank">codeplex</a>.</p>

<p>Ones that we finished to add all references to the project, let’s get to work!</p>

<p>In this example we are going to fill a List(<a href="http://mravinale.wordpress.com/2010/11/14/flick-behavior-using-a-sl-listbox-control-as-you-do-on-wp7/" target="_blank">Flick Behavior</a>) with some mock items and add some effects.</p>

<p>Add a new Class named IocContainer.cs</p>

<p style="text-align:center;"><a href="http://mravinale.files.wordpress.com/2011/02/image1.png"><img style="display:inline;" class="aligncenter" title="image" border="0" alt="image" src="http://mravinale.files.wordpress.com/2011/02/image_thumb1.png" width="335" height="222" /></a></p>

<p>Add the binding for each instance that you need in your project, as an example I’ve made some bindings for the main instance that I need to use:</p>

<pre class="code"><span style="color:blue;">public class </span><span style="color:#2b91af;">IocContainer </span>: <span style="color:#2b91af;">NinjectModule </span>{

       <span style="color:blue;">public override void </span>Load() {

           <span style="color:blue;">this</span>.Bind&lt;<span style="color:#2b91af;">ISampleDataSource</span>&gt;().To&lt;<span style="color:#2b91af;">SampleDataSource</span>&gt;();   

           <span style="color:green;">//Services
           </span><span style="color:blue;">this</span>.Bind&lt;<span style="color:#2b91af;">IDataService</span>&gt;().To&lt;<span style="color:#2b91af;">MockItemsService</span>&gt;();            

           <span style="color:green;">//Pages
           </span><span style="color:blue;">this</span>.Bind&lt;<span style="color:#2b91af;">MainPage</span>&gt;().ToSelf();

           <span style="color:green;">//ViewModels
           </span><span style="color:blue;">this</span>.Bind&lt;<span style="color:#2b91af;">MainViewModel</span>&gt;().ToSelf();

       }
   }</pre>

<p><a href="http://11011.net/software/vspaste"></a></p>

<p><a href="http://11011.net/software/vspaste"></a></p>

<p>Add the Ioc Initialization into the app, Adding The Bootstrapper</p>

<pre class="code"><span style="color:blue;">    public class </span><span style="color:#2b91af;">Bootstrapper
    </span>{
        <span style="color:blue;">private static </span><span style="color:#2b91af;">IKernel </span>Container { <span style="color:blue;">get</span>; <span style="color:blue;">set</span>; }

        <span style="color:blue;">public static void </span>Initialize()
        {
            Container = <span style="color:blue;">new </span><span style="color:#2b91af;">StandardKernel</span>(<span style="color:blue;">new </span><span style="color:#2b91af;">DynamicProxy2Module</span>(), <span style="color:blue;">new </span><span style="color:#2b91af;">IocContainer</span>());
            <span style="color:#2b91af;">ServiceLocator</span>.SetLocatorProvider(() =&gt; <span style="color:blue;">new </span><span style="color:#2b91af;">NinjectServiceLocator</span>(Container));
        }

        <span style="color:blue;">public static void </span>ShutDown()
        {
            Container.Dispose();
        }
    }</pre>

<p><a href="http://11011.net/software/vspaste"></a></p>

<p>The <a href="http://www.timmykokke.com/2010/12/dependency-injection-mvvm-ninject-and-silverlight/" target="_blank">normal implementation</a> of this would be something like this:</p>

<pre class="code"><span style="color:blue;">var </span>kernel = <span style="color:blue;">new </span><span style="color:#2b91af;">StandardKernel</span>(<span style="color:blue;">new </span><span style="color:#2b91af;">IocContainer</span>());</pre>

<p>But we need to create proxies in order to do the Interception to our virtual members, that’s way we’ve added the new instance of <a href="http://www.castleproject.org/dynamicproxy/index.html" target="_blank">DynamicProxy2Module</a>, and way we need the reference to Castle.Core assembly.</p>

<p>Let’s do the last and important part of IANPC , the atribute NotifyOfChanges :)</p>

<p>[<span style="color:#2b91af;">NotifyOfChanges</span>]</p>

<p><span style="color:blue;">public virtual</span><span style="color:#2b91af;">ObservableCollection</span>&lt;<span style="color:#2b91af;">Item</span>&gt; ListCollection { <span style="color:blue;">get</span>; <span style="color:blue;">set</span>; }</p>

<p>But we need to implement the Interface IAutoNotifyPropertyChanged from Ninject.Extensions.Interseption in our view model otherwise it won’t work, but in MvvmLight we have a ViewModelBase with a diferent contract implementation instead to use OnPropertyChanged MvvmLight uses RaisePropertyChanged shit!!, I’ve been forced to create a new class, AutoNotifyViewModelBase that implements IAutoNotifyPropertyChanged and mvvm methods.</p>

<p>Implement the interface IAutoNotifyPropertyChanged in this case is already implmented in AutoNotifyViewModelBase .</p>

<p><span style="color:blue;">public class </span><span style="color:#2b91af;">MainViewModel </span>: <span style="color:#2b91af;">AutoNotifyViewModelBase </span>{</p>

<p><a href="http://11011.net/software/vspaste"></a></p>

<pre class="code">      [<span style="color:#2b91af;">NotifyOfChanges</span>]
       <span style="color:blue;">public virtual </span><span style="color:#2b91af;">ObservableCollection</span>&lt;<span style="color:#2b91af;">Item</span>&gt; ListCollection { <span style="color:blue;">get</span>; <span style="color:blue;">set</span>; }</pre>

<p>}</p>

<p>So far we have develop all things required to retrieve data from a service and fill our List Collection, we have the infrastructure using Ninject Ioc, and the interception IANPC Atributes cool right.</p>

<p>May the code be with you!</p>

<p>Mariano Ravinale.</p>

<p><a href="http://creativecommons.org/licenses/by/3.0/" rel="license"><img style="border-width:0;" alt="Creative Commons License" src="http://creativecommons.org/images/public/somerights20.png" /></a></p>

<p>This work is licensed under a <a href="http://creativecommons.org/licenses/by/3.0/" rel="license">Creative Commons Attribution 3.0 Unported License</a></p>

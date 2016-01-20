---
layout: post
title: Asp.net MVC Simple CQRS part 1 – Query
date: 2012-06-20 17:48
author: mravinale
comments: true
categories: [Asp.Net MVC]
header-img: "img/post-bg-06.jpg"
---
Inspired by the Jeremy blog post <a href="http://csharperimage.jeremylikness.com/2012/01/crud-its-now-cqrs-or-is-it.html" title="Jeremy blog post" target="_blank"> crud its now cqrs or is it</a> and
Gregory Young simple CQRS project: <a href="https://github.com/gregoryyoung/m-r">https://github.com/gregoryyoung/m-r</a>

I’ve been working in a very simple insight for Asp.net MVC, in order to work comfortably in our business applications.

Quoting some words of Jeremy Likness: “CQRS provides what I think is a very valuable insight: that how you read and query data is probably very different from how you manage, update, and manipulate data. You don’t have to have a picture-perfect implementation of CQRS to take advantage of this concept”

<a href="http://mravinale.files.wordpress.com/2012/06/image.png"><img style="display:inline;border-width:0;" title="image" src="http://mravinale.files.wordpress.com/2012/06/image_thumb.png" alt="image" width="640" height="385" border="0" /></a>

The main Idea is to encapsulate our query and execute it using a processor, the processor wil be in charge to match each query to their handler in order to return the result from the data source.

Let's take a look inside the code:

<strong>Query</strong>: Will define an unique query action with an expected result

[code language="csharp"]
public interface IQuery { }
public class GetAlbumsByIdQuery : IQuery
{
    public int Id { get; set; }
}
[/code]

<strong>QueryProcessor: </strong>Will be in charge to match each query to their handler (using reflection and with a little help of dynamics) returning the handler's result from the datasource.

[code language="csharp"]
public class QueryProcessor : IProcessQuery
{
	private readonly IWindsorContainer container;
	public QueryProcessor(IWindsorContainer container)
    	{
        	this.container = container;
    	}
 	public TResult Execute(IQuery query)
 	{
		 dynamic handler = Assembly.GetExecutingAssembly()
		 	.GetTypes()
			 .Where(t =&gt;typeof(IHandleQuery)
			 .MakeGenericType(query.GetType(), typeof(TResult)).IsAssignableFrom(t)
			 	&amp;&amp; !t.IsAbstract
			 	&amp;&amp; !t.IsInterface)
			 .Select(i =&gt; container.Resolve(i)).Single();
		 return handler.Handle((dynamic)query);
 	}
}
[/code]

<strong>Abstract</strong> <strong>QueryHandler: </strong>This abstract class will implement the base signature for each handler that we will create, in order to have NHibernate Infrastructure ready for our queries.

[code language="csharp"]
public abstract class BaseQueryHandler : IHandleQuery where TQuery : IQuery
{
	public ISessionFactory SessionFactory { get; set; }
	protected ISession Session
	{
        	get { return SessionFactory.GetCurrentSession(); }
	}
	#region IQueryHandler Members
	public abstract TResult Handle(TQuery query);
	#endregion
}
[/code]

<strong>QueryHandler</strong>: Given the query and the returning signature we will execute the query and map the entity to our view Model using <a href="http://automapper.codeplex.com/">Automapper</a>

[code language="csharp"]
public class GetAlbumsByIdQueryHandler : BaseQueryHandler
{
	public override AlbumModel Handle(GetAlbumsByIdQuery query)
 	{
 		var entity = Session.Get(query.Id);
		return Mapper.Map(entity);
 	}
}
[/code]

<strong>Windsor Installer: </strong>

[code language="csharp"]
public class HandlersInstaller : IWindsorInstaller
{
	public void Install(IWindsorContainer container, IConfigurationStore store)
	{
      		var metadataProviderContributorsAssemblies = new[] { typeof (QueryProcessor).Assembly };
		container.Register(AllTypes.From(metadataProviderContributorsAssemblies
                                           .SelectMany(a =&gt; a.GetExportedTypes()))
                           .Where(t =&gt; t.Name.EndsWith(&quot;Handler&quot;) &amp;&amp; t.IsAbstract == false)
                           .Configure(x =&gt; x.LifestyleSingleton())
        	);
      		
                container.Register(Component.For&lt;IProcessQuery&gt;()
                         .ImplementedBy&lt;QueryProcessor&gt;().LifeStyle.Singleton);
                container.Register(Component.For&lt;IProcessCommand&gt;()
                         .ImplementedBy&lt;CommandProcessor&gt;().LifeStyle.Singleton);

    	}
}
[/code]

<strong>Asp.Net MVC Controller's example: </strong>

[code language="csharp"]
public class AlbumsController : Controller
{
	public IProcessQuery QueryProcessor { get; set; }
	public ActionResult Index() { return View(); }

	[HttpGet]
	public ActionResult Details(GetAlbumsByIdQuery query)
	{
	      return PartialView(&quot;Details&quot;, QueryProcessor.Execute(query));
    	}
}
[/code]

Now executing our query we'll get the result in order to fill our view.

In case you are wondering about how to Handle NHibernate Session in Asp.net Mvc, I would recommend take a look to this post:

<a href="http://nhforge.org/blogs/nhibernate/archive/2011/03/03/effective-nhibernate-session-management-for-web-apps.aspx">http://nhforge.org/blogs/nhibernate/archive/2011/03/03/effective-nhibernate-session-management-for-web-apps.aspx</a>

And of course  read about mapping by code:

<a href="http://fabiomaulo.blogspot.com.ar/2011/04/nhibernate-32-mapping-by-code.html">http://fabiomaulo.blogspot.com.ar/2011/04/nhibernate-32-mapping-by-code.html</a>

Next post I'll show the view, and command handling.

Download or Fork the project on <a href="https://github.com/mravinale/Cronos">GitHub </a>

May the code be with you!.

Mariano Ravinale

<a href="http://creativecommons.org/licenses/by/3.0/" rel="license"><img src="http://creativecommons.org/images/public/somerights20.png" alt="Creative Commons License" /></a>

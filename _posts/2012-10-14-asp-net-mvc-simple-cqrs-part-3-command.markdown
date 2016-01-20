---
layout: post
title: Asp.net MVC Simple CQRS part 3 – Command
date: 2012-10-14 21:05
author: mravinale
comments: true
categories: [Asp.Net MVC, command processor, CQRS]
---
In this part we need to implement a mechanism that will change the state of our domain, this mechanism will be the <a href="http://c2.com/cgi/wiki?CommandMessagePattern"><em>CommandMessage</em></a>.

this <em>Command </em>should delivery a message that will change the state of our domain, and our view should reflect that change.

after each commit or domain change, we should trigger a <em>view </em>update, for that reason I thaught that <a href="http://signalr.net/">SignalR </a>would be a good fit.

<strong>Example:</strong>
[youtube=http://youtu.be/E06pejXe0_I]


<strong>Command</strong>
[code language="csharp"]
public interface ICommand { }

public class DeleteAlbumCommand : ICommand
{
	public int Id { get; set; }
}
[/code]
This <em>command </em>is an action that will be triggered by the user in order to delete an Album

<strong>CommandProcessor</strong>
[code language="csharp"]
public class CommandProcessor : IProcessCommand
{
	private readonly IWindsorContainer container;
	public CommandProcessor(IWindsorContainer container)
	{
		this.container = container;
	}	
	public void Execute(ICommand command)
	{
		dynamic handler = Assembly.GetExecutingAssembly()
					.GetTypes()									
					.Where(t =&gt;typeof(IHandleCommand&lt;&gt;)
					.MakeGenericType(command.GetType()).IsAssignableFrom(t)
					 &amp;&amp; !t.IsAbstract
					 &amp;&amp; !t.IsInterface)	
					.Select(i =&gt; container.Resolve(i)).Single();

		handler.Handle((dynamic)command);
	}
}
[/code]
Each command needs to be sent to the handler, for that we need a routing mechanism as well,
and that should be the resposability of the <em>command processor</em>.

<strong>Abstract</strong> <strong>CommandHandler: </strong>
[code language="csharp"]
public abstract class BaseCommandHandler&lt;TCommand&gt; : IHandleCommand&lt;TCommand&gt; where TCommand : ICommand
{
	public ISessionFactory SessionFactory { get; set; }
	public IConnectionManager ConnectionManager { get; set; }

	protected ISession Session
	{
		get { return SessionFactory.GetCurrentSession(); }
	}

	#region Implementation of IHandleCommand&lt;TCommand&gt;
		public abstract void Handle(TCommand command);
	#endregion
}
[/code]
The <em>BaseCommandHandler </em>will provide the basic infrastructure to handle any business logic
these handlers will cause a change of the states of our domain, each command is particular action
for that will cause a change and after this change we need to alert or syncronize all clients.


<strong>CommandHandler</strong>
[code language="csharp"]

public class DeleteAlbumCommandHandler : BaseCommandHandler&lt;DeleteAlbumCommand&gt; 
{
	public override void Handle(DeleteAlbumCommand command)
	{
		var entity = Session.Get&lt;Core.Album&gt;(command.Id);
		Session.Delete(entity);

		//client side method invocation (update the clients!)
		ConnectionManager.GetClients&lt;AlbumsHub&gt;().Redraw();
	}
}
[/code]

The mechanism and resposability for synchronize all domain changes will be asigned to SignalR, after each change
SignalR will trigger the redraw event of the datatable plugin, this event will update all the data that is
being displaying in that moment. 


<strong>Handlers Installer: </strong>

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
An example of the registration of our components using Windsor Castle Container.

<strong>MvcInstaller: </strong>
[code language="csharp"]
 public class MvcInstaller : IWindsorInstaller
{
	public void Install(IWindsorContainer container, IConfigurationStore store)
	{
		container.Register(
			AllTypes.FromAssemblyContaining&lt;HomeController&gt;()
				.BasedOn&lt;IController&gt;()
				.WithService.Self()
				.Configure(cfg =&gt; cfg.LifestyleTransient()),
				
			Component.For&lt;IControllerFactory&gt;().ImplementedBy&lt;WindsorControllerFactory&gt;(),
			Component.For&lt;IConnectionManager&gt;().Instance(AspNetHost.DependencyResolver.Resolve&lt;IConnectionManager&gt;())
		
			);

		DependencyResolver.SetResolver(new WindsorDependencyResolver(container));
	}
}
[/code]
Remeber that be need to register the SignalR components here you have an example of how 
to register the IConectionManager.

<strong>Asp.Net MVC Controller's example: </strong>

[code language="csharp"]
public class AlbumsController : Controller
{
	public IProcessCommand CommandProcessor { get; set; }
	public ActionResult Index() { return View(); }

	[HttpPost]
	public void Delete(DeleteAlbumCommand command)
	{
		CommandProcessor.Execute(command);
	}
}
[/code]
In our Controller we need to inject the command Processor, and execute
the delete command that contains the id of the album the we need to delete.


<strong>SignalR in Album.js: </strong>
[code language="javascript"]
var Albums = (function ($) {
    var public = {};

    public.init = function (param) {
        initSignalR();
        initLayoutResources();
    };

    var initLayoutResources = function () {      
        $(&quot;.link&quot;).button();       

        $(&quot;#dialog&quot;).dialog({ autoOpen: false, modal: true });

        $(&quot;.edit-link&quot;).createForm({ baseUrl: &quot;Albums/Edit/&quot; });
        $(&quot;.details-link&quot;).createForm({ baseUrl: &quot;Albums/Details/&quot; });
        $(&quot;.insert-link&quot;).createForm({ baseUrl: &quot;Albums/Insert/&quot; });
        $(&quot;.delete-link&quot;).deleteAction({ baseUrl: &quot;Albums/Delete/&quot; });
    };

    var initSignalR = function () {
        //Create SignalR object to get communicate with server
        var hub = $.connection.AlbumsHub;

        hub.Redraw = function () {
            DataTable.reload();
        };
        // Start the connection
        $.connection.hub.start();
    };

  return public;

} (jQuery));
[/code]
Remember that we need to update the state of the clients after each command action that
triggered a domain change, for that we should initialize the SignalR comunication with the server and
declare the redraw function that will call the jQuery Datatable reload method in order to update all the data.

Download or Fork the project on <a href="https://github.com/mravinale/Cronos">GitHub </a>

May the code be with you!.

Mariano Ravinale

<a href="http://creativecommons.org/licenses/by/3.0/" rel="license"><img src="http://creativecommons.org/images/public/somerights20.png" alt="Creative Commons License" /></a>

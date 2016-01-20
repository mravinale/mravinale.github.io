---
layout: post
title: Reactive Extensions en Silverlight(RX)
date: 2010-08-16 04:45
author: mravinale
comments: true
categories: [Silverlight, Windows Phone 7]
---
Si bien, la idea inicial era ir haciendo los post de forma incremental y que evolucione desde los patrones de diseño, hacia patrones mas avanzados como mvc, mvp hasta llegar a mvvm y empezar con Silverlight,  crei que es importante hablar de Rx ya que nos ayudara enormemente en Silverlight, y como estamos en el browser desde el lado cliente, seguramente habra muchas llamadas asyncronicas, es ahi donde Rx entra en juego.

podriamos definir a Rx(Reactive extensions) como una libreria para aplicaciones que necesiten hacer llamadas asincronicas y orientadas a eventos utilizando colecciones del tipo Observable.

En la pagina principal explica un poco el significado de esta nueva herramienta, que nos ayudará mucho a controlar y orquestar programaticamente estos eventos y llamadas.

Pagina principal, introduccion y Video Chanel 9.

<a href="http://msdn.microsoft.com/en-us/devlabs/ee794896.aspx">http://msdn.microsoft.com/en-us/devlabs/ee794896.aspx</a>

Ejemplos:

<a href="http://rxwiki.wikidot.com/101samples">http://rxwiki.wikidot.com/101samples</a>

Bien si se fijan en la pagina principal dice:

"Rx is a superset of the standard LINQ sequence operators that exposes asynchronous and event-based computations as push-based, observable collections via the new .NET 4.0 interfaces IObservable&lt;T&gt; and IObserver&lt;T&gt;.  These are the mathematical dual of the familiar IEnumerable&lt;T&gt; and IEnumerator&lt;T&gt; interfaces for pull-based, enumerable collections in the .NET Framework."

Creo que este parrafo explica rapidamente de que se trata Rx. A mi entender lo que trata de expresar es que Rx es un conjunto de operaciones que utiliza los eventos a los que se suscribe como punto de entrada para llenar collecciones del tipo observable para que de esta manera nos avise cuando podemos empezar a consumir los datos como si estos estuvieran en una collecion o dispare el consumo de estos de forma automatica.

Pero volviendo a Silverlight podemos encontrarle gran utilidad si utilizamos el patron mvvm, ya que tenemos un binding entre la vista y el ViewModel por lo tanto nos ayudara a simplificar y automatizar el proceso de llamado al  servidor en busqueda de datos, por ejemplo, que es mas o menos lo que esta en el codigo de abajo.

En este caso lo que hice es crear una clase con patron Singleton como Clase y utilizarla como Modelo , para poderla reutilizar, y una llamada a un servicio para poder obtener datos,  en este caso,consumiremos los nombres de unas personas.

[sourcecode language="csharp"]

public class ServicePeopleModel{

        private static volatile ServicePeopleModel instance;
        private static object keyLock = new Object();

        public static ServicePeopleModel CurrentInstance {
            get {
                if (instance == null) {
                    lock (keyLock) {
                        instance = new ServicePeopleModel();
                    }
                }
                return instance;
            }
        }

        /// &lt;summary&gt;
        /// Gets the remote people data.
        /// &lt;/summary&gt;
        /// &lt;returns&gt;&lt;/returns&gt;
        public IObservable&lt;Person&gt; GetPeople() {

            SLPeopleServiceClient client = new SLPeopleServiceClient();

            var observable =
            
             //first we are going to listen the event, 
             //when is ready to fill our collection
             (from response in Observable.FromEvent
             &lt;GetPeopleCompletedEventArgs&gt;(client, &quot;GetPeopleCompleted&quot;)
            
            //when the data arrives and the collection is ready,
            //I need to transform this data.
             from people in ConvertItems
             (response.EventArgs.Result.ToList&lt;String&gt;()).ToObservable()
            
            //and when it is ready return observable
            //collection, -and stay tuned
            //when all the operations
            //are done, because i'm gonna call you, 
            //says the observable collection.
            select people).ObserveOnDispatcher();
            
            //let's call the service
            client.GetPeopleAsync();

            return observable;

        }

        /// &lt;summary&gt;
        /// Converts the items.
        /// &lt;/summary&gt;
        public IEnumerable&lt;Person&gt; ConvertItems(List&lt;String&gt; response){
            var people = new List&lt;Person&gt;();
            response.ForEach(person =&gt; people.Add(
                new Person() {Name = person})
            );
            return people;
        }

    }
public class Person {

    public string Name { get; set; }

}
[/sourcecode]

Ok, ya hemos visto como hariamos nuestro modelo para pedir los datos a nuestro servicio, transformar los datosy devolver una coleccion con todos los datos, listos para utilizar.
Pero nos falta bindear estos datos contra la vista, para automatizar la tarea, de forma que cuando vengan los datos la vista lo muestre estos datos sin que tengamos que intervenir.

[sourcecode language="csharp"]
//On The PeopleViewModel

public class PeopleViewModel: ViewModelBase {

    /// &lt;summary&gt;
    /// Initializes a new instance of the PeopleViewModelclass.
    /// &lt;/summary&gt;
    public PeopleViewModel() {
         
        var client = Observable.Empty&lt;Person&gt;();
                       
        //we've suscribed to a changed event and we 
        //want to select the first person to show it on the view
        PeopleCollection.CollectionChanged += (s, e) =&gt; {
           SelectedPersonProperty = PeopleCollection[0].Name;
        };

        //let's the magic starts
        client = ServicePeopleModel.CurrentInstance.GetPeople();

        //and suscribe us to the service result
        client.Subscribe((item) =&gt; {
           PeopleCollection.Add(item);
        });
    }

     /// &lt;summary&gt;
     /// Gets or sets SelectedPersonProperty 
     /// &lt;/summary&gt;
     public string SelectedPersonProperty {
         get {
             return _selectedPersonProperty ;
         }
         set {
             if (_selectedPersonProperty == value) return;
                 _selectedPersonProperty = value;
             RaisePropertyChanged(&quot;SelectedPersonProperty &quot;);
         }
     }
     
       /// &lt;summary&gt;
        /// Gets or sets the PeopleCollection.
        /// &lt;/summary&gt;
        /// &lt;value&gt;PeopleCollection.&lt;/value&gt;
        public ObservableCollection&lt;Person&gt; PeopleCollection{
            get {
                if(_popleCollection == null){
                    _peopleCollection = new ObservableCollection&lt;Person&gt;();
                }
                return _peopleCollection;
            }
            set {
                if (_peopleCollection == value) return;
                _peopleCollection= value;
                RaisePropertyChanged(&quot;PeopleCollection&quot;);
            }
        }
}
[/sourcecode]

Sin embargo si ven la pagina de ejemplos podran ver mushisimas mas aplicaciones de este framework, que a mi parecer es excelente tratando de unir el observer pattern y el Iterator, de forma sencilla y con altos beneficios en cantidad y calidad de codigo.

tengo otra aplicacion parecida hecha con Rx que posteare mas adelante, creo que vale la pena bajarselo y probarlo e irle toamndo la mano, creo que despues se va volviendo natural y uno le comienza a ver mas aplicaciones.

Dejo algunos links:

<a href="http://www.silverlightshow.net/items/Using-Reactive-Extensions-in-Silverlight.aspx">http://www.silverlightshow.net/items/Using-Reactive-Extensions-in-Silverlight.aspx</a>

<a href="http://blogs.msdn.com/b/somasegar/archive/2009/11/18/reactive-extensions-for-net-rx.aspx">http://blogs.msdn.com/b/somasegar/archive/2009/11/18/reactive-extensions-for-net-rx.aspx</a>

Para los amantes y expertos javascript (<a href="http://twitter.com/cortezcristian">@cortezcristian</a>) ;)

<a href="http://codebetter.com/blogs/matthew.podwysocki/archive/2010/02/16/introduction-to-the-reactive-extensions-to-javascript.aspx">http://codebetter.com/blogs/matthew.podwysocki/archive/2010/02/16/introduction-to-the-reactive-extensions-to-javascript.aspx</a>

</BR>
</BR>
<a rel="license" href="http://creativecommons.org/licenses/by/3.0/"><img alt="Creative Commons License" style="border-width:0;" src="http://creativecommons.org/images/public/somerights20.png" /></a><br />This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a>.

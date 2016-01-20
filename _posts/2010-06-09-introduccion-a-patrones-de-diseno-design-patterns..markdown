---
layout: post
title: Introduccion a patrones de diseño (design patterns)
date: 2010-06-09 05:05
author: mravinale
comments: true
categories: [Diseño y arquitectura de software]
---
Hoy quisiera compartir un resumen de los patrones de diseño mas conocidos y publicados en el libro "<a href="http://www.amazon.com/gp/product/0201633612">Design Patterns</a>" escrito por como se los conoce los "El grupo de los cuatro" (Gof) una guia para tener siempre a mano para el diseño y desarrollo de software.

Ahora bien, que son patrones de diseño?

“Los patrones de diseño son el esqueleto de las soluciones a problemas comunes en el desarrollo de software.”

Nos brindan una solución ya probada y documentada a problemas de desarrollo de software que están sujetos a contextos similares, para esto debemos tener presente los siguientes elementos de un patrón: su nombre, el problema , la solución y las consecuencias .

Entender un poco la aplicacion de estos patrones nos ayudará mas adelante a realizar implementaciones de patrones mas complejos como el MVC o MVP (que veremos mas adelante).

Ademas todo esto viene a colacion de encaminar los post hacia microarquitecturas utilizadas en las aplicaciones RIA ya sea Flash, Flex o Silverlight, pero tiempo al tiempo.

<span style="color:#c0c0c0;"><span style="text-decoration:underline;">Ejemplo "Patron singleton" :</span></span>

[sourcecode language="csharp"]
public sealed class Singleton {

    private static volatile Singleton instance;
    private static object syncRoot = new Object();

    public static Singleton Instance {
        get {
            if (instance == null) {
                lock(syncRoot) {
                        instance = new Singleton();
                }
           }
           return instance;
       }
   }
}
[/sourcecode]

<span style="color:#333399;"><strong><span style="color:#c0c0c0;"><span style="text-decoration:underline;">Patrones de creación</span></span></strong></span>
<ul>
	<li><strong><span style="color:#ff9900;">Abstract Factory</span></strong><span style="color:#ff9900;">.</span> Proporciona una interfaz para crear familias de objetos o que dependen entre sí, sin especificar sus clases concretas.</li>
	<li><span style="color:#ff9900;"><strong>Builder</strong></span>. Separa la construcción de un objeto complejo de su representación, de forma que el mismo proceso de construcción pueda crear diferentes representaciones.</li>
	<li><strong><span style="color:#ff9900;">Factory Method</span></strong>. Define una interfaz para crear un objeto, pero deja que sean las subclases quienes decidan qué clase instanciar. Permite que una clase delegue en sus subclases la creación de objetos.</li>
	<li><strong><span style="color:#ff9900;">Prototype</span></strong>. Especifica los tipos de objetos a crear por medio de una instancia prototípica, y crear nuevos objetos copiando este prototipo.</li>
	<li><strong><span style="color:#ff9900;">Singleton</span></strong>. Garantiza que una clase sólo tenga una instancia, y proporciona un punto de acceso global a ella.</li>
</ul>
<span style="color:#333399;"><strong><span style="color:#c0c0c0;"><span style="text-decoration:underline;">Patrones estructurales</span></span></strong></span>
<ul>
	<li><strong><span style="color:#ff9900;">Adapter</span></strong>. Convierte la interfaz de una clase en otra distinta que es la que esperan los clientes. Permiten que cooperen clases que de otra manera no podrían por tener interfaces incompatibles.</li>
	<li><strong><span style="color:#ff9900;">Bridge</span></strong>. Desvincula una abstracción de su implementación, de manera que ambas puedan variar de forma independiente.</li>
	<li><strong><span style="color:#ff9900;">Composite</span></strong>. Combina objetos en estructuras de árbol para representar jerarquías de parte-todo. Permite que los clientes traten de manera uniforme a los objetos individuales y a los compuestos.</li>
	<li><strong><span style="color:#ff9900;">Decorator</span></strong>. Añade dinámicamente nuevas responsabilidades a un objeto, proporcionando una alternativa flexible a la herencia para extender la funcionalidad.</li>
	<li><strong><span style="color:#ff9900;">Facade</span></strong>. Proporciona una interfaz unificada para un conjunto de interfaces de un subsistema. Define una interfaz de alto nivel que hace que el subsistema se más fácil de usar.</li>
	<li><strong><span style="color:#ff9900;">Flyweight</span></strong>. Usa el compartimiento para permitir un gran número de objetos de grano fino de forma eficiente.</li>
	<li><strong><span style="color:#ff9900;">Proxy</span></strong>. Proporciona un sustituto o representante de otro objeto para controlar el acceso a éste.</li>
</ul>
<span style="color:#333399;"><strong><span style="color:#c0c0c0;"><span style="text-decoration:underline;">Patrones de comportamiento</span></span></strong></span>
<ul>
	<li><strong><span style="color:#ff9900;">Chain of Responsibility</span></strong>. Evita acoplar el emisor de una petición a su receptor, al dar a más de un objeto la posibilidad de responder a la petición. Crea una cadena con los objetos receptores y pasa la petición a través de la cadena hasta que esta sea tratada por algún objeto.</li>
	<li><strong><span style="color:#ff9900;">Command</span></strong>. Encapsula una petición en un objeto, permitiendo así parametrizar a los clientes con distintas peticiones, encolar o llevar un registro de las peticiones y poder deshacer la operaciones.</li>
	<li><strong><span style="color:#ff9900;">Interpreter</span></strong>. Dado un lenguaje, define una representación de su gramática junto con un intérprete que usa dicha representación para interpretar las sentencias del lenguaje.</li>
	<li><strong><span style="color:#ff9900;">Iterator</span></strong>. Proporciona un modo de acceder secuencialmente a los elementos de un objeto agregado sin exponer su representación interna.</li>
	<li><strong><span style="color:#ff9900;">Mediator</span></strong>. Define un objeto que encapsula cómo interactúan un conjunto de objetos. Promueve un bajo acoplamiento al evitar que los objetos se refieran unos a otros explícitamente, y permite variar la interacción entre ellos de forma independiente.</li>
	<li><strong><span style="color:#ff9900;">Memento</span></strong>. Representa y externaliza el estado interno de un objeto sin violar la encapsulación, de forma que éste puede volver a dicho estado más tarde.</li>
	<li><strong><span style="color:#ff9900;">Observer</span></strong>. Define una dependencia de uno-a-muchos entre objetos, de forma que cuando un objeto cambia de estado se notifica y actualizan automáticamente todos los objetos.</li>
	<li><strong><span style="color:#ff9900;">State</span></strong>. Permite que un objeto modifique su comportamiento cada vez que cambia su estado interno. Parecerá que cambia la clase del objeto.</li>
	<li><strong><span style="color:#ff9900;">Strategy</span></strong>. Define una familia de algoritmos, encapsula uno de ellos y los hace intercambiables. Permite que un algoritmo varíe independientemente de los clientes que lo usan.</li>
	<li><strong><span style="color:#ff9900;">Template Method</span></strong>. Define en una operación el esqueleto de un algoritmo, delegando en las subclases algunos de sus pasos. Permite que las subclases redefinan ciertos pasos del algoritmo sin cambiar su estructura.</li>
	<li><strong><span style="color:#ff9900;">Visitor</span></strong>. Representa una operación sobre los elementos de una estructura de objetos. Permite definir una nueva operación sin cambiar las clases de los elementos sobre los que opera.</li>
</ul>
Dejo el link para que lo descarguen una solucion en VS2010 con todos los patrones en C# para ir probando :
<a href="http://cid-86b4ae59157bdbba.office.live.com/self.aspx/dotNet%20Project/GOFPatterns.rar">GoFPaternsC#</a>

Happy Coding!!!

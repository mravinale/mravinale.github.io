---
layout: post
title: Un poco de JsTree
date: 2010-03-02 02:03
author: mravinale
comments: true
categories: [Jquery]
---
Quizas esta sea la primera de una serie de post sobre mi experiencia con JsTree, ya veremos.

En un momento me toco armar un control que utilizara un dual list, pero con una variante el cada list tendria la estructura de arbol!, para eso tomé el maravilloso <a href="http://www.jstree.com/">jsTree</a>.
y empecé a armar un custom server control en asp.net utilizando este componente como estandarte.

Si bien este control tiene la posibilidad de drag and drop, se preguntarán por que hacer botones para pasar
los nodos de un lado a otro, pues la respuesta es simple, la gente esta acostumbrada a los botones.

Imagen del control:
<pre><a href="http://mravinale.files.wordpress.com/2010/03/jstree.png"><img class="alignnone size-medium wp-image-50" title="My JsTree .Net" src="http://mravinale.files.wordpress.com/2010/03/jstree.png?w=300" alt="" width="300" height="110" /></a></pre>
Veamos un poco de los botones.

[sourcecode language="javascript"]
$(function() {
//nos suscribimos al evento click del boton
 $('#btnToRight').click(function() {
                //cortamos el elemento seleccionado del arbol de partida
                $.tree.reference(&quot;#divJsTree1&quot;).cut();
                //obtenemos el nodo padre de la ultima hoja del arbol de destino
                var last = $('#divJsTree2Root li:last-child .last').parent().parent();
                //lo seleccionamos
                var selected = $.tree.reference(&quot;#divJsTree2&quot;)
                                      .select_branch('#divJsTree2Root ' + '#' + $(last).attr('id'));
                //y pegamos el nodo de partida debajo del de destino.
                $.tree.reference(&quot;#divJsTree2&quot;).paste(selected, &quot;after&quot;);
});

//hacemos lo mismo pero de manera contraria para el boton con direccion opuesta.
$('#btnToLeft').click(function() {
                $.tree.reference(&quot;#divJsTree2&quot;).cut();
                var last = $('#divJsTree1Root li:last-child .last').parent().parent();

                var selected = $.tree.reference(&quot;#divJsTree1&quot;)
                                      .select_branch('#divJsTree1Root ' + '#' + $(last).attr('id'));

                $.tree.reference(&quot;#divJsTree1&quot;).paste(selected, &quot;after&quot;);
});
});
[/sourcecode]

Como el Jstree esta construido en su totalidad con Json.
Podriamos pensar que quizas serializando y deserealizando estos objetos podriamos reconstruirlo en el servidor creando su representacion como clases en C# utilizando <a href="http://msdn.microsoft.com/en-us/library/system.web.script.serialization.javascriptserializer.aspx">JavaScriptSerializer</a>.

mmm interesante, pero como obtengo la representacion Json del arbolito? y donde lo guardo para que el servidor lo pueda reconstruir?

quizas con este codigo se aclaren las dudas:

[sourcecode language="javascript"]
//creamos una funcion donde le enviamos como parametro el Id de nuestro arbolito y de nuestro hidden
function saveJsTree(treeName, hiddenName) {
           //el Id del arbolito es para obtener la referencia y utilizamos su funcion .get(&quot;json&quot;) para que nos devuelva su representacion Json.
           //$.toJson lo transformara a string para gurdarlo en el hidden. 
           var jsonTree = $.toJSON($.tree.reference(&quot;#&quot; + treeName).get(&quot;json&quot;));
            $(&quot;#&quot; + hiddenName).val(jsonTree);
        }
[/sourcecode]
Bueno con esto ya tenemos las bases para ir construyendo nuestro control de servidor.
Happy Coding.</pre>

---
layout: post
title: Visual State Publisher
date: 2011-02-20 00:41
author: mravinale
comments: true
categories: [Uncategorized]
header-img: "img/post-bg-06.jpg"
---
The Idea is that when we do doubleClick over the list we’ll see the detail and again doing doubleClick we’ll see the list again.

In this proyect I’ve added some flavor doing some states animations, so in the viewmodel you may see this implementation:
<pre class="code"><span style="color:blue;"> private </span><span style="color:#2b91af;">ICommand </span>_showDetailCommand;
       <span style="color:blue;">public </span><span style="color:#2b91af;">ICommand </span>ShowDetailCommand {
           <span style="color:blue;">get </span>{
               <span style="color:blue;">if </span>(_showDetailCommand == <span style="color:blue;">null</span>) {
                   _showDetailCommand = <span style="color:blue;">new </span><span style="color:#2b91af;">RelayCommand</span>&lt;<span style="color:blue;">string</span>&gt;(state =&gt;
                       <span style="color:#2b91af;">Publisher</span>.<span style="color:#2b91af;">VSM</span>.Publish(state)
                   );
               }
               <span style="color:blue;">return </span>_showDetailCommand;
           }
       }

       <span style="color:blue;">private </span><span style="color:#2b91af;">ICommand </span>_showListCommand;
       <span style="color:blue;">public </span><span style="color:#2b91af;">ICommand </span>ShowListCommand {
           <span style="color:blue;">get </span>{
               <span style="color:blue;">if </span>(_showListCommand == <span style="color:blue;">null</span>) {
                   _showListCommand = <span style="color:blue;">new </span><span style="color:#2b91af;">RelayCommand</span>&lt;<span style="color:blue;">string</span>&gt;(state =&gt;
                        <span style="color:#2b91af;">Publisher</span>.<span style="color:#2b91af;">VSM</span>.Publish(state)
                   );
               }
               <span style="color:blue;">return </span>_showListCommand;
           }
       }</pre>
For that we need to implement both commands as you can see and a event publisher implementation for each one, that what really is a <a href="http://en.wikipedia.org/wiki/Facade_pattern" target="_blank">Facade</a> that encapsulates a <a href="http://en.wikipedia.org/wiki/Mediator_pattern" target="_blank">Mediator Pattern</a> implementation that in Mvvm Light is named as Messenger .
<pre class="code"><span style="color:blue;"> public static class </span><span style="color:#2b91af;">VSM </span>{

           <span style="color:blue;">public static void </span>Publish&lt;T&gt;(T message) <span style="color:blue;">where </span>T : <span style="color:blue;">class </span>{
               <span style="color:#2b91af;">Messenger</span>.Default.Send&lt;T&gt;(message, <span style="color:blue;">typeof</span>(<span style="color:#2b91af;">VSM</span>));
           }

           <span style="color:blue;">public static void </span>Register&lt;T&gt;(<span style="color:blue;">object </span>reference, <span style="color:#2b91af;">Action</span>&lt;T&gt; action) <span style="color:blue;">where </span>T : <span style="color:blue;">class </span>{
               <span style="color:#2b91af;">Messenger</span>.Default.Register&lt;T&gt;(reference, <span style="color:blue;">typeof</span>(<span style="color:#2b91af;">VSM</span>), action);
           }

            <span style="color:blue;">public static void </span>UnRegister&lt;T&gt;(<span style="color:blue;">object </span>reference) <span style="color:blue;">where </span>T : <span style="color:blue;">class </span>{
               <span style="color:#2b91af;">Messenger</span>.Default.Unregister&lt;T&gt;(reference);
           }

       }</pre>
Ok, but how can I call the View?!??! because the VSM is there!!! there’s a couple solutions for that, but what I really like is to use a behavior, the code is quite reusable and we can attach to any control the code is quite simple:
<pre class="code"><span style="color:blue;"> public class </span><span style="color:#2b91af;">VSMBehavior </span>: <span style="color:#2b91af;">Behavior</span>&lt;<span style="color:#2b91af;">FrameworkElement</span>&gt; {

        <span style="color:blue;">public </span>VSMBehavior() { }

        <span style="color:blue;">protected override void </span>OnAttached() {
            <span style="color:blue;">base</span>.OnAttached();

            <span style="color:#2b91af;">Publisher</span>.<span style="color:#2b91af;">VSM</span>.Register&lt;<span style="color:blue;">string</span>&gt;(<span style="color:blue;">this</span>, state =&gt;
                <span style="color:#2b91af;">VisualStateManager</span>.GoToState(AssociatedObject <span style="color:blue;">as </span><span style="color:#2b91af;">Control</span>, state, <span style="color:blue;">true</span>)
            );
        }

        <span style="color:blue;">protected override void </span>OnDetaching() {
            <span style="color:blue;">base</span>.OnDetaching();

            <span style="color:#2b91af;">Publisher</span>.<span style="color:#2b91af;">VSM</span>.UnRegister&lt;<span style="color:blue;">string</span>&gt;(<span style="color:blue;">this</span>);
        }
    }</pre>
In case you want to see other implementations your are free to see some of them <a href="http://tdanemar.wordpress.com/2009/11/15/using-the-visualstatemanager-with-the-model-view-viewmodel-pattern-in-wpf-or-silverlight/" target="_blank">VSM1</a>, <a href="http://blogs.infosupport.com/blogs/alexb/archive/2010/04/02/silverlight-4-using-the-visualstatemanager-for-state-animations-with-mvvm.aspx" target="_blank">VSM2</a>,

But one of the best implementations for MVVM VSM is here the <a title="Visual State Agregator" href="http://csharperimage.jeremylikness.com/2010/03/introducing-visual-state-aggregator.html">Visual State Aggregator</a> by <a href="http://twitter.com/jeremylikness">Jeremy Likness</a>. Based on the <a href="http://martinfowler.com/eaaDev/EventAggregator.html">Event Agregator Pattern</a>, but is very similar to the Mediator pattern and in Mvvm is already implemented, so I thaught to use it in a similar way.

Ok, but all this really works?

that would be my question if I was reading a post like this, for that reason I’ve made this short video showing the project working:

<span style="font-family:monospace;">[youtube=http://www.youtube.com/watch?v=07KweYe5yRc]</span>

I really hope this article would be helpful and encorage you to use Ninject Di/Ioc and extensions with Mvvm Light,

Here you can download the <a href="http://cid-86b4ae59157bdbba.office.live.com/self.aspx/Silverlight%20Projects/SLMvvmLightNinjectINPC.zip">project</a>.

The project uses some Data sample provided by Blend, in some future post we can add some real services and some more animations.

---
layout: post
title: Clean Mvvm Property Changed Handler
date: 2012-06-10 20:12
author: mravinale
comments: true
categories: [Uncategorized]
---
In order to handle property changed events in the view Model, you need to implement something like this:

[code lang="csharp"]&lt;/pre&gt;
&lt;pre class=&quot;csharpcode&quot;&gt;&lt;span class=&quot;kwrd&quot;&gt;private&lt;/span&gt; &lt;span class=&quot;kwrd&quot;&gt;int&lt;/span&gt; _property;
&lt;span class=&quot;kwrd&quot;&gt;public&lt;/span&gt; &lt;span class=&quot;kwrd&quot;&gt;int&lt;/span&gt; Property
{
       get
       {
           &lt;span class=&quot;kwrd&quot;&gt;return&lt;/span&gt; _property;
       }
       set
       {
            _property = &lt;span class=&quot;kwrd&quot;&gt;value&lt;/span&gt;;
            NotifyOfPropertyChange(() =&gt; Property);
       }
}

&lt;span class=&quot;kwrd&quot;&gt;private&lt;/span&gt; &lt;span class=&quot;kwrd&quot;&gt;int&lt;/span&gt; _property2;
&lt;span class=&quot;kwrd&quot;&gt;public&lt;/span&gt; &lt;span class=&quot;kwrd&quot;&gt;int&lt;/span&gt; Property2
{
       get
       {
           &lt;span class=&quot;kwrd&quot;&gt;return&lt;/span&gt; _property2;
       }
       set
       {
            _property2 = &lt;span class=&quot;kwrd&quot;&gt;value&lt;/span&gt;;
            NotifyOfPropertyChange(() =&gt; Property2);
       }
}&lt;/pre&gt;
&lt;pre class=&quot;csharpcode&quot;&gt;&lt;/pre&gt;
&lt;pre class=&quot;csharpcode&quot;&gt;&lt;span class=&quot;kwrd&quot;&gt;protected&lt;/span&gt; &lt;span class=&quot;kwrd&quot;&gt;void&lt;/span&gt; Init()
{
&lt;span class=&quot;kwrd&quot;&gt; this&lt;/span&gt;.PropertyChanged += (s, e) =&gt;
        {
            &lt;span class=&quot;kwrd&quot;&gt;if&lt;/span&gt; (e.PropertyName == &lt;span class=&quot;str&quot;&gt;&quot;Property&quot;&lt;/span&gt;)
            {
                MessageBox.Show(&lt;span class=&quot;str&quot;&gt;&quot;Hello&quot;&lt;/span&gt;);
            }
            &lt;span class=&quot;kwrd&quot;&gt;if&lt;/span&gt; (e.PropertyName == &lt;span class=&quot;str&quot;&gt;&quot;Property2&quot;&lt;/span&gt;)
            {
                MessageBox.Show(&lt;span class=&quot;str&quot;&gt;&quot;Hello2&quot;&lt;/span&gt;);
            }
        };
}
[/code]


I think this could be quite messy, using this magic strings, and can be harder to maintain,as well.

may be this solution can be clear to understand, when you review the code:
[code lang="csharp"]

PropertyChanged += (s, e) =&gt; PropertyChangedHandler(e.PropertyName, () =&gt; Property, ()=&gt; MessageBox.Show(&quot;Hello&quot;));
 PropertyChanged += (s, e) =&gt; PropertyChangedHandler(e.PropertyName, () =&gt; Property2, () =&gt; MessageBox.Show(&quot;Hello2&quot;));

[/code]


<div class="csharpcode">
<span class="kwrd">public</span> <span class="kwrd">static</span> <span class="kwrd">class</span> Helper

{

<span class="kwrd">   public</span> <span class="kwrd">static</span> <span class="kwrd">string</span> GetProperty&lt;TProperty&gt;(Expression&lt;Func&lt;TProperty&gt;&gt; propertyExpresion)

{

var property = propertyExpresion.Body <span class="kwrd">as</span> MemberExpression;

<span class="kwrd">if</span> (property == <span class="kwrd">null</span> || !(property.Member <span class="kwrd">is</span> PropertyInfo))

{

<span class="kwrd">throw</span> <span class="kwrd">new</span> ArgumentException(<span class="str">"Expression must be of the form   this.PropertyName'.  Invalid expression "</span> + propertyExpresion);

}

<span class="kwrd">return</span> property.Member.Name;

}

}
</div>

this Resource was taken from: <a href="http://pastie.org/960139">http://pastie.org/960139</a>

And the you can write several custom property change handlers, even you may include some extra conditions, as CanExecute functions:

<pre class="csharpcode"><span class="kwrd">public</span> <span class="kwrd">void</span> PropertyChangedHandler(<span class="kwrd">string</span> propertyChanged, Expression&lt;Func&lt;<span class="kwrd">object</span>&gt;&gt; classProperty, Func&lt;<span class="kwrd">string</span>, <span class="kwrd">string</span>&gt; func,  Func&lt;<span class="kwrd">bool</span>&gt; canExecute)
{
    var property = Helper.GetProperty(classProperty);
    <span class="kwrd">if</span> (propertyChanged == property &amp;&amp; canExecute()) func(propertyChanged);
}

<span class="kwrd">public</span> <span class="kwrd">void</span> PropertyChangedHandler(<span class="kwrd">string</span> propertyChanged, Expression&lt;Func&lt;<span class="kwrd">object</span>&gt;&gt; classProperty, Func&lt;<span class="kwrd">string</span>, <span class="kwrd">string</span>&gt; func)
{
    var property = Helper.GetProperty(classProperty);
    <span class="kwrd">if</span> (propertyChanged == property) func(propertyChanged);
}

<span class="kwrd">public</span> <span class="kwrd">void</span> PropertyChangedHandler(<span class="kwrd">string</span> propertyChanged, Expression&lt;Func&lt;<span class="kwrd">object</span>&gt;&gt; classProperty, Action action, Func&lt;<span class="kwrd">bool</span>&gt; canExecute)
{
    var property = Helper.GetProperty(classProperty);
    <span class="kwrd">if</span> (propertyChanged == property &amp;&amp; canExecute()) action();
}

<span class="kwrd">public</span> <span class="kwrd">void</span> PropertyChangedHandler(<span class="kwrd">string</span> propertyChanged, Expression&lt;Func&lt; classProperty, Action action)
{
    var property = Helper.GetProperty(classProperty);
    if(propertyChanged == property) action();
}</pre>
may the code be with you

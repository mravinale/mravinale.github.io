---
layout: post
title: Flick Behavior – Using a SL Listbox control as you do on WP7
date: 2010-11-14 02:40
author: mravinale
comments: true
categories: [Silverlight]
---
Working with Windows Phone 7, I get used to handle the ListBox control using that flickering efect with my mouse, then I’ve started to seek on the web for some control in Silverlight that imitates the same behavior, and I’ve found the <strong>Sasha Barber's<a href="http://sachabarber.net/?p=628" target="_blank"> </a>web</strong>,  who built a <a title="Permalink to  Creating A Scrollable Control Surface In WPF" rel="bookmark" href="http://sachabarber.net/?p=225">Creating A Scrollable Control Surface In WPF</a> with a nice efect and a very cool way to do the maths and calculate the speed and distance.

So I’ve started to make it work in Silverlight as a user control, the first drop worked, but I made it using a ScrollViewer and ItemsControl, as this example:

<span style="color:blue;">&lt;</span><span style="color:#a31515;">Grid </span><span style="color:red;">x</span><span style="color:blue;">:</span><span style="color:red;">Name</span><span style="color:blue;">="LayoutRoot"&gt;
&lt;</span><span style="color:#a31515;">ScrollViewer </span><span style="color:red;">x</span><span style="color:blue;">:</span><span style="color:red;">Name</span><span style="color:blue;">="MyScrollViewer"</span><span style="color:red;">Margin</span><span style="color:blue;">="0"</span><span style="color:red;">Background</span><span style="color:blue;">="White"</span>
<pre class="code"><span style="color:blue;">            </span><span style="color:red;">HorizontalScrollBarVisibility</span><span style="color:blue;">="Disabled"&gt;</span></pre>
<pre class="code"><span style="color:blue;">
            &lt;</span><span style="color:#a31515;">ItemsControl </span><span style="color:red;">x</span><span style="color:blue;">:</span><span style="color:red;">Name</span><span style="color:blue;">="itemsControl"
                </span><span style="color:red;">MouseLeftButtonDown</span><span style="color:blue;">="itemsControl_MouseLeftButtonDown"
                </span><span style="color:red;">MouseLeftButtonUp</span><span style="color:blue;">="itemsControl_MouseLeftButtonUp"
                </span><span style="color:red;">MouseMove</span><span style="color:blue;">="itemsControl_MouseMove"
                </span><span style="color:red;">ItemTemplate</span><span style="color:blue;">="{</span><span style="color:#a31515;">StaticResource </span><span style="color:red;">ItemTemplate</span><span style="color:blue;">}"
                </span><span style="color:red;">ItemsSource</span><span style="color:blue;">="{</span><span style="color:#a31515;">Binding </span><span style="color:red;">DataSource</span><span style="color:blue;">, </span><span style="color:red;">ElementName</span><span style="color:blue;">=userControl}" /&gt;

        &lt;/</span><span style="color:#a31515;">ScrollViewer</span><span style="color:blue;">&gt;
&lt;/</span><span style="color:#a31515;">Grid</span><span style="color:blue;">&gt;
</span></pre>
<a href="http://11011.net/software/vspaste"></a>

but I didn’t like it, because the listbox it is more popular, and it has already a ScrollViewer and the ItemsPresenter as a Template, but it was ok, so far I’ve learned that the key is handle the MouseLeftbuttonDown , MouseLeftButtonUp and Mousemove events, to do the Maths.

So now what I’ve got to do is create a <strong>behavior </strong>for the ListBox, navigate inside the template, and get attached to the events mentionated before.

yap,but how can I find the ItemsPresenter within the ListBox?,the answer would be with the findByType&lt;T&gt; method, inspired by the XamlQuery library, with some magic to make it portable ;)
<pre><span style="color:blue;">protected </span><span style="color:#2b91af;">IEnumerable</span>&lt;<span style="color:#2b91af;">DependencyObject</span>&gt; FindByType&lt;T&gt;(<span style="color:#2b91af;">DependencyObject </span>control) <span style="color:blue;">where </span>T : <span style="color:blue;">class </span>{</pre>
<pre><span style="color:blue;"><span style="color:#222222;"> </span> var </span>childrenCount = <span style="color:#2b91af;">VisualTreeHelper</span>.GetChildrenCount(control);</pre>
<pre><span style="color:blue;">  for </span>(<span style="color:blue;">var </span>index = 0; index &lt; childrenCount; index++) {</pre>
<pre><span style="color:blue;">      var </span>child = <span style="color:#2b91af;">VisualTreeHelper</span>.GetChild(control, index);</pre>
<pre><span style="color:blue;">      if </span>(child <span style="color:blue;">is </span>T) <span style="color:blue;">yield return </span>child;</pre>
<pre><span style="color:blue;">      foreach </span>(<span style="color:blue;">var </span>desc <span style="color:blue;">in</span>FindByType&lt;T&gt;(child)) {</pre>
<pre><span style="color:blue;">        if </span>(desc <span style="color:blue;">is </span>T) <span style="color:blue;">yield return </span>desc;</pre>
<pre>      }</pre>
<pre>  }</pre>
<pre>}</pre>
And now let’s find the scrollViewer and the ItemsPresenter inside the ListBox.
<pre class="code">MyScrollViewer = FindByType&lt;<span style="color:#2b91af;">ScrollViewer</span>&gt;(AssociatedObject).ToList().First() <span style="color:blue;">as </span><span style="color:#2b91af;">ScrollViewer</span>;</pre>
<pre class="code">MyItemPresenter = FindByType&lt;<span style="color:#2b91af;">ItemsPresenter</span>&gt;(AssociatedObject).ToList().First() <span style="color:blue;">as </span><span style="color:#2b91af;">ItemsPresenter</span>;</pre>
Yea!, so far so good, now I've got the main controls for the LisBox, and now let's try to do the same that I did with the user control. but the problem was that the MouseLeftButtonDown didn’t work beacuse the event was toked by it’s parent in this case Listbox for the  SelectionChanged event so the trick is to add handlers:
<pre class="code">MyItemPresenter.AddHandler(<span style="color:#2b91af;">UIElement</span>.MouseLeftButtonDownEvent,</pre>
<pre class="code">                           <span style="color:blue;">new </span><span style="color:#2b91af;">MouseButtonEventHandler</span>(ItemPresenter_MouseLeftButtonDown), <span style="color:blue;">true</span>);
MyItemPresenter.AddHandler(<span style="color:#2b91af;">UIElement</span>.MouseLeftButtonUpEvent,</pre>
<pre class="code">                           <span style="color:blue;">new </span><span style="color:#2b91af;">MouseButtonEventHandler</span>(ItemPresenter_MouseLeftButtonUp), <span style="color:blue;">true</span>);</pre>
Ok, now lets handle the events: first let’s track the mouse position, and calculate the desviation between the last point that it has been captured, and the current,so the follow event will be in charge of that:
<pre class="code"><span style="color:blue;">private void </span>AssociatedObject_MouseMove(<span style="color:blue;">object </span>sender, <span style="color:#2b91af;">MouseEventArgs </span>e) {

<span style="color:blue;">   if </span>(IsMouseCaptured ) {
      CurrentPoint = <span style="color:blue;">new </span><span style="color:#2b91af;">Point</span>(e.GetPosition(AssociatedObject).X,</pre>
<pre class="code">                               e.GetPosition(AssociatedObject).Y);
      <span style="color:blue;">var </span>delta = <span style="color:blue;">new </span><span style="color:#2b91af;">Point</span>(ScrollStartPoint.X - CurrentPoint.X,</pre>
<pre class="code">                            ScrollStartPoint.Y - CurrentPoint.Y);

      ScrollTarget.X = ScrollStartOffset.X + delta.X / Speed;
      ScrollTarget.Y = ScrollStartOffset.Y + delta.Y / Speed;

      MyScrollViewer.ScrollToHorizontalOffset(ScrollTarget.X);
      MyScrollViewer.ScrollToVerticalOffset(ScrollTarget.Y);
  }</pre>
<span style="font-family:'lucida sa';">And now let’s take a look to the events in charge to do the capture of the mouse clicks and points: </span>

<a href="http://11011.net/software/vspaste"></a>
<pre class="code"><span style="color:blue;">private void </span>ItemPresenter_MouseLeftButtonDown(<span style="color:blue;">object </span>sender, <span style="color:#2b91af;">MouseButtonEventArgs </span>e) {

    ScrollStartPoint = <span style="color:blue;">new </span><span style="color:#2b91af;">Point</span>(e.GetPosition(AssociatedObject).X,</pre>
<pre class="code">                                 e.GetPosition(AssociatedObject).Y);
    ScrollStartOffset.X = MyScrollViewer.HorizontalOffset;
    ScrollStartOffset.Y = MyScrollViewer.VerticalOffset;

    IsMouseCaptured = !IsMouseCaptured;

}

<span style="color:blue;">private void </span>ItemPresenter_MouseLeftButtonUp(<span style="color:blue;">object </span>sender, <span style="color:#2b91af;">MouseButtonEventArgs </span>e) {

    IsMouseCaptured = !IsMouseCaptured;
}</pre>
Take a look to this video Introducing the behavior.

[youtube=http://www.youtube.com/watch?v=-Mwg6BGqQpA]

if you are interested to see how it works take a look to this video

[youtube=http://www.youtube.com/watch?v=j6AVwqhl9ZI]

remember to visit Michael's page for see how to create sample datasource  <a href="http://geekswithblogs.net/mbcrump/archive/2010/09/03/easy-way-to-generate-sample-data-for-your-silverlight-4.aspx">http://geekswithblogs.net/mbcrump/archive/2010/09/03/easy-way-to-generate-sample-data-for-your-silverlight-4.aspx</a>

you’re free to grab the code and see how it was built, whith this post I’ve tried to share with you the journey making this component, and I really hope, that you may find this control useful and you may improve it for your own needs.

<a href="http://gallery.expression.microsoft.com/en-us/FlickBehavior">See it in action and download In expression Blend Gallery!</a>

<a href="http://gallery.expression.microsoft.com/en-us/FlickBehavior"><img class="size-thumbnail wp-image-283 alignleft" title="Expression galery" src="http://mravinale.files.wordpress.com/2010/11/expression-galery.jpg?w=150" alt="" width="150" height="95" /></a>

may the code be with you.

Mariano Ravinale

<a rel="license" href="http://creativecommons.org/licenses/by/3.0/"><img style="border-width:0;" src="http://creativecommons.org/images/public/somerights20.png" alt="Creative Commons License" /></a>

This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/3.0/">Creative Commons Attribution 3.0 Unported License</a>

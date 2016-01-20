---
layout: post
title: Asp.net MVC Simple CQRS part 2 - Edit form using jquery dialog
date: 2012-09-23 21:04
author: mravinale
comments: true
categories: [Asp.Net MVC]
---
<strong>The goal:</strong>

Reuse all Asp.net MVC Form client/server validations functionality using jquery dialog.

This feature will be useful for this post series and for building general crud applications, as well.

<strong>Example:</strong>
[youtube=http://www.youtube.com/watch?v=3aTI5MdDezo]

First, let's create our model:

[code language="csharp"]
public class AlbumModel
	{
		[Display(Name = &quot;Id&quot;)]
		public int AlbumId { get; set; }

		[Required]
		public string Title { get; set; }

		[HiddenInput(DisplayValue = false)]
		[Display(Name = &quot;Artists&quot;)]
		public int ArtistId { get; set; }

		public string ArtistName { get; set; }
		
		public IEnumerable&lt;SelectListItem&gt; Artists { get; set; }
	}
[/code]

Now, we should create a partial view in order to render the Form

<strong>Edit.cshtml:</strong>

[code language="html"]
@model Chinook.Core.AlbumModel

@using (Html.BeginForm(&quot;Edit&quot;, &quot;Albums&quot;,FormMethod.Post, new { id = &quot;EditForm&quot; }))
{
    @Html.ValidationSummary(true)
    &lt;fieldset&gt;
        &lt;legend&gt;Edit Album - @Model.AlbumId&lt;/legend&gt;
         @Html.HiddenFor(model =&gt; model.AlbumId)
        &lt;div class=&quot;editor-label&quot;&gt;
            @Html.LabelFor(model =&gt; model.Title)
        &lt;/div&gt;
        &lt;div class=&quot;editor-field&quot;&gt;
            @Html.EditorFor(model =&gt; model.Title)
            @Html.ValidationMessageFor(model =&gt; model.Title)
        &lt;/div&gt;

        &lt;div class=&quot;editor-label&quot;&gt;
            @Html.LabelFor(model =&gt; model.ArtistId)
        &lt;/div&gt;
        &lt;div class=&quot;editor-field&quot;&gt;
            @Html.DropDownListFor(model =&gt; model.ArtistId, Model.Artists, new { style = &quot;width: 206px&quot; })
            @Html.ValidationMessageFor(model =&gt; model.ArtistId)
        &lt;/div&gt;
                &lt;p&gt;
			&lt;input type=&quot;submit&quot; value=&quot;Save&quot; /&gt;
		&lt;/p&gt;
    &lt;/fieldset&gt;
}

&lt;!--Lets add jquery validations for model Validations before submit--&gt;
&lt;script src=&quot;@Url.Content(&quot;~/Scripts/jquery.validate.min.js&quot;)&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;
&lt;script src=&quot;@Url.Content(&quot;~/Scripts/jquery.validate.unobtrusive.min.js&quot;)&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;
&lt;script type=&quot;text/javascript&quot; charset=&quot;utf-8&quot;&gt;
    $(document).ready(function () {
        $('#EditForm').ajaxForm({
            beforeSubmit: function () {
                return $('#EditForm').valid();
            },
            success: function () {
                $('#dialog').dialog(&quot;close&quot;);
            }
        });
    });
&lt;/script&gt;
[/code]

As you can see I've added jquery.validate.min.js and jquery.validate.unobtrusive.min.js this will executed when 
the partial view is loaded after ajax call has been made.

A little peace of functionality must be writen first, two events: 
<em>before submit</em>, verify all the fields are valid and
<em>Success</em>, when everything is ok close the dialog.


<strong>Album.js</strong>

[code language="Javascript"]
$(function () {
    Albums.init();
});

//Module Pattern: http://tinyurl.com/crdgdzn
var Albums = function () {

  var initLayout = function () {
        $(&quot;.edit-link&quot;).button();
        $(&quot;#dialog&quot;).dialog({ autoOpen: false, modal: true });
    };

  var initEditForm = function () {
     $(&quot;.edit-link&quot;).live('click', function (event) {
        event.preventDefault();
         $.ajax({
             url: &quot;Albums/Edit/1&quot;,
             success: function (data) {
                  $(&quot;#dialog&quot;).html(data);
                 $(&quot;#dialog&quot;).dialog('open');
             },
             error: function () { alert(&quot;Sorry,error&quot;); }
          });
     });
   }
    // Reveal public pointers to
    // private functions and properties
    return {
        init: function () {
            initLayout();
            initEditForm();
        }
    };

} ();
[/code]

Now in order to call the Form we need to add an ajax call, we should call to the server asking for the information and the html, and add this html to the div with the id "dialog".


<strong>AlbumsController:</strong>

[code language="csharp"]
[HttpGet]
public ActionResult Edit(GetAlbumsByIdQuery query)
{
    return PartialView(&quot;Edit&quot;, QueryProcessor.Execute(query));
}
[/code]

When someone calls to this method we will return the partial view with the information retrieved from the database.

<strong>Index.cshtml</strong>

[code language="html"]
&lt;div id=&quot;demo&quot;&gt;	
	&lt;div&gt;	   
	    &lt;a class=&quot;edit-link&quot; href=&quot;&quot;&gt;Edit Album&lt;/a&gt;
	&lt;/div&gt;

	&lt;table id=&quot;example&quot; class=&quot;display&quot; &gt;
		&lt;thead&gt;
			&lt;tr&gt;
			   &lt;th&gt;Id&lt;/th&gt;
			   &lt;th&gt;Title&lt;/th&gt;
                           &lt;th&gt;Artist Name&lt;/th&gt;
			&lt;/tr&gt;
		&lt;/thead&gt;
		&lt;tbody&gt;&lt;/tbody&gt;
	&lt;/table&gt;
    
    &lt;div id=&quot;dialog&quot; title=&quot;Basic dialog&quot;&gt;&lt;/div&gt;
&lt;/div&gt;

&lt;script src=&quot;@Url.Content(&quot;~/Scripts/albums.js&quot;)&quot; type=&quot;text/javascript&quot;&gt;&lt;/script&gt;
[/code]


Finally the index page, here you can see the anchor that will be the trigger for the dialog call.
and the div that will be the container for the html form rendered.

in the next post, we'll see how to render the data grid using <a href="http://datatables.net/" title="datatables" target="_blank">http://datatables.net/</a> using ajax.

Download or Fork the project on <a href="https://github.com/mravinale/Cronos">GitHub </a>

May the code be with you!.

Mariano Ravinale

<a href="http://creativecommons.org/licenses/by/3.0/" rel="license"><img src="http://creativecommons.org/images/public/somerights20.png" alt="Creative Commons License" /></a>

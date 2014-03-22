Created by: Urs Braem
Created date: 2014-03-22
Last updated: 2014-03-22
Authors: Urs Braem
Title: How do I use Perch to manage and render related items?
Tags: templates

# How do I use Perch to manage and render related items?

## You want to use Perch to manage relational content items and output fully rendered templates with perch:content.

In the backend, use the "dataselect" field type (http://docs.grabaperch.com/docs/templates/attributes/type/dataselect/) to refer to other items.

In the frontend, use the "each" callback function (http://docs.grabaperch.com/docs/content/perch-content-custom/).

## Example

Your client is a company that manufactures everything. Ah no. Let's stay real, it's a theatre company that has done a bunch of plays. In our case (we're retrofitting an existing site), each play's entry just consists of a title and a link.

They have several lists they are now managing with Perch: an event calendar and several statistics. In each list, the plays' titles and links are used. To avoid errors, for convenience etc., we want them to enter the play's details only once.

Of course, in the rendered template, e.g. the calendar, we then want the link and play title to render based on a template.

In the example, we'll stick with the event calendar.

## Define the play template: play.html

Create a template for a play. Just a regular perch template in perch/templates/content

```
<perch:if exists="play_link"><a href="<perch:content id="play_link" type="text" label="Link" />"></perch:if>
<perch:content id="play_title" type="text" label="Title" title="true" required="true" />
<perch:if exists="play_link"></a></perch:if>
```

> Note: don't forget that ID's can't have dashes (play-html) and that tags must always be properly closed or self-closing.


## Seize the plays: plays.php

Create a new page plays.php

```
<?php include($_SERVER['DOCUMENT_ROOT'].'/perch/runtime.php');
	// true to suppress the output if the page should stay here
  perch_content('Plays',true); 
  ?>
```

Load it in the browser, go to Perch, assign the play.html template to the region, add some plays.

You also make the region shared (see below).

## Define an event: event_be.html and event_fe.html

Now we define the template for the event calendar. 

For the example's sake, let's say it's that basic it only features the play's title, a link and some text (of course, this would have date etc. added).

> Note: I like to divide my templates into frontend and backend templates, so I can move stuff freely for the backend and have less clutter in the already cluttered frontend template. If you do so too, you can also use separate folders for FE and BE templates.

Of course, you can put everything into one template and hide / suppress fields for the front- and backend.

So in my case it's:

event_be.html
```
<perch:content id="playID" type="dataselect" label="Choose a play" page="/plays.php" region="Plays" options="play_title" values="_id" required="true" />
<perch:content id="morecontent" type="textarea" label="Description" />
```

> Note: if you have set the 'Plays' region to "shared", you have to write set the attribute page="*".

event_fe.html
```
<section class="play">
   <perch:content id="play_html" encode="false" />
   <perch:content id="morecontent" />
</section>
```

> Note: The FE template doesn't even need the dataselect field. Instead we give it a field play_html we will fill later. But you have to add encode="false" to achieve proper rendering.

And: If you add <perch:content id="playID"> to the FE template, you'll see that the dataselect field only returns it's value. Not the entire template of the linked item. That's why we have to do some more rendering.

## Put it together: events.php

On the page events.php, finally, we use the "each" callback of perch_content_custom.

Use perch_content('Events') to create the region and remove it after you've mapped it to events_be.html.

Enter some Events in Perch. You will be able to select a play for each entry via the dataselect dropdown, the value being the play's ID.

> Note: I have one remaining issue here: I wasn't able to use the text used as "option" in the dataselect as item title in the Perch Backend. Ugly workaround would be to have editors enter the play's title again manually for better recognition in perch and use that as the item's title. I hope there's a better solution. 

Back in events.php, use perch_content_custom to render not only the events list, but also the referred "play" entry.

For this, we'll use perch_content_custom's "each" callback to fill the ```<perch:content id="play_html" encode="false" />``` tag from our events_fe.html template.

```
perch_content_custom('Events',array(
	  'template'=>'events_fe.html',
	  'each' => function($item) {
          // fill the slot we've prepared in the FE template
		  		$item['play_html'] = perch_content_custom('Plays', array(
		  		// use this as a "sub" template
          'template'=>'play.html',
          // perch sees the region only on the page we've put it on, so tell it to look there
          // remove if using a shared region
		  		'page'=>'/plays.php',
          // now it would render all items. we don't want all items
          // just the one where the play's id
		  		'filter'=>'_id',
          // is equal to
		  		'match'=>'eq',
          // the play's id we've selected in the dataselect
		  		'value'=>$item['playID'],
	  			// return : true
          ),true);
          // and return the item to the callback
		  		return $item;
		  	}
	  	));
```

> Note: We're using the template for the play item type as a "sub" template for rendering. That's not mandatory, we could also write another custom FE template and use that.

> Note: If it's becoming confusing, I use <perch:showall /> in the template and in PHP, e.g. after `'each' => function($item) {`:

```
echo '<pre>';
print_r($item);
echo '</pre>'; 
```

That's it! The "subtemplate" will be included into the region's main template.

Here's the desired output:

```
<section class="play">
<a href="/link/">The play's title</a>
<p>The play's description</p>
</section>
```

Now you can use Perch to manage and output all kinds of related data.












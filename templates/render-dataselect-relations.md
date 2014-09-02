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

Your client is a theatre company that has done a bunch of plays. In the website, each play's entry just consists of a title and a link. We will use the data from the plays in an event calendar.

## Add some plays

Create a new page plays.php, create a multi-item region "Plays", e.g. with perch_content_create() and a corresponding perch template. Add a few entries in the backend.

## Add some events

On another page, events.php, add a multi-item region for events. The template could look like this:

event.html
```
<perch:content id="playID" type="dataselect" label="Choose a play" page="/plays.php" region="Plays" options="play_title" values="_id" required="true" suppress="true"/>
<section class="play">
   <perch:content id="play_html" encode="false" />
   <perch:content id="morecontent" type="textarea" label="Description" />
</section>
```

> Note: if you have set the 'Plays' region to "shared", you have to set the attribute ```page="*"``` instead of ```page="/plays.php"```.

We suppress the dataselect field in the frontend - we will place some content from it into "play_html" later on. Make sure to add encode="false" to achieve proper rendering.

Enter some data for the events. You'll already be able to select plays via the dataselect dropdown - without effect in the frontend yet though.

> Try adding <perch:content id="playID"> to the FE template. You'll notice that the dataselect field only returns it's value. Not the entire template of the linked item. That's why we have to do some more rendering.

## Render the whole template

Finally, we use the "each" callback of perch_content_custom to render the whole, referenced "play" template.

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
          // now it would render all items. but we don't want all items,
          // just the one where the play's ID
		  		'filter'=>'_id',
          // is equal to
		  		'match'=>'eq',
          // the ID of the play we've selected in the dataselect
		  		'value'=>$item['playID'],
	  			// return : true
          ),true);
          // and return the item to the callback
		  		return $item;
		  	}
	  	));
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

---
layout: post
title: ACF's get_fields();
description: "Sometimes a big array is exactly what you want."
modified: 2013-12-10
tags: [acf, code, wordpress, roots]
share: true  
---

Sometimes [Advanced Custom Field's](http://www.advancedcustomfields.com){:target="_blank"} `get_field();` and `the_field();` functions are enough for pages with a few custom fields, but for pages with lots more, or with really complex custom fields (like those using a repeater or flexible image field), you've got to use something with a little more firepower (and organization).

##Let's talk about ACF for a sec.
I love Advanced Custom Fields. It makes developing custom WordPress sites SO MUCH BETTER, especially when you have a client who has very specific needs. It changed the way I develop custom WordPress themes, which by the way, is pretty much all I do. 

##Use case.
Developers are like kings of bad examples, so here I go. We've got a animal shelter that wants to be able to easily edit the information about the pets they have up for adoption.

When we look at our information architecture, we see our custom post type of "pet" need to have these fields:

* Name (WordPress gives us this)
* Type (lets pretend this is a select box with options Dog, Cat, Other)
* Image (ACF image field)
* Description (ACF wysiwyg)

Simple enough.


##Outputting with the_field();
So we've made our custom post type and we're in our `single-pet.php` file or whatever. I use the [Roots](http://www.roots.io){:target="_blank"} framework so it makes it incredibly easy for me to create new custom post types and put whatever I want into my template file.

Let's print our custom fields stuff using `the_field();` - remember this function echos the variable for us!
{% highlight html+php %}
<div class="page-header">
  <h1>
    <?php echo roots_title(); ?>
  </h1>
</div>

<div class="type">
  <?php the_field('type'); ?>
</div>

<img src="<?php the_field('image'); ?>" alt="a pet">

<div class="description">
  <?php the_field('description'); ?>
</div>
{% endhighlight %}

(let's pretend the image field isn't printing an image object, just the url)

But that's not good enough, because if any of these are non-existant, we might get an error, or nothing will print, or something. Not reliable enough and NOT good enough if a client is paying you to do good programming.


##Outputting with get_field();

So let's use `get_field();` to check that the field is there before we go all gung-ho and print the little bugger. I like to save the variables up at the top of the file.

I also like to use php shortcodes `<?= $echo_this_variable; ?>` within my templates so I know where I'm outputting my variables. This is totally personal preference.
{% highlight html+php %}
<?php
//Save Variables
$name = roots_title();
$type = get_field('type');
$image = get_field('image');
$description = get_field('description');

  if($name) :
?>

<div class="page-header">
  <h1>
    <?= $name; ?>
  </h1>
</div>

<?php
  endif;
  if($type):
?>

<div class="type">
  <?= $type; ?>
</div>

<?php
  endif;
  if($image):
?>

<img src="<?= $image ?>" alt="<?= $name; ?>">

<?php
  endif;
  if($description):
?>

<div class="description">
  <?= $description; ?>
</div>
<?php
  endif;
?>
{% endhighlight %}

Ok so that's cool, and we have some kind of simple field checking, but at the top we're saving each variable on a new line, making a new DB call using `get_fields();`. This is totally fine, until you have over 100 custom fields on your page. That sucks because you're calling the DB each time you try and get that new value.

##Give me an array, baby.
Enter the world of `get_fields();` a function created to give those who would rather call the DB once and go through an array of the custom fields attached to that post. I much prefer this method. I like to think it's more performant (correct me if I'm wrong) and for me, it's personally easier to navigate. You can read Elliot's Documentation on the function [here](http://www.advancedcustomfields.com/resources/functions/get_fields/).

I've added my dog, Dina, as a pet on my development site for some sample data (you can't have her). Let's grab some information using `get_fields();`.

First we have to save the variable and grab all of the fields.
{% highlight html+php %}
<?php
//Save Variables
$pet = get_fields();
?>
{% endhighlight %}

Now we will use php's var_dump() function to display our data about the pet on the page.
{% highlight html+php %}
<?php
//Save Variables
$pet = get_fields();

var_dump($pet);
?>
{% endhighlight %}
We get an array back. Sometimes it's easier to open up developer tools to check out the source and/or copy and paste it into your text editor. Btw, I'm returning an image object now, not just the image url.
{% highlight html %}
array(3) {
  ["type"]=>
  string(3) "dog"
  ["image"]=>
  array(10) {
    ["id"]=>
    int(13)
    ["alt"]=>
    string(17) "some text for alt"
    ["title"]=>
    string(19) "2012-11-04 16.51.40"
    ["caption"]=>
    string(0) ""
    ["description"]=>
    string(0) ""
    ["mime_type"]=>
    string(10) "image/jpeg"
    ["url"]=>
    string(77) "http://local.wordpress.dev/wp-content/uploads/2013/11/2012-11-04-16.51.40.jpg"
    ["width"]=>
    int(960)
    ["height"]=>
    int(635)
    ["sizes"]=>
    array(9) {
      ["thumbnail"]=>
      string(85) "http://local.wordpress.dev/wp-content/uploads/2013/11/2012-11-04-16.51.40-150x150.jpg"
      ["thumbnail-width"]=>
      int(150)
      ["thumbnail-height"]=>
      int(150)
      ["medium"]=>
      string(85) "http://local.wordpress.dev/wp-content/uploads/2013/11/2012-11-04-16.51.40-300x198.jpg"
      ["medium-width"]=>
      int(300)
      ["medium-height"]=>
      int(198)
      ["large"]=>
      string(77) "http://local.wordpress.dev/wp-content/uploads/2013/11/2012-11-04-16.51.40.jpg"
      ["large-width"]=>
      int(960)
      ["large-height"]=>
      int(635)
    }
  }
  ["description"]=>
  string(9) "blablabla"
}
{% endhighlight %}
That just seems kinda...sexy to me. Data perfectly organized and ready for us to use it as we need. Let's save our variables now and print our stuff. 

You'll notice, we're grabbing the data for the variables directly from the array which is saved in the `$pet` variable, and we can extract lots of data from the image field. I'm going to display a medium-sized image and also grab the alt for the image.

Here's the whole page's source:
{% highlight html+php %}
<?php
//Save Variables
$pet = get_fields();
$name = roots_title();
$type = $pet["type"];
$image = $pet["image"];
$image_url = $image["sizes"]["medium"];
$image_alt = $image["alt"];
$description = $pet["description"];

  if($name) :
?>

<div class="page-header">
  <h1>
    <?= $name; ?>
  </h1>
</div>

<?php
  endif;
  if($type):
?>

<div class="type">
  <?= $type; ?>
</div>

<?php
  endif;
  if($image):
?>

<img src="<?= $image_url; ?>" alt="<?= $image_alt; ?>">

<?php
  endif;
  if($description):
?>

<div class="description">
  <?= $description; ?>
</div>
<?php
  endif;
?>
{% endhighlight %}

So outputting the variables is the same, but it's much easier to actually save those variables.

##The end.
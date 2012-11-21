Advanced Custom Post Types: 2.0
===

This is a framework for creating not only custom post types, roles and taxonomies in WordPress but it will also give you the ability to rapidly create custom fields (post types only).

by Kevin Dees at http://kevindees.cc or on twitter https://twitter.com/kevindees

New to WordPress? Use Plugins!
- http://wordpress.org/extend/plugins/advanced-custom-fields/
- http://wordpress.org/extend/plugins/custom-post-type-ui/

Usage
===

Use a php include to add the file acpt.php to your plugin or functions.php theme file. For more advanced users look at the code comments for help on what args are avalible.

Troubleshooting
===

Slug
---

If your slugs are not working be sure you have flushed the permalink rules. To do this go to the permalinks and save the settings. No need to mod the .htaccess file if told.

Making a Custom Post Type
===

Advanced Users See: post_type.php

Making post types with ACPT is fast and easy. The post_type class takes up to 4 arguments (only the first two are required). First the singular name and then the plural name of your post type (makes these lowercase). The next is for capabilities. If you don’t know how capabilities work set this to false and everything should work expected (the default, false, is the same as posts capabilities). Set capabilities to true to create custom capabilities using the post types name (see roles for advanced usage). Last, you have the settings argument. This is used if you want to change the default settings or override them. Use the settings argument the same as you would for creating post types using Wordpress building registration method.

```php
<?php
include('acpt/acpt.php');

add_action('init', 'makethem');
function makethem() {
    $args = array(
        'taxonomies' => array('category', 'post_tag'),
        'supports' => array( 'title', 'editor', 'page-attributes'  ),
        'hierarchical' => true,
    );

    $books = new post_type('book','books', false,  $args );

}
```

Making a Taxonomy
===

Advanced Users See: tax.php

Making taxonomies with ACPT is fast and easy. The tax class takes up to 6 arguments (only the first 2 are required). First the singular name and then the plural name of your taxonomy (makes these lowercase). Third, you list have post types in an array (you can also set this in the post type itself, I recommend this way). Fourth, hierarchy. Set hierarchy to true if you want to allow the taxonomy to have descendants (the default, false). The last is for capabilities. If you don’t know how capabilities work set this to false and everything should work expected (the default, false). Set capabilities to true to create custom capabilities using the taxonomies name (see roles for advanced usage). Last, you have the settings argument. This is used if you want to change the default settings or override them. Use the settings argument the same as you would for registering taxonomies using Wordpress building registration method.

```php
<?php
include('acpt/acpt.php');

add_action('init', 'makethem');
function makethem() {
    $colors = new tax('color','colors', null, false );
}
```

Roles
===

Advanced Users See: role.php

Roles are the most powerful part of ACPT. You can make(), update() and remove() with the role class. When working with roles in ACPT you need to understand how roles work in Wordpress to keep your site installation working smoothly. Unlike Taxonomies and Post Types, when roles are made they are added to the DB. This means you only need to run role code once for it to work. It is best to run this code on theme switching or plugin activation. You can get away with running the code once other ways but this is a common way to do so without a UI.

For basic usage be sure you know how switching themes and activating plugins work.

Themes: http://www.krishnakantsharma.com/2011/01/activationdeactivation-hook-for-wordpress-theme/
Plugins: http://codex.wordpress.org/Function_Reference/register_activation_hook

WARNING: You should not work with roles unless you know what you are doing. Also, Be sure you consider a plan of attack for when your theme or plugin is removed or deactivated. Using roles is ment for advanced users only.

Make Arguments
---

You can set the first argument with capital letters. Formatted name is suggested.

```php
<?php
// Bad code, don't do this
include('acpt/acpt.php');
add_action('init', 'makethem');
function makethem() {
    $r = new role();
    $r->make('Library Manager', array('read'), array('book', 'books'));
    $r->update('Administrator', null, null, array('book','books'));
}
```

Meta Boxes
===

Advanced Users See: meta_box.php

You can now add Meta Boxes with ACPT. The meta_box class takes up to 3 arguments (only the first is required). First the name of the meta box. Second, the post types you want to use. Last any settings you want to override (priority for example). You can add custom meta boxes to you post types by adding the name of the meta box to the post types supports arg or by applying the post type within the make function. To add HTML/PHP to the meta box create a function beginning with "meta_" and append the name of the field to the end of it.

```php
<?php
include('acpt/acpt.php');

add_action('init', 'makeThem');
function makeThem() {

    $argsCourse = array(
        'supports' => array( 'title', 'editor', 'page-attributes', 'details' ),
        'hierarchical' => true,
    );

    $argsBook = array(
        'supports' => array( 'title', 'editor', 'page-attributes' ),
        'hierarchical' => true,
    );

    $courses = new post_type('course','courses', false, $argsCourse );
    $books = new post_type('book','books', false, $argsBook );

}

add_action( 'add_meta_boxes', 'addThem' );

function addThem() {
    new meta_box('Details', array('book'));
}

// Note: forms API explained below
function meta_details() {
    $form = new form('details');
    $form->text('name');
}
```

Forms
===

Advanced Users See: form.php

You can now make Forms with ACPT. Please see the code for how to use this section. You will need to modify for best results. Plus I don't have time to document it right now. The meta box section has the code example you need.

Forms API also come with a dev mode. Set third arg to true.

```php
<?php
function meta_details() {
    // name, options, dev-mode
	$form = new form('details', null, true);
	$form->text('name');
	$form->textarea('address');
	$form->select('options', array('1', '2', '3'));
	$form->radio('options', array('1', '2', '3'));
    $form->editor('custom');
}
```

Together: Post Type and Taxonomy
===

Here is an example of how to work with Post Types and Taxonomies together.

```php
<?php
include('acpt/acpt.php');

add_action('init', 'makethem');
function makethem() {

    $args = array(
        'supports' => array( 'title', 'editor', 'page-attributes'  ),
        'hierarchical' => true,
    );

    $books = new post_type('book','books', false,  $args );
    $courses = new post_type('course','courses', false,  $args );

    new tax('color', 'colors', true,  array($books));
    new tax('author', 'authors', true, array($books, $courses) );

}
```

Together: Post Type, Meta Box, Form and Taxonomy
===

This is still not fully tested and needs a lot of security work. Use at your own risk.

```php
<?php
include('acpt/acpt.php');

add_action('init', 'makeThem');
function makeThem() {

    $args = array(
        'supports' => array( 'title', 'editor', 'page-attributes'  ),
        'hierarchical' => true,
    );

    $books = new post_type('book','books', false,  $args );
    $courses = new post_type('course','courses', false,  $args );

    new tax('color', 'colors', 'book', true);
    new tax('author', 'authors', array($books, $courses), true );

}

add_action( 'add_meta_boxes', 'addThem' );

function addThem() {
    new meta_box('Details', array('book', 'course'));
}

function meta_details() {
    $form = new form('details', null);
    $form->text('name');
    $form->textarea('address');
}
```

Contributors
===

- Gina Guerrero https://twitter.com/mnemyx
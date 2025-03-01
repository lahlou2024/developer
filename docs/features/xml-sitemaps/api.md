---
id: api
title: "Yoast SEO XML Sitemaps: API documentation"
sidebar_label: API
description: The API to change the XML sitemap Yoast SEO puts out.
---

Whilst Yoast SEO provides sensible default behaviors for XML sitemaps (and UI controls for inclusion/exclusion), custom themes or plugins sometimes need to alter our markup or logic.
In those cases, you can use the examples below to modify how our sitemaps are generated and output.

## Excluding content types

### Exclude specific posts
```php
/**
 * Excludes posts from XML sitemaps.
 *
 * @return array The IDs of posts to exclude.
 */
function exclude_posts_from_xml_sitemaps() {
    return [ 1, 2, 3 ];
}

add_filter( 'wpseo_exclude_from_sitemap_by_post_ids', 'exclude_posts_from_xml_sitemaps' );
```

### Exclude a post type
```php
/**
 * Exclude a post type from XML sitemaps.
 *
 * @param boolean $excluded  Whether the post type is excluded by default.
 * @param string  $post_type The post type to exclude.
 *
 * @return bool Whether a given post type should be excluded.
 */
function sitemap_exclude_post_type( $excluded, $post_type ) {
    return $post_type === 'recipes';
}

add_filter( 'wpseo_sitemap_exclude_post_type', 'sitemap_exclude_post_type', 10, 2 );
```

### Exclude a taxonomy
```php
/**
 * Exclude a taxonomy from XML sitemaps.
 *
 * @param boolean $excluded Whether the taxonomy is excluded by default.
 * @param string  $taxonomy The taxonomy to exclude.
 *
 * @return bool Whether a given taxonomy should be excluded.
 */
function sitemap_exclude_taxonomy( $excluded, $taxonomy ) {
    return $taxonomy === 'ingredients';
}

add_filter( 'wpseo_sitemap_exclude_taxonomy', 'sitemap_exclude_taxonomy', 10, 2 );
```

### Exclude an author
```php
/**
 * Excludes author with ID of 5 from author sitemaps.
 *
 * @param array $users Array of User objects to filter through.
 *
 * @return array The remaining authors.
 */
function sitemap_exclude_authors( $users ) {
   return array_filter( $users, function( $user ) {
        if ( $user->ID === 5 ) {
            return false;
        }

        return true;
    } );
}

add_filter( 'wpseo_sitemap_exclude_author', 'sitemap_exclude_authors' );
```

### Exclude a taxonomy term
```php
/**
 * Excludes terms with ID of 3 and 11 from terms sitemaps.
 *
 * @param array $terms Array of term IDs already excluded.
 *
 * @return array The terms to exclude.
 */
function sitemap_exclude_terms( $terms ) {
    return [ 3, 11 ];
}

add_filter( 'wpseo_exclude_from_sitemap_by_term_ids', 'sitemap_exclude_terms' );
```

## Adding content

### Add a custom post type
```php
/**
 * Adds an XML sitemap for a custom post type.
 * 
 * @return void
 */
function enable_custom_sitemap() {
    global $wpseo_sitemaps;
    if ( isset( $wpseo_sitemaps ) && ! empty ( $wpseo_sitemaps ) ) {
        $wpseo_sitemaps->register_sitemap( 'recipe', 'create_recipe_sitemap' );
    }
}

add_action( 'init', 'enable_custom_sitemap' );
```

### Add additional/external XML sitemaps to the XML sitemap index
```php
/**
 * Writes additional/custom XML sitemap strings to the XML sitemap index.
 *
 * @param string $sitemap_custom_items XML describing one or more custom sitemaps.
 *
 * @return string The XML sitemap index with the additional XML.
 */
function add_sitemap_custom_items( $sitemap_custom_items ) {
    $sitemap_custom_items .= '
<sitemap>
<loc>https://www.example.com/external-sitemap-1.xml</loc>
<lastmod>2017-05-22T23:12:27+00:00</lastmod>
</sitemap>';
    return $sitemap_custom_items;
}

add_filter( 'wpseo_sitemap_index', 'add_sitemap_custom_items' );

```

### Add images to posts 

Some themes or page builder modules may not show the images on the sitemap. You may need to add them via a filter: `wpseo_sitemap_urlimages`. This filter will then register images to appear on the sitemap.

```php
/**
 * An example of adding images to posts.
 *
 * @param array $images An array of the images urls related to the post.
 * @param int   $post_id  The post ID.
 * 
 * @return array The array of images urls to add.
 */
function filter_wpseo_sitemap_urlimages( $images, $post_id ) {
  array_push( $images, [ 'src' => 'https://www.example.com/wp-content/uploads/extra-image.jpg' ]);
  return $images;
};
add_filter( 'wpseo_sitemap_urlimages', 'filter_wpseo_sitemap_urlimages' );
```

### Add images to terms 

Some themes, page builder modules or plugins may not show the images on the sitemap. You may need to add them via a filter: `wpseo_sitemap_urlimages_term`. This filter will then register images to appear on the sitemap.

```php
/**
 * An example of adding images to terms.
 *
 * @param array $images An array of the images urls related to the term.
 * @param int   $term_id  The term ID.
 * 
 * @return array The array of images urls to add.
 */
function filter_wpseo_sitemap_urlimages_term( $images, $term_id ) {
  array_push( $images, [ 'src' => 'https://www.example.com/wp-content/uploads/extra-image.jpg' ]);
  return $images;
};
add_filter( 'wpseo_sitemap_urlimages_term', 'filter_wpseo_sitemap_urlimages_term' );
```

### Add images to front page 

When the front page is not a static page, but the latest posts, you can add images to the sitemap via a filter: `wpseo_sitemap_urlimages_front_page`. This filter will then register images to appear on the sitemap.

```php
/**
 * An example of adding images to front page.
 *
 * @param array $images An array of the images urls related to the front page.
 * @return array The array of images urls to add.
 */
function filter_wpseo_sitemap_urlimages_front_page( $images ) {
  array_push( $images, [ 'src' => 'https://www.example.com/wp-content/uploads/extra-image.jpg' ]);
  return $images;
};
add_filter( 'wpseo_sitemap_urlimages_front_page', 'filter_wpseo_sitemap_urlimages_front_page' );
```

## Miscellaneous

### Alter the URL of an entry
```php
/**
 * Alters the URL structure for an example custom post type.
 *
 * @param string  $url  The URL to modify.
 * @param WP_Post $post The post object.
 *
 * @return string The modified URL.
 */
function sitemap_post_url( $url, $post ) {
	if ( $post->post_type === 'guest_authors' ) {
		return \str_replace( 'guest-authors', 'guests', $url );
	}

	return $url;
}

add_filter( 'wpseo_xml_sitemap_post_url', 'sitemap_post_url', 10, 2 );
```

### Alter the number of sitemap entries
```php
/**
 * Alters the number of entries in each XML sitemap.
 *
 * @return integer The maximum entries per sitemap.
 */
 function max_entries_per_sitemap() {
    return 100;
}

add_filter( 'wpseo_sitemap_entries_per_page', 'max_entries_per_sitemap' );
```

### Add extra properties to the `<video:video>` container
```php
/**
 * Adds an example <video:live> property to a <video:video> container for a particular post
 *
 * @param string $property A placeholder for a custom property.
 * @param int    $post_id  The post ID.
 * 
 * @return string The property to add.
 */
 function add_video_live_property( $property = '', $post_id ) {

    // Bail if this isn't the example post we want to modify.
    if ( $post_id !== 12345 ) {
        return $property;
    }

    // Set our custom property.
    $property = '<video:live>yes</video:live>';

    return $property;
}

add_filter( 'wpseo_video_item', 'add_video_live_property', 10, 2 );
```

### Filter the `urlset` element
For instance, if you want to use xhtml:link elements in your XML sitemap to add, for instance, hreflang markup, you could do the following:
```php
/**
 * Filters the `urlset` for all sitemaps.
 *
 * @param string $urlset The output for the sitemap's `urlset`.
 */
add_filter( 'wpseo_sitemap_urlset', function( $urlset ) { 
  return str_replace( '>', ' xmlns:xhtml="http://www.w3.org/1999/xhtml">', $urlset );
}, 1, 10 );
```

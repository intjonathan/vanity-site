---
title: "How to get character encoding right in Smarty"
date: 2007-10-21
author: Jonathan Owens
---

[Smarty](http://smarty.php.net/) is a great tool, but like its underlying PHP, has no native understanding of internationalization. One aspect of it that's easy to get wrong is character output encoding. Fortunately it's also fairly easy to get right.

HTTP useragents send a header with their requests named "[Accept-Charset](http://www.freesoft.org/CIE/RFC/2068/157.htm)". This header has a really stupid format, the parsing of which I'll address another time. For now, let's start from the point where you know what encoding the client prefers. Let's say it's something really weird like BIG-5. All your Chinese Smarty templates have text stored in UTF-8 because you're a good programmer. How do you get those lovely UTF-8 templates mangled into something this silly UA likes?

First, if you're even reading this post, you're probably aware of PHP's mbstring library. This library offers a lovely function called mb_convert_encoding. The basic idea is to attach an output filter to Smarty that runs the template output through mb_convert_encoding. Here's how this looks:

```php
/**
 * This sets an internal property in the PHP instance. Of course, this should be set to whatever the UA wants, within reason.
*/

mb_http_output( "BIG-5" );

function convertEncoding( $templateOutput, &$smarty )
{
    return mb_convert_encoding( $templateOutput, mb_http_output() );
}

$smarty->register_outputfilter( "convertEncoding");

/**
 * You must send headers so the browser knows that you gave it what it wanted
 */
header( "Content-Type: text/html; charset=".mb_http_output());
```

Now, when you call `display()` or `fetch()` on that Smarty instance, all its output will be converted to the current value of `mb_http_output()` - even if the file was already cached!

This approach guarantees a few nice things:

- It does not depend on output buffering
- It is insulated from changes in php.ini
- Only templates have their encoding converted
- The encoding can be converted on a per-request basis

So go get this right, and stop sending everybody ISO-8859-1.

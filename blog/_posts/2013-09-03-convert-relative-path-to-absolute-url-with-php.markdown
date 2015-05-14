---
title: "Convert Relative Path to Absolute URL with PHP"
layout: post
tags:
- php
- url
- absolute-url
- relative-path
---
I was developing a Web Crawler with PHP. But whenever I was crawling a url to get the links from that url, I found that most of the websites use relative path to link to their internal pages. The browser is smart enough to convert those relative paths to absolute url and request the correct page whenever those relative paths are clicked. But, as it seems, PHP does not have any inbuilt support to convert those relative paths to absolute urls.

Therefore, I decided to define a method to do the job. Here I am sharing my method of converting relative path to absolute url.

<br>

## The Base

The first thing to mention is the base url. The relative paths are, actually, relative to the base url. If the page contains any `<base>` tag with its `href` attribute set, the browser will treat that url as base url. Otherwise, the browser will treat the url of the current page as the base url.

Now first define a base url:

    $baseUrl = 'http://example.com/some/fake/path/page.html';

So we are assuming that we are currently on this page and this url is our base url as defined with `$baseUrl`.

<br>

## The Method

As the base url is set, its time to start defining the method:
    
    function absoluteUrl($relativeUrl, $baseUrl) {
        // some codes to execute
    }

Here we will pass the relative path as the `$relativeUrl` and the base url as the `$baseUrl`. I decided to pass each and every path I encounter while crawling. Therefore the path may be an absolute one. So we have to keep that in mind.

Now here is the body of the method:
    
    // Skip converting if the relative url like http://... or android-app://... etc.
    if (preg_match('/[a-z0-9-]{1,}(:\/\/)/i', $relativeUrl)) {
        return $relativeUrl;
    }
    
    // Treat path as invalid if it is like javascript:... etc.
    if (preg_match('/^[a-zA-Z]{0,}:[^\/]{0,1}/i', $relativeUrl)) {
        return NULL;
    }

    // Convert //www.google.com to http://www.google.com
    if(substr($relativeUrl, 0, 2) == '//') {
        return 'http:' . $relativeUrl;
    }

    // If the path is a fragment or query string,
    // it will be appended to the base url
    if(substr($relativeUrl, 0, 1) == '#' || substr($relativeUrl, 0, 1) == '?') {
        return $baseUrl . $relativeUrl;
    }

    // Treat paths with doc root, i.e, /about
    if(substr($relativeUrl, 0, 1) == '/') {
        return onlySitePath($baseUrl) . $relativeUrl;
    }

    // For paths like ./foo, it will be appended to the furthest directory
    if(substr($relativeUrl, 0, 2) == './') {
        return uptoLastDir($baseUrl) . substr($relativeUrl, 2);
    }

    // Convert paths like ../foo or ../../bar
    if(substr($relativeUrl, 0, 3) == '../') {
        $rel = $relativeUrl;
        $base = uptoLastDir($baseUrl);
        while(substr($rel, 0, 3) == '../') {
            $base = preg_replace('/\/([^\/]+\/)$/i', '/', $base);
            $rel = substr($rel, 3);
        }
        return $base . $rel;
    }

    // else
    return uptoLastDir($baseUrl) . $relativeUrl;

Now in the method you have found two another method `uptoLastDir()` and `onlySitePath()`. Here is those methods:
    
    // Get the root path from url
    // http://example.com/some/fake/path/page.html => http://example.com/
    function onlySitePath($url) {
        $url = preg_replace('/(^https?:\/\/.+?\/)(.*)$/i', '$1', $url);
        return rtrim($url, '/');
    }

    // Get the path with last directory
    // http://example.com/some/fake/path/page.html => http://example.com/some/fake/path/
    function uptoLastDir($url) {
        $url = preg_replace('/\/([^\/]+\.[^\/]+)$/i', '', $url);
        return rtrim($url, '/') . '/';
    }

<br>

## Usage

    echo absoluteUrl('/about', 'http://example.com/dir/');
    // Output: http://example.com/about

    echo absoluteUrl('dir.html', 'http://example.com/dir/page.html');
    // Output: http://example.com/dir/dir.html
    
    echo absoluteUrl('../../foo.html', 'http://example.com/dir/foo/bar/index.html')
    // Output: http://example.com/dir/foo.html
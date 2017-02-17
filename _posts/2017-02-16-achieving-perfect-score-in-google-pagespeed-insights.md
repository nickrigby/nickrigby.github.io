---
layout: post
title: "Achieving a Perfect Score in Google PageSpeed Insights"
date: 2017-02-16 22:00:00 -0500
categories: general
tags: [jekyll, github pages, performance, google] 
---
I've been thinking about performance a lot recently, especially since Google are apparently ranking "faster" sites higher in their  indexes. So, I decided to take a look into what it might take to get this site to the holy grail of 100/100 in the [Google PageSpeed Insights tool](https://developers.google.com/speed/pagespeed/insights/). With a few code adjustments I'm pleased to say I achieved it, so I wanted to share the things I did to get there.

## Disclaimer
Before I start, I want to mention that my site wasn't in bad shape before I started this quest — with a score of around 70 for mobile, and 80 for desktop. My site is also built with static files through [Jekyll](http://jekyllrb.com/), and I currently have zero images on the site, which put me in a really good starting position. I'm also using [Cloudflare](https://www.cloudflare.com), which is handling some of the advanced performance features like minifying HTML, browser cache expiration, and some general caching. If you're not using Cloudflare, I highly recommend it (they have a [free plan](https://www.cloudflare.com/plans/)).

Even if the things I'm about to share don't get you to 100, I'm hoping they might help improve your score.

## Custom Fonts
My site uses a single Google Font, Karla, and one of the issues reported by the insights tool was to "Eliminate render-blocking JavaScript and CSS in above-the-fold content". One of the reported offenders was the CSS import that I was using to include the font. After a little bit of searching around, I found the [Webfont Loader](https://github.com/typekit/webfontloader) library, which is co-developed by [Google](https://www.google.com) and [Typekit](https://typekit.com). The tool allows you to load fonts via JavaScript, which opens up the ability to load the scripts asynchronously, which gets around the render-blocking of content.

As a little bit of background, when scripts load synchronously (the default), rendering of the document pauses until that resource has been fully downloaded to the client. When you load asynchronously, the download starts in the background, while parsing and rendering of the rest of the document continues.

I added the following scripts to the head of my pages (before the CSS) to load the font using Webfont Loader:

{% highlight html linenos %}
{% raw %}<script>{% endraw %}
if(sessionStorage.fonts) {
    document.getElementsByTagName('html')[0].classList.add('wf-active');
}

WebFontConfig = {
    google: {
        families: ['Karla']
    },
    timeout: 2000,
    active: function() {
        sessionStorage.fonts = true;
    }
};
{% raw %}</script>{% endraw %}
{% raw %}<script src="//ajax.googleapis.com/ajax/libs/webfont/1.6.26/webfont.js" async></script>{% endraw %}
{% endhighlight %}

Here's an explanation of what's going on:

 - **Lines 2-4:** Because the fonts load asyncronously, there can be a brief flash of the fallback font before the custom font renders. This code checks to see if the custom font has been previously loaded in this session, and applies the class to the HTML element (which controls the custom font in the CSS), before the webfont script even runs.
 - **Line 6:** Start of the config object that will be passed to the Webfont script on line 16.
 - **Lines 7-9:** Specify which fonts to load. You can load fonts from Typekit, Google, Fonts.com, custom and more.
 - **Line 10:** Specifies a maximum time of 2 seconds for the fonts to load, otherwise the fallback fonts are used.
 - **Lines 11-13:** Creates the session storage variable that we check for on line 2.
 - **Line 16:** Asyncronously load the WebFont Loader library.

Next, the CSS:

{% highlight css linenos %}
body {
    font-family: sans-serif; // Fallback Font
}

.wf-active body {
    font-family: "Karla"; // Custom Google font Karla
}
{% endhighlight %}

When the custom font loads, a class of `wf-active` is added to the `html` element, which then matches the CSS rule on line 6. You can also now see why we have the `sessionStorage.fonts` variable check on line 2 from the scripts above.

With this change in place, the warnings regarding fonts being "render-blocking" should be gone.

## Inline CSS
The next issue is similar to the fonts issue, in that the CSS is also considered "render-blocking". The suggested solution is that you move critical "above the fold" CSS to inline styles (in a `style` tag), leaving the non-critical CSS in a linked CSS file. The idea here is that the inline CSS is instantly available (it doesn't need to be downloaded), so the site can start displaying as quickly as possible. You can then move the linked CSS file, containing "below the fold" styles to the footer of your site, before the closing `{% raw %}</body{% endraw %}`. Identifying as to what classifies as "above the fold" is down to your own discretion.

### Outputting inline styles in Jekyll
My site is using the [Minima theme for Jekyll](https://github.com/jekyll/minima), which is very light, so I moved the entire CSS file into a `{% raw %}<style>{% endraw %}` tag. To do this in Jekyll, you can use the `include_to_scssify` function as follows:

{% highlight liquid linenos %}
{% raw %}
<style>
{% capture include_to_scssify %}
    {% include main.scss %}
{% endcapture %}
{{ include_to_scssify | scssify | strip_newlines }}
</style>
{% endraw %}
{% endhighlight %}

_Note:_ In order for this to work, you will need to move your `main.scss` file to the `_includes` folder.

You should also set your SASS output to compressed, by adding the following to your `_config.yml` file:

{% highlight yaml linenos %}
sass:
  style: compressed
{% endhighlight %}

## Google Analytics
The last issue I faced was an issue with Google Analytics. The issue here is that the Google Analytics script is loaded from Google, and the browser cache settings are set to 2 hours. PageSpeed insights apparently doesn't like this, which is ironic being that both products are from Google.

Anyway, after a little digging around, I found [Google Analytics Light](https://github.com/jehna/ga-lite). This is a non-official Google implementation, but it was built for this very issue, and is a smaller, cachable subset of the default Google Analytics library. Installing it can be done as follows:

{% highlight html linenos %}
<script src="https://cdn.jsdelivr.net/ga-lite/latest/ga-lite.min.js" async></script>
<script>
var galite = galite || {};
galite.UA = 'UA-XXXXXX'; // Insert your tracking code here
</script>
{% endhighlight %}

Alright, now we're in business! Once I had implemented this last step, I received the 100/100 score in Google PageSpeed insights for both mobile and desktop, and the site does seem to be pretty darn snappy.

Let me know your thoughts, and if you have any other tips for achieving the perfect score.

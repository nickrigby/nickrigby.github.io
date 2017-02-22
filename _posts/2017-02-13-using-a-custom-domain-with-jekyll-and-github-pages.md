---
layout: post
title: "Using a custom domain with Jekyll and Github Pages"
date: 2017-02-13 16:00:00
categories: jekyll
tags: [github pages, jekyll]
---

By default, Github Pages have a `.github.io` domain, but wouldn't it be cool to use your own custom domain? Well my friend, it's your lucky day.

## Adding a custom domain in Github
Browse to your repository on Github, and go to the "settings" tab. Scroll about half way down the page until you see the "custom domain" box (within the Github Pages section).

Add your custom domain to the box and hit save. We're Done!

_Note:_ Github will automatically create a new file in your repository called `CNAME` which includes a single line containing your custom domain.
 
## Configuring DNS
We've told Github Pages about our custom domain, but now we need to configure the DNS so it points to our Github Pages site. Github Pages, rather conveniently, offers a [number of different ways to configure the DNS](https://help.github.com/articles/using-a-custom-domain-with-github-pages/). I'm going to continue with the [A record method](https://help.github.com/articles/setting-up-an-apex-domain/).

Using the DNS configuration manager for your domain, add two A records as follows:

{% highlight plaintext %}
Host: @
Points to: 192.30.252.153
{% endhighlight %}
{% highlight powershell %}
Host: @
Points to: 192.30.252.154
{% endhighlight %}

_Note:_ Because DNS can take some time to propagate, there may be a delay until the world can view your site using your domain. If you're on a Mac, you can [flush your DNS cache](https://www.igeeksblog.com/how-to-flush-dns-in-mac-os-x/) which should allow _you_ to see the changes straight away.

## Configuring Jekyll
Almost there. Now we need to instruct Jekyll to use your custom domain too. Thankfully, it only requires one simple change to your `_config.yml` file. Open the file and change the url key to your new domain:

{% highlight yaml %}
url: "http://yourdomain.com"
{% endhighlight %}

Push your changes to Github and you should now see your Github Pages site at your domain. Bam!

## Bonus level: Enabling SSL
If you want to use SSL with your domain – which is highly recommended – read on. Github Pages do not support SSL with custom domains, but there is a way — [Cloudflare](https://www.cloudflare.com). I won't go into detail about what Cloudflare does, all you need to know is it allows you to use SSL with your Github Pages domain — for free.

 1. Sign up for a Cloudflare account (if you don't already have one).
 2. Enter your domain name to initiate a scan of the DNS records.
 3. Once the scan has finished, hit the "Continue Setup" button.
 4. Review the DNS settings that CloudFlare found. Hit "continue".
 5. Choose the plan that you want to use with your site. The free plan is fine.
 6. Use the provided nameservers to update the DNS with your domain name provider e.g. GoDaddy.
 7. On the overview page, make sure SSL is set to "flexible".

_Note:_ Again, due to the way DNS works, changing your nameservers could take 24-48 hours to take effect.

Cloudflare will initially show the status of your domain as "pending". Eventually, once everything is set up and configured, the status will change to "Active".

### Always use HTTPS
Currently both HTTP and HTTPS protocols will work for your domain, but we really want everyone to see the HTTPS version. Fortunately, this is really easy using "Page Rules" in CloudFlare. Click on the "page rules" icon from within Cloudflare, and create a rule as follows:

{% highlight plaintext %}
If URL matches: http://*yourdomain.com/*
Then settings are: "Always Use HTTPS"
{% endhighlight %}

Click "save and deploy" to make this change. This change may take a few minutes to take effect, but once it does, you should be able to browse to the HTTPS version of your site!

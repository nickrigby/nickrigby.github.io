---
layout: post
title: "Building a website with Jekyll and Github Pages"
date: 2017-02-12 22:30:00 -0500
categories: jekyll
tags: [github pages, jekyll, deployments]
redirect_from: "/magento/deployment/envoyer/2017/02/12/deploying-magento-2-with-composer-and-envoyer.html"
---
I've heard a lot of people talking about [GitHub Pages](https://pages.github.com) recently, so I decided to find out what the fuss was about. If you've been hiding under rock like me, Github Pages allow you to host a website directly from a Github repository — no separate hosting account necessary. It also integrates with [Jekyll](https://jekyllrb.com) — a simple, blog-aware, static site generator — so getting a website up and running couldn't be easier.

## Configuring Github Pages
Head over to the [GitHub Pages](https://pages.github.com) home page for detailed instructions, but essentially it boils down to three simple steps:

 1. Create a new repository in your Github account named `{% raw %}<username>.github.io{% endraw %}`. _Note:_ The `{% raw %}<username>{% endraw %}` part must match your Github username exactly for it to work. Take a look at [my Github Pages repository](https://github.com/nickrigby/nickrigby.github.io) as an example.
 2. Clone the repository to your own machine using your favorite git client.
 3. Create an `index.html` page and commit back to the master branch.

Once you have followed the above steps you should be able to browse to `yourusername.github.io` in a web browser and see your shiny new website. My site is [nickrigby.github.io](https://nickrigby.github.io).

You can also verify that everything is working correctly if you open the "settings" tab in your repository in Github. About half way down the page you should see the Github Pages section, where you can administer additional options. You'll notice that one of the options is "custom domain", which I covered in [another post]({{ site.baseurl }}{% post_url 2017-02-13-using-a-custom-domain-with-jekyll-and-github-pages %}).

## Installing Jekyll
If you don't already have a site, you should seriously consider installing [Jekyll](https://jekyllrb.com). In their own words "Jekyll is a simple, blog-aware, static site generator perfect for personal, project, or organization sites. Think of it like a file-based CMS, without all the complexity.". Yup, that'll do!

_Note:_ Be sure to check out the [Jekyll Requirements Page](https://jekyllrb.com/docs/installation/) prior to running these commands, to make sure you have all of the necessary dependencies.

{% highlight powershell linenos %}
gem install jekyll bundler
jekyll new my-awesome-site
cd my-awesome-site
bundle exec jekyll serve
{% endhighlight %}

If everything worked, you should be able to view your site at a localhost address like `http://127.0.0.1:4000/`. _Note:_ You'll need to run the `bundle exec jekyll serve` every time you want to start the server, or make changes to your `_config.yml` file (discussed below).

### Configuring Jekyll with Github Pages
In your shiny new Jekyll installation directory you should see a `Gemfile` at the root of your project. We need to make a couple edits to this page for the best compatibility with Github Pages. Open the file in your favourite editor and make two changes:

 1. Line 12(ish), comment out (or remove) `gem "jekyll", "3.2.1"`
 2. Line 19(ish), uncomment `gem "github-pages", group: :jekyll_plugins`

Take a look at [my Gemfile](https://github.com/nickrigby/nickrigby.github.io/blob/master/Gemfile) to see a complete example. Once these edits have been made, run the following:

{% highlight powershell linenos %}
gem update bundler
bundle update
bundle install
bundle update github-pages
{% endhighlight %}

 Jekyll and the Github Pages gem should now be ready to go go! Re-run the `bundle exec jekyll serve` command, to start the server.

### Configuring your development environment
Like any site, you will often require different configuration settings for production vs. development, and Jekyll is no different. Jekyll handles these settings via a simple [YAML](http://yaml.org) file found at the root of your project called `_config.yml`. We can create additional configuration settings for our development environment by adding a new file called `_config-dev.yml`.

By creating this file you can override values from the `_config.yml` file that are specific to the development environment, so you only need to override values that are different, rather than duplicating the whole file. Take a look at [default configuration file](https://github.com/nickrigby/nickrigby.github.io/blob/master/_config.yml) and [development configuration file](https://github.com/nickrigby/nickrigby.github.io/blob/master/_config-dev.yml) for comparison.

Once this file has been created, we need to tell Jekyll to use it. Remember the `bundle exec jekyll serve` command from earlier? We need to add some parameters to this command to tell Jekyll about our new development configuration.

{% highlight powershell %}
bundle exec jekyll serve --config _config.yml,_config-dev.yml
{% endhighlight %}

This new command now tells Jekyll to load both configuration files, with the second file overriding values in the first. Neat huh? Also, if you have a terrible memory like me, you might want to save this command as an alias in your bash profile, so it's easy to execute later. I added the following to my `.bash_profile` file:

{% highlight bash %}
alias jekyll-serve="bundle exec jekyll serve --config _config.yml,_config-dev.yml"
{% endhighlight %}

Now you can just run `jekyll-serve` from your project folder.

## Launch that sucker!
Alright, that's it! Commit all of your changes, sit back and admire your sweet new website. I hope you'll agree that using Github Pages and Jekyll makes it really easy to build a fully functional, hosted website in literal minutes. I hope you found this post useful, and if you have any questions or comments, please let me know below.

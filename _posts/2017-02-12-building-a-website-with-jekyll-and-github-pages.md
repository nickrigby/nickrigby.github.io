---
layout: post
title:  "Building a website with Jekyll and Github Pages"
date:   2017-02-12 22:30:00 -0500
categories: jekyll github
excerpt: A guide to creating a website using Jekyll, and hosting with Github Pages.
comments: true
---
I've heard a lot of people talking about [GitHub Pages](https://pages.github.com) recently, so I decided to find out what all the fuss was about. If you've been hiding under rock like me, Github Pages allow you to host a website directly from a Github repository — no separate hosting account necessary. So basically, free hosting!

## Configuring Github Pages
Head over to the [GitHub Pages](https://pages.github.com) home page for detailed instructions, but essentially it boils down to three simple steps:

 1. Create a new repository in your Github account named `{% raw %}<username>.github.io{% endraw %}`. _Note:_ The `{% raw %}<username>{% endraw %}` part must match your Github username exactly for it to work. You can [view my Github Pages repository](https://github.com/nickrigby/nickrigby.github.io) as an example.
 2. Clone the repository to your own machine using your favorite git client. The clone command should look something like this: `git clone https://github.com/username/username.github.io`
 3. Create an `index.html` page and commit back to the repository master branch.

Once you have followed the above steps you should be able to browse to `yourusername.github.io` in a web browser and see your shiny new website. You can see my site here: [nickrigby.github.io](https://nickrigby.github.io).

You can also verify that everything is working correctly if you open the "settings" tab in your repository in Github. About half way down the page you should see the Github Pages section, where you can see if everything is working, and also administer some options. You'll notice that one of the options is "custom domain", which I'll come to later in this post.

## Installing Jekyll
If you don't already have a site, you should seriously consider installing [Jekyll](https://jekyllrb.com). Jekyll is very cool [NoSQL](https://en.wikipedia.org/wiki/NoSQL) framework written in Ruby.

_Note:_ Be sure to check out the [Jekyll Requirements Page](https://jekyllrb.com/docs/installation/) prior to running these commands, to make sure you have all of the necessary dependencies.

{% highlight powershell linenos %}
gem install jekyll bundler
jekyll new my-awesome-site
cd my-awesome-site
bundle exec jekyll serve
{% endhighlight %}

If everything worked, you should be able to view your site at a localhost address like `http://127.0.0.1:4000/`.

### Configuring Jekyll with Github Pages
In your shiny new Jekyll installation directory you should see a `Gemfile` at the root of your project. We need to make a couple edits to this page for the best compatibility with Github Pages. Open the file in your favourite editor and make two changes:

 1. Line 12(ish), comment `gem "jekyll", "3.2.1"`. _Note:_ The version number may differ depending on when you installed Jekyll.
 2. Line 19(ish), uncomment `gem "github-pages", group: :jekyll_plugins`

Once these edits have been made, run the following:

{% highlight powershell linenos %}
gem update bundler
bundle install
bundle update github-pages
{% endhighlight %}

Jekyll and the Github Pages gem should now be ready to go go!

### Configuring your development environment
Like any site, you will often require different configuration settings for production vs. development, and Jekyll is no different. Jekyll handles these settings via a simple [YAML](http://yaml.org) file found at the root of your project called `_config.yml`. We can create additional configuration settings for our development environment by adding a new file called `_config-dev.yml`.

By creating this file you can override values from the `_config.yml` file that are specific to the development environment. The nice thing about this approach is that you only need to override values that are different, rather than duplicating the whole file.

Once this file has been created, we just need to tell Jekyll to use it. Remember the `bundle exec jekyll serve` command from earlier? We need to add some paramters to this command to tell Jekyll about our new development configuration.

{% highlight powershell %}
bundle exec jekyll serve --config _config.yml,_config-dev.yml
{% endhighlight %}

This new command now tells Jekyll to load both configuration files, with the second file overriding values in the first. Neat huh? Also, if you have a terrible memory like me, you might want to save this command as an alias in your bash profile, so it's easy to execute later. I added the following to my `.bash_profile` file:

{% highlight bash %}
alias jekyll-serve="bundle exec jekyll serve --config _config.yml,_config-dev.yml"
{% endhighlight %}

Now you can just run `jekyll-serve` and that complete command will run.

### Using a custom URL
By default, your new Github Pages site has a .github.io domain, but it's possible to add a custom URL to use your own domain name.

 1. In the settings tab for your repository on Github, add your custom domain in the Custom domain box about half way down the page.
 2. Add the domain to the "url" key in your `_config.yml` file e.g. `url: "https://nickrigby.uk"`
 3. Follow the [custom domain instructions](https://help.github.com/articles/using-a-custom-domain-with-github-pages/) to configure the DNS depending on what kind of domain you are using.

 It's also possible to force SSL for a custom domain, which I'll cover in a future post.

## Launch that sucker!
Alright, that's it! Using Github Pages and Jekyll makes it really easy to build a fully functional, hosted website in literal minutes. I hope you found this post useful, and if you have any questions or comments, please let me know below.

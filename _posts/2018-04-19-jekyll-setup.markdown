---
layout: post
title:  "devlog = github + jekyll"
date:   2018-04-19 20:50:42 -0700
categories: jekyll update
---
## First, Hello!
For years, I've kept a devlog while working on my side projects at home.  Not
for posterity's sake, but because I only code a couple hours a night and
I tend to get side tracked by the shiny-new.  ...so, I need notes to help me
remember the lessons I once learned and have since forgotten.  

My devlog is typically pretty obtuse; mis-typed notes, screen shots and code
snippets.  Since I'm going to try to keep my notes in this repo now, I will
try to also keep them cleaner, so that they might be of use to other poor souls
who spend way too much time googling for answers.

## Now, some notes...

### I followed this [tutorial to install Jekyll and dependencies][amanda-visconti]:  



Amanda includes detailed steps for both Mac and Windows; here are the steps, in brief, for **windows**:
#### 1. Install Chocolatey (package manager)
{% highlight powershell %}
@powershell -NoProfile -ExecutionPolicy unrestricted -Command "(iex ((new-object net.webclient).DownloadString('https://chocolatey.org/install.ps1'))) >$null 2>&1" && SET PATH=%PATH%;%ALLUSERSPROFILE%\chocolatey\bin
{% endhighlight %}

#### 2. Install Ruby
{% highlight dosbatch %}
choco install ruby -y
{% endhighlight %}

#### 3. Install Jekyll
{% highlight dosbatch %}
gem install jekyll
{% endhighlight %}


### I followed this [tutorial to build my first page][hyouk-seo]:

In brief, the steps I pulled out:
#### 1. Create a repo on GitHub
#### 2. Create your local Git repo
#### 3. Install Jekyll Bundler
{% highlight dosbatch %}
gem install jekyll bundler
{% endhighlight %}
#### 4. Create a new jekyll blog
{% highlight dosbatch %}
jekyll new <username>.github.io
{% endhighlight %}
#### 5. Serve the blog locally (port 4000)
{% highlight dosbatch %}
cd <username>.github.io
bundle exec jekyll serve
{% endhighlight %}
#### 6. Commit locally; push to Github 


Check out the [Jekyll docs][jekyll-docs] for more info on how to get the most out of Jekyll. File all bugs/feature requests at [Jekyll’s GitHub repo][jekyll-gh]. If you have questions, you can ask them on [Jekyll Talk][jekyll-talk].

[amanda-visconti]: https://programminghistorian.org/lessons/building-static-sites-with-jekyll-github-pages
[hyouk-seo]: https://medium.com/spemer/free-github-blog-and-hosting-with-jekyll-c24c408d158f
[jekyll-docs]: https://jekyllrb.com/docs/home
[jekyll-gh]:   https://github.com/jekyll/jekyll
[jekyll-talk]: https://talk.jekyllrb.com/

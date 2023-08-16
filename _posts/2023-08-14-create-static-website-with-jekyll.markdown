---
layout: post
title:  Static website creation on Jekyll with ease
date:   2023-07-01 23:31:51 +0200
categories: static websites
excerpt: "Exploring my transition from Medium to a personal Jekyll-powered blog. Take a look if you want to effortlessly create a static website. You will find all the basics to get familiar with this tool."
---
I've always enjoyed sharing my knowledge, thoughts, and ideas with others. As soon as I matured as a software engineer, I decided to create an account on Medium and start writing about topics that intrigued me and could be of interest to others.

However, nowadays more and more content becomes hidden behind the paywall. This led me to seek an alternative platform for my humble content.
Although I typically prefer ready-made solutions over reinventing the wheel, this time though
I felt the urge to redesign my personal website for the first time in ten years of its existence. The idea of creating a small static website with personal information and a blog seemed fitting.
Being an experienced Ruby (and Rails) developer, I was thrilled to leverage my acquired knowledge and finally craft a [blog in 15 minutes](https://www.youtube.com/watch?v=Gzj723LkRJY)! Just before writing the cherished `rails new blog`, I swiftly researched on out-of-the-box solutions for static websites. Why invest 15 minutes when merely 5 minutes could suffice with the aid of a specific tool? Of course, neither 5 nor 15 minutes can yield something truly meaningful, but you got the idea, right?

I formulated the next criteria:
1. **Lightweight and easy to use** \
Something, which allows to hit the ground running without spending too much time reading
documentation.


2. **Easy to host** \
I host my page on GitHub pages and ideally wanted to continue as this is more than enough for
my needs.


3. **Customizable** \
Maybe tomorrow I will add another page with a nice-looking gallery with images of unicorns. Who knows.


Once I crystallized these criteria, the answer poped out from my mind. I have read about
[Jekyll](https://jekyllrb.com) some time ago. It turned out it was a great candidate that met my needs. The bonus point was
that it is written in Ruby! Please don't drop this article if you are not familiar with Ruby as it turned out that besides Ruby itself
it only needed to install a gem and start the project:

{% highlight ruby %}
  gem install bundler jekyll
  jekyll new my-site
  cd my-site
  bundle exec jekyll serve --livereload
{% endhighlight %}

"Why choose Jekyll over writing plain HTML?" you may wonder. Because it supports some convenient features, which simplify development:
layouts, templates with Liquid, ability to create content not only in HTML, but also using Markdown and the easiest deployment
to GitHub pages. Let's go through everything in order.

### Pages

The most essential part of building a static website. They can be written in HTML or Markdown.
Some examples of pages: `index.html`, `blog.html`, `about.markdown`, etc. Just drop these files into
the root of the project. Also, pages can also be organized into subfolders, if some organization is required.

### Configuration

Basic configuration can be set in `_config.yml`. Sometimes it's needed to set a custom configuration for a specific page or post.
For such cases, Jekyll provides the tool called Front Matter. It can be used for setting predefined global variables like
title and layout, as well as any other custom variable you need.

To implement Front Matter, place YAML-formatted configurations at the top of the page, encased within three-dashed lines:


about.html

{% highlight ruby %}
---
layout: default
title: Custom title
header: About
---

<article>
  About me
</article>
{% endhighlight %}

### Includes

Repeatetive chunks of code can be extracted and organized within the `_includes` directory and included wherever needed.
Here are couple of examples of what make sense to move there: navigation, email subscription form, footer and so on.
Use `include` or `include_relative` directives to insert partial into the page.

{% highlight ruby %}
...
<body>
  <header>
    {% raw %}{% include nav.html %}{% endraw %}
  </header>
  ...
</body>
{% endhighlight %}

### Liquid templates

While reading the example above you might wonder about the usage of `{% raw %}{% ... %}{% endraw %}`. Indeed, this doesn't
look like a pure HTML.
And you are right: Jekyll uses open-source template language called Liquid.
If you're unfamiliar with it, all you need to know is that variables enclosed within double curly brackets, such as
`{% raw %}{{ page.title }}{% endraw %}` will be rendered on a page. The code situated between curly bracket and percent
`{% raw %}{% if author %}{% endraw %}` is not displayed on the page, but is used as control flow structure. More information
you can find in [the official documentation](https://shopify.github.io/liquid).

### Layouts

Serve for containing basic repetitive code for pages, wrap content from pages. Loading of all necessary css and js files
is done there. For instance, consider the following layout template:

\_layouts/default.html

{% highlight html %}
<!DOCTYPE HTML>
<html>
<head>
  <title>{% raw %}{{ page.title }}{% endraw %}</title>
  <link rel="stylesheet" href="/assets/css/styles.css">
</head>
<body>
  <header>
    {% raw %}{% include navigation.html %}{% endraw %}

    <h2>{% raw %}{{page.header}}{% endraw %}</h2>
  </header>

  {% raw %}{{ content }}{% endraw %}
</body>
</html>
{% endhighlight %}

The content of a page will be rendered in place of `{% raw %}{{ content }}{% endraw %}` variable.


### Variables

Any file that includes Front Matter, is processed by Jekyll, allowing to use variables in your templates.
Here are main types of variables:

1. **site** \
This variable contains configuration elements sourced from `_config.yml` as well as global information.
Here are example of what can be accessed through it: `site.data` – data loaded from the `_data` directory,
`site.pages`, `site.posts`, `site.<config>` and so on.

2. **paginator** \
Is used for simplifying work with pagination: `paginator.page`, `paginator.total_pages`, `paginator.next_page`, etc.

3. **page** \
Is used for storing page-specific information. Any data specified via Front Matter in a given page resides within this variable.

about.html

{% highlight ruby %}
---
title: Custom title
header: About
---

<article>
  About me
</article>
...
{% endhighlight %}

Later specified variables can be accessed like so:

\_layouts/default.html

{% highlight html %}
<html>
<head>
  <title> {% raw %}{{ page.title }}{% endraw %} </title>
  ...
</head>
<body>
  <h2>{% raw %}{{page.header}}{% endraw %}</h2>
</body>
{% endhighlight %}

### Data

You can create custom data files (in JSON, YAML, CSV, and TSV formats), which should be located in `_data` directory.
Later inside pages it can be accessed via `site.data` variable.
Let's think about when it may be useful.
Consider a situation where you wish to incorporate a special gallery featuring unicorns. By creating a dedicated data file
for these unicorn profiles, you can maintain a structured repository of details that can be nicely integrated into your pages.


\_data/unicorns.yml


{% highlight html %}
{% raw %}
  - name: Bob
    img: imgs/bob_unicorn.jpg
    about: Likes walking in the forrest.
  - name: Patricia
    img: imgs/patricia_unicorn.jpg
    about: Used to live in mountains.
  - name: Kyle Jr
    img: imgs/kyle_unicorn.jpg
    about: Eats grass and pine cones.
{% endraw %}
{% endhighlight %}

index.html

{% highlight html %}
{% raw %}
{% for unicorn in site.data.unicorns %}
  <div>
    <h1>{{ unicorn.name }}</h1>
    <p>{{ unicorn.about }}</p>
    <img src={{ unicorn.img }} \>
  </div>
{% endfor %}
{% endraw %}
{% endhighlight %}

### Posts

Jekyll offers a straightforward method for creating a blog. Just drop HTML or Markdown files into `_posts` directory. Files
should be named in a specific format: `YYYY-MM-DD-title.md`, or `YYYY-MM-DD-title.html`. At the top of the most, you can
also use Front Matter:

{% highlight yaml %}
---
layout: post
title:  Static website creation on Jekyll with ease
date:   2023-07-01 23:31:51 +0200
categories: static websites
---

Some **useful** content.
{% endhighlight %}

Now it's time to render posts.

blog.html

{% highlight html %}
{% raw %}
{% for post in site.posts %}
  <article>
    <a href={{ post.url }}>
      <h1>{{ post.title }}</h1>
      <p>{{ post.excerpt }}</p>
    </a>
  </article>
{% endfor %}
{% endraw %}
{% endhighlight %}

### Deployment

It turned out GitHub pages support Jekyll out of the box. Simply commit and push all your changes.
Everything else will handled by GitHub. Nice and easy!

\
That's all what I wanted to share today. Thank you for reading my post.
If you are interested in diving a bit deeper, I highly recomend to check out
[official documentation](https://jekyllrb.com/docs/).

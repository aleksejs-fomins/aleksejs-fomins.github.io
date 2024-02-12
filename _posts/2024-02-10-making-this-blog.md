---
title: "Making this blog"
categories:
  - Meta
tags:
  - Jekyll
  - Github
---

In this first post, I would like to give a detailed summary of the engineering behind this blog.

## Requirements

The hardest requirement in my opinion is the **simplicity of implementation**, as well as **ease of use**. As long as it is flexible enough for my needs, it makes sense to re-use existing functionality, and focus most of the efforts on the actual content. In terms of implentation, that means potentially avoiding any coding whatsoever, simply tuning an off-the-shelf solution. In terms of ease of use and transferability, it makes sense to separate content from implementation. 20 years ago, the simplest solution was to directly write the blog in HTML, or write ones own text parsers in JavaScript. Right now, tools like [markdown](https://www.markdownguide.org/getting-started/) allow the writing of blogs more or less in plain text.

Secondly, I lot of the topics of interest to me are science related, and thus benefit greatly from integration of inline **mathematical equations**. While, in theory, any equations can be rendered as images, having automatic parsing of latex equations in the browser is a game changer in terms of development time, and thus [mathjax](https://www.mathjax.org/) is a must.

$$E = mc^2$$

Thirdly, as much of my work is coding in various languages (mostly Python recently), it would make sense to be able to have inline **code snippets**.

Finally, eventually I would like the ability to itegrate **dynamic figures**, i.e. interactive dashboards and plots.

## Solutions

I have considered several solutions

Firstly, I have briefly considered fully online solutions, such as [WordPress](https://wordpress.com/). While the simplicitly of using such tools cannot be matched, I wanted a little more control over my blog. It is desirable to have direct access to the full sources of each blog post, so that it could be transfered to another medium (such as a personal website) in the future, should the need arise.

On the opposite end of the spectrum is creating one's own website and running some existing blog software on it. While certainly the most flexible solution (allowing, for example, a forum self-contained on a local machine), it is also the hardest in terms of the initial time investment, and also the most expensive. Thus, I decided to look for easier solutions first.

After quick googling, I have arrived at a solution called [static site generators](https://jamstack.org/generators/). Such tools take source documents written in markdown language and convert them into a tunable website, such as a blog. The resulting websites to not have a GUI. Instead, new pages are added by writing the corresponding markdown documents in one's favourite editor, and uploading them directly to the folder where the website is stored. Perhaps a nuisance for some, but coming from Latex background this is quite natural. Better yet, such static website tools are supported by [GitHub pages](https://pages.github.com/), allowing anyone with a github account to have a blog for free, simply by storing the website source in a corresponding github repository. There is a multitude of static site generators. I have briefly experimented with [Hugo](https://gohugo.io/) and [Jekyll](https://jekyllrb.com/), and settled on the latter, mostly because I had found a stumbled on a nice guide to help me with the setup process.

## Installation

Firstly, I needed to create a dummy website on [GitHub pages](https://pages.github.com/),. It is as simple as creating the repository with a very specific name on your github. The instructions on the official website are very clear. I had to wait for quite a bit for the correct URL to start showing the dummy website, maybe half an hour, before it was showing nothing. One suggestion I had found was to rewrite the dummy website in proper HTML. I did it, but I am not sure it was necessary, as the official guide uses plain text.

Next, I needed to install Jekyll on my home computer, download an example theme, test it, and upload it to the git repository. I followed a combination of steps on [this blog](https://medium.com/coffee-in-a-klein-bottle/creating-a-mathematics-blog-with-jekyll-78cdee0339f3), as well as the originall Jekyll installation guide. The guide uses the [minima](git clone https://github.com/jekyll/minima.git) theme, one of the simplest Jekyll themes. The guide mentioned that enabling mathjax was a bit of a hack, requiring the appending of a few extra script URLS in a defaults file. This suggestion no longer works. After some googling, I had found a solution that worked at the time of writing on [another blog](https://www.martinklein.co/2022/02/18/latex-in-jekyll.html).

I am doing all of my development in Ubuntu Linux. Here, in order to view the website, it is sufficient to navigate to the website location in the terminal, running the following command

```
bundle exec jekyll serve --livereload
```
followed by running the link from the terminal in a web browser. When editing pages in a text editor, one can simply refresh the page in the browser to see the edits. However, changes in the configuration files do not propagate immediately. One needs to stop the terminal job and restart it to see the results locally.

## Customization

Since customization of a theme is theme-dependent, it was helpful to select a nice theme first. I have looked at the themes highlighted in [this github post](https://github.com/topics/jekyll-theme), and selected the theme [minimal-mistakes](https://mmistakes.github.io/minimal-mistakes/about/) by Michael Rose. The quick-start guide in this page is extremely detailed and well-written, so I highly recommend going through it if this theme takes your fancy.

Since this is a different theme, I first needed to figure out how to enable Mathjax. After reading the documentation of the theme, I had found that external head/body scripts could be specified directly in the `_config.yml` file, which seemed to me far nicer than "hacking" some of the source files of the theme without fully understanding how it works. I tried to add the following edit to the config file, and it worked like a charm (needed to restart jekyll for it to take action).

```
# Scripts for mathjax
head_scripts:
  - https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.7/MathJax.js?config=TeX-MML-AM_CHTML
```

Here's a Gauss' law to celebrate this achievement

$$\iiint_V \nabla \cdot \vec{E}dV = \oint_S \vec{E}\cdot d\vec{S} $$






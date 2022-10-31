---
title:  "Creating a personal site in GitHub"
date:   2022-10-14 13:00:00 +0200
toc: true
toc_sticky: true
categories: personal
tags: personal portfolio development
---

I have been thinking about creating a personal site for a while and now that I have a little more time in my hands I decided to go a head and do it. The fastest way to get something done is to actually getting started on it :D.

The easiest option would be to use WordPress.com given my familiarity with the platform. Although it's a great way to get something done quickly I think I want to spend a little extra time and look at the options.

# Features

The main feature I want to have is version control and I want to separate the content from the visualization. I would like that editing the website feels similar to editing a program.

Considering that, the first thing to came to mind was to take a look at GitHub. It already has everything a developer needs and I am evry familiar with it. Let's head to [pages.github.com](https://pages.github.com/).

# Requirements

GitHub uses Jekyll to transform plain text into static website. Which sounds exactly like what we need. Instead of using html and css, I would like to write the website using Markdown which is like plain text with extra formatting.

* We need a GitHub account, in this case I will use `leodonethat`
* We create a repository name with that account
  * [New repository](https://github.com/new): `leodonethat.github.io`

Our repository is now available at https://github.com/leodonethat/leodonethat.github.io

Next we clone the repository in our local environment:

``` bash
$ git clone git@github.com:leodonethat/leodonethat.github.io.git
```

**Note:** if this is your first time using GitHub it's better to read [this doc](https://docs.github.com/en/get-started/getting-started-with-git/about-remote-repositories#cloning-with-ssh-urls) as you can't use your password anymore when cloning a repo from the terminal. Make sure to [create an ssh key pair](https://docs.github.com/en/authentication/connecting-to-github-with-ssh/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent) and setup the public key in your github account.

This is really cool! all page is already live in [leodonethat.github.io](https://leodonethat.github.io/). Empty... but live.

# First Commit
Let's do a quick change following the tutorial:

``` bash
cd leodonethat.github.io
echo "Hello World" > index.html
```

``` bash
git add --all
git commit -m "First commit"
git push -u origin main
```

We go back to our site [leodonethat.github.io](https://leodonethat.github.io/) and we will see the first "Hello World" reflecting the changes! Really really cool.

**Note:** GitHub pages will show the file index.html by default and it cases it doesn't exist it will show a rendered version of `README.md`. That's why we saw that one the first time we accessed the site and then the hello world after we created the html file.

Now we are all set! the only small detail left is adding the contents of our website 😬

---

# Custom Domain

As an extra bonus, what if we happen to have a custom domain at hand?

A couple of years ago, I snatched the domain `leo.sh`. Let's see if we can put it to a good use.

Luckily for us, there is a good guide for [custom domains in the GitHUb docs](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site). The part that we need is about [configuring a subdomain](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site) (to make it www.leo.sh).

* GitHub repository
* -> Settings
* -> Code and automation
* -> Pages
* -> Custom domain: `leo.sh`

The tricky part is to go to the DNS provider and make the changes there.
* Added `A` records pointing the apex domain to the IP addresses for GitHub Pages (`leo.sh` -> `185.199.108.153`, `185.199.109.153`, `185.199.110.153`, `185.199.111.153`).
* Addeed a `CNAME` record that pointed the subdomain to the github pages address (`www.leo.sh` -> `leodonethat.github.io`)


``` bash
$ dig leo.sh +noall +answer -t A
leo.sh.			300	IN	A	185.199.111.153
leo.sh.			300	IN	A	185.199.109.153
leo.sh.			300	IN	A	185.199.110.153
leo.sh.			300	IN	A	185.199.108.153

$ dig www.leo.sh +nostats +nocomments +nocmd
;www.leo.sh.			IN	A
www.leo.sh.		14400	IN	CNAME	leodonethat.github.io.
leodonethat.github.io.	3600	IN	A	185.199.109.153
leodonethat.github.io.	3600	IN	A	185.199.111.153
leodonethat.github.io.	3600	IN	A	185.199.110.153
leodonethat.github.io.	3600	IN	A	185.199.108.153
```

I am not sure which kind of magic GitHub does for the A record behind the scenes and my networking skills are a little basic so I will leave it at that. The important part is that it works and we can celebrate having our page served at `leodonethat.github.io` and accessible at `leo.sh` 🥳

---

# Jekyll for Content Management

Although it's fantastic to be able to deploy our site by pushing code to GitHub, that is not the final objective. At the end we want to have fully fledged website (even if it's still a very basic one). Instead of manually coding all the html and css needed for the visualization, we can make use of a framework and focus on content. Fortunately, GitHub already supports a framework that can create a static website from Markdown file. We can add a quick note here saying it's a static site because all its contents are generated before hand and served to the browser. There are no calculations happening in real time when we browse. This is not the case for a site like Google where the output depends on the input but it's very suitable for us at the moment.

Here is the official page about [GitHub Pages and Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/about-github-pages-and-jekyll).

If we want to use Jekyll locally and generate our site before pushing it to GitHub we will need:

* [Jekyll](https://jekyllrb.com/docs/installation/)
* [Bundler](http://bundler.io/) to install and run Jekyll
* [Ruby](https://www.ruby-lang.org/en/documentation/installation/) to install Bundler

I already had Ruby installed and went straight for Bundler. With a local installation to this user instead of trying to do it as root

``` bash
$ gem install bundler --user-install
```

``` bash
$ gem install jekyll --user-install
```

Go to the directory of your GitHub site, in my case

``` bash
 $ cd ~/GitHub/leodonethat.github.io
```

Create a new Jekyll site in the current directory (use --force if you are ok overwriting existing files)

``` bash
$ jekyll new --skip-bundle . --force
```

Then follow the GitHub instructions and edit the `Gemfile` plus the file `_config.yml` if there are any changes you want (title, description and theme of the site)

```` bash
$ bundle install
````

Once that's finished we can commit and push our new code!

``` bash
$ git add .
$ git commit -m "First commit with Jekyll"
$ git push -u origin main
```

Then we head to our website's address, in my case `https://leo.sh`.

One issue here is that I had an `index.html` for testing that GitHub since to be taking by default. Let's delete it.

``` bash
$ git rm index.html
$ git commit -m "Delete testing index.html"
$ git push -u origin main
```

We go back to the browser and we see the Jekyll theme in action! 🥳

We can now take [a look at the different themes](https://pages.github.com/themes/) and choose a new one.

I decided that I liked a theme that wasn't there 😅  and installed [minimal mistakes](https://mmistakes.github.io/minimal-mistakes/docs/quick-start-guide/) (I had to add the extra plugin `jekyll-include-cache` for it to work in GitHub pages).

+++
title = "Tutorial: Static sites for non-programmers"
slug = "tutorial-static-sites-for-non-programmers"
date = "2019-07-01"
categories = [ "archive", "notes" ]
tags = [ "0t30", "tutorial" ]
+++

Static sites are the blog flavour *du jour*, and they are likely here to stay. Comprised of HTML files made available by a server, these websites are ideal for personal pages or projects which have few contributors and do not need to be updated frequently. And, because they are not as "plug and play" as popular content management systems (CMS's) or website builders, they also lend a certain mark of geek cred -- albeit rapidly diminishing with the advent of user-friendly (the horror!) static site generators (SSGs).

[Five years late to the hype train](https://news.ycombinator.com/item?id=7747517), but still better late than never, I finally found myself with more reasons to ditch Wordpress than not. Enter Hugo and GitHub Pages.

## Reasons to migrate

There are many good things to be said about Wordpress: it offers a free, immediately publishable platform with extensive customizability via myriad extensions and add-ons. The content editor is intuitive and easy to use. Nevertheless, I had a couple of persistent and frustrating problems.

First, the plugins, themes and even site host (?) needed constant patching. Fall behind a few updates, and everything runs/loads noticeably slower. Updating the base theme, however, sometimes caused my custom CSS/HTML to stop working or revert. Additionally, perhaps related to or resulting from the frequent patches, Wordpress seemed quite susceptible to security breaches. I was alarmed by the number of malicious login attempts every day. Many plugin updates were released not because they introduced new functionality, but to patch an inadvertent backdoor.

A static website promised endless customizability that would remain, by definition, *static*. I would also have complete control and ownership over all content, and security would no longer be a concern. 

## Things a website needs

Prior to this, I hardly considered where my website was being hosted and how it was being served or secured. After some research, I gathered there are a few essential players involved in turning local content into a public page, namely:
* Domain name -- online url where the website can be viewed
* Web host -- server space where the code for the website exists, linked to the domain name
* Web publisher -- software that "builds" the website (renders all the HTML/CSS into viewable content), updates contents/pages, etc
* TLS (aka Transport Layer Security) -- security, authentication, encryption, the protocol for https

## Final choices and alternatives

I already purchased the domain name.

Since one motivation for moving to self-hosting was cost saving, I only looked at free hosts and publishers. Even among these, there were several options. Hugo is a [marginally faster option](https://forestry.io/blog/hugo-vs-jekyll-benchmark/) (innate perk of Go language), though I doubt I would notice the perfomrance boost. Since Hugo has no innate compatability with GitHub, [people tend to set it up with GitLab and Netlify](https://brainfood.xyz/post/20190518-host-your-own-blog-in-1-hour/). I went ahead and set it up with GitHub Pages anyway, because I was familiar with GitHub and both GitLab/Netlify gate some of their features behind a subscription.

## 1. Setting up the Hugo + GitHub Pages + AWS website

I followed [these](http://jasonlawrencewong.com/blog/setupghpages/) instructions as well as [Hugo's own guide](https://gohugo.io/hosting-and-deployment/hosting-on-github/) for hosting on GH Pages, on macOS Mojave 10.14.4.

### 1.1. Create GitHub repositories

Register for GitHub and [install git](https://pages.github.com/).

Create two new repositories. The first must be named `$USERNAME.github.io`, which is where we will keep all the HTML/CSS rendering instructions for our website. The second, `$HUGO_PROJECT`, will contain all the files we need for Hugo to generate the static site (i.e. the HTML files in our first repo) -- this includes posts written in markdown, any images, theme layout, etc. Choose a descriptive name that will identify this project to you; I named mine [hugo_blog].

### 1.2. Create Hugo site

[Install Hugo](https://gohugo.io/getting-started/installing/). On macOS, this is as easy as `brew install hugo` assuming you have Homebrew, otherwise [install Homebrew](https://brew.sh/) first.

Decide where you want to create a local repository for your hugo site. Navigate into that location and create a new hugo site named $HUGO_PROJECT   
	`hugo new site $HUGO_PROJECT`

This is a good time to add a theme. Hugo has a vast selection of really nice [themes](https://themes.gohugo.io/) that, once installed, will render a template site we can further customize. I liked [mainroad](https://themes.gohugo.io/mainroad/) because it most resembled the Ryan theme I was using on Wordpress. We can add this in the[themes] submodule (essentially a sub-directory or sub-repo) under $HUGO_PROJECT:   
	`cd $HUGO_PROJECT/themes`    
	`git clone https://github.com/vimux/mainroad`    
	`cd ..`

Initialize our local $HUGO_PROJECT repository with `git init`

Check that our site looks correct by running the [hugo server] command. This launches a local server at **http://localhost:1313** in the browser. Once satisfied, we can kill the server with `[ctrl]+[c]`.

### 1.3. Connect local code base to GitHub repositories

Push the current local repo (code base) to the GitHub $HUGO_PROJECT repo:   
	`git remote add origin https://github.com/$USERNAME/$HUGO_PROJECT.git`    
	`git add .`    
	`git commit -m "Commit local hugo_blog to github hugo_blog"`

Hugo automatically generates HTML files for a Hugo site in a Public directory. We need these files to be generated in a $USERNAME.github.io local submodule under the $HUGO_PROJECT repo. Check that the local $HUGO_PROJECT repo contains a `config.toml` file and edit these flags (we can edit this config file later to further customize our site):   
	`baseURL = "https://$USERNAME.github.io"`    
	`publishDir = "$USERNAME.github.io"`

Create an empty $USERNAME.github.io submodule under the local $HUGO_PROJECT repo:   
	`git submodule add https://github.com/$USERNAME/$USERNAME.github.io.git`    
	`git add .`   
	`git commit -m "Commit new submodule for HTML files"`   
	`git push -u origin master`

Run `hugo` to generate HTML files for hugo_blog. Check that the $USERNAME.github.io local submodule is no longer empty, but now contains HTML files to be rendered on GitHub Pages.

### 1.4. Deploy and automate

Push local $USERNAME.github.io repo to deploy at https://$USERNAME.github.io:   
	`cd $USERNAME.github.io`    
	`git add .`   
	`git commit -m "Commit HTML files to GitHub Pages"`   
	`git push origin master`

Any time we want to update the website content, we need to add and/or make changes to content in the local $HUGO_PROJECT repo: for example, making a new post by adding a markdown file into the post submodule (we can decide how we want to organize the url directories by specifying slugs... but that's a future discussion). Then we have to run `hugo` to generate HTML files for the update into the local $USERNAME.github.io submodule. Finally we can push the changes to GitHub Pages. We can automate this process using a bash script `deploy.sh` which we run from the local $HUGO_PROJECT repo   

```bash
#!/bin/bash

echo -e "\033[0;32mDeploying updates to GitHub...\033[0m"

# Build the project.
hugo

# Go to publishing directory
cd $USERNAME.github.io

# Add changes to git.
git add .

# Commit changes.
msg="rebuilding site `date`"
if [ $# -eq 1 ]
	then msg="$1"
fi
git commit -m "$msg"

# Push source and build repos.
git push origin master

# Return to $HUGO_PROJECT root
cd .. 
```

Make this bash script executable with `chmod +x deploy.sh`

## 2. Migrating Wordpress blog content to Hugo site

Although is currently no Wordpress to Hugo plugin, the [wordpress-to-jekyll-exporter](https://github.com/benbalter/wordpress-to-jekyll-exporter) suffices for an easy preliminary transfer.

### 2.1. Converting posts to markdown

Once all our posts have been converted to markdown format, edit the front matter to be consistent with [Hugo format](https://gohugo.io/content-management/front-matter/). 

Of note: I don't have my Wordpress website backed up -- idiotic, yes, but I don't really know how to back up a website hosted remotely... which is another reason to switch to a static site, where I will (in theory) have total ownership and control of all the content.

## 3. Using a custom domain with GitHub Pages

Create a `CNAME` file in the local $USERNAME.github.io repository. The file should be named `CNAME` (no extension) and the contents should be the custom domain url.

Depending on where the domain was purchased, modify the DNS host settings ([with Namecheap, this is edited via the user dashboard](https://www.namecheap.com/support/knowledgebase/article.aspx/9645/2208/how-do-i-link-my-domain-to-github-pages)). Point the `www.$DOMAIN.TLD` to GitHub Pages and four additional address records pointing our apex domain to the following IP addresses:   

| Type			| Host 	| Value 				|
|---------------| :----:|-----------------------|
| CNAME Record 	| www	| #USERNAME.gitub.io 	|
| A Record		| @ 	| 185.199.108.153		|
| A Record		| @ 	| 185.199.109.153		|
| A Record		| @ 	| 185.199.110.153		|
| A Record		| @ 	| 185.199.111.153		|

It may take up to 24 hours for the DNS to update and configure.

Finally, under the settings of the remote (online) #USERNAME.github.io repository, make sure the custom domain is set correctly and check the **Enforce HTTPS** option.

## Further customization

Hugo is admittedly less-well documented than Jekyll, and I ran into multiple instances (after setting up the static site) where I had difficulty adding custom features. I am still struggling to understand the capabilities of Hugo; I found it annoying that widgets and reliability of shortcodes vary tremendously across templates.

I can tell that it is a powerful and versatile package -- the taxonomies, shortcodes, partials, and CSS overrides are probably game-changing in the hands of an expert. Unfortunately, because it is not as popular as Jekyll, there are fewer examples of custom code or functionality.

For now, it delivers on the basic promise: a simple, static, self-sufficient site, easy to set up and ready to go in half an hour.
---
title: Moving To Jekyll
date: 2019-08-29T19:17:13+01:00
author: Rich
permalink: /moving-to-jekyll
layout: post
categories:
  - personal
---

Last weekend I made a decision, after what feels like a lifetime having sites running on WordPress I am leaving the application behind! But why? 

I wanted something simple, easy to maintain and lightweight, I didn't want all the overhead that came with WordPress and managing a WordPress site, I wanted to focus more on writing technical documentation, don't get me wrong I know that WordPress is great for a lot of people but for me we just didn't suit each other anymore. 

## Where are you going

I decided to use Jekyll as my static site generator hosting by Netlify with the code residing in a GitHub Repo. 

## Installing Jekyll

I decided to install Jekyll on my Windows 10 desktop, this is where I do most of my writing and thought it was best suited here. This is how I went about it. (I followed the official Jekyll documentation) 

1. Download [Ruby+Devkit](https://rubyinstaller.org/downloads/) from the Ruby Installer downloads page
2. Run the <code>ridk install</code>
3. Open Command Prompt, cd to chosen directory and run <code>gem install jekyll bundler</code>
4. Check install was successful <code>jekyll -v</code>

### Creating the site

Once Jekyll was installed, I had to create a new site, this is pretty easy and is done with one line of code 

```
        jekyll new codenameowl
  ```

Replacing codenameowl with the name of your site

## The Prep

Now that we have 
Moving out of WordPress is quite involved, there are a number of steps that you need to do before you can flip the DNS switch. 

1. Export all your data from WordPress, for this I used [jekyll exporter](https://wordpress.org/plugins/jekyll-exporter/)
2. Move the images from the export directory into the Jekyll site

```
        $src = "C:\jekyll-export\wp-content\uploads"
        $dst = "C:\Development\Web Developments\CodeNameOwl\assets\img"

        Get-ChildItem -Path $src -Recurse -File | Copy-Item -Destination $dst    
  ```

3. Purge all of the thumbnail images
4. Resize the images so the long edge was no larger than 800px
5. Update all of the markdown files so the images pointed to /img/
6. Update all of the code blocks to use the formatting supported by [highlighter.js](https://highlightjs.org/usage/)

## Converting the theme

Over on WordPress I had written my own theme template which I wanted to bring with me, the majority of it was based around Bootstrap, the Jekyll documentation was really helpful in getting to understand how all the theme files fit together and I managed to get it converted on and off in a few days.

I didn't really want to throw lots of plugins at my site so I tried where possible to use as much vanilla code as possible, however one of the things I struggled with was getting the post date for individual posts, I found a solution over on GitHub which I ended up running with.

#### The Plugin

I created a file called myfilters.rb in the **_plugins** directory, inside I put the following;

```
    module Jekyll
        module MyFilters
            def file_date(input)
            File.mtime(input)
            end
        end
    end

    Liquid::Template.register_filter(Jekyll::MyFilters)
```

In the **_layouts** directory I have a post.html file, inside this I put the following code which gave me the post date

```
        <time>Posted: {{ page.date | date: "%B %-d, %Y"  }}</time>
  ```

For the code highlighting I decided to use [highlighter.js](https://highlightjs.org/usage/) it may not be the best solution and as I learn more about Jekyll I may change it but for now it works well.

## Creating the repo

I use [GitKracken](https://www.gitkraken.com) to manage my Git repositories so I created my repo for the site using it, once it was created I copied my Jekyll site into the new site directory and committed the initial change. 

## Hosting them images

One thing I couldn't quite get my head around was where I was going to host my images, in the end I just checked them into GitHub and Netlify served them directly from their CDN. Pretty neat! As traffic to my site is low I think this will be sufficient for a little while. 

## Setting up Netlify

There are loads and loads of guides out there on how to do this, I loosely followed [this one](https://www.chrisanthropic.com/blog/2018/migrating-blog-from-aws-stack-to-netlify/) which got me setup, I did some playing around in the dashboard to find my feet.

1. Signup for a Netlify Account
2. Connect GitHub
3. Select the GitHub Repo where your jekyll site lives
4. Select the branch you want to use for publication, I selected <code>master</code>
5. Netlify automatically detects that my site was a jekyll site and populated the build commands, the publish directory was also automatically populated.
6. Once you are happy select deploy site

#### DNS

I decided to switch my DNS into Netlify, just because it was easier, but there is documentation on how you can configure your existing DNS to point to your new blog, I had to wait about 12 hours for it to all switch over so don't worry if things don't happen right away.

#### SSL

The part that took the longest for me was getting an SSL, these are provided for free by [Let's Encrypt](https://letsencrypt.org) I had to wait for all TTL to expire on existing DNS records before Netlify could assign an SSL to my site, as I selected Netlify DNS this all seemed to happen automatically, when I checked a few hours after making the switch the SSL had been assigned.

## A small problem 

After I had setup Netlify and connected my account to the Repo in GitHub, the build process began on Netlify however it failed, after checking the logs I found this;

```
        7:20:53 AM: Warning: the running version of Bundler (2.0.1) is older than the version that created the lockfile (2.0.2). We suggest you upgrade to the latest version of Bundler by running `gem install bundler`.
  ```

At first I was a bit confused, I had just installed Jekyll fresh so wasn't 100% sure what was wrong, however it turns out the bundler I was using as the error message suggests is newer than the bundler in the image that is deployed by Netifly, this was easy to fix

1. Delete the Gemfile.lock
2. <code>gem uninstall bundler</code>
3. <code>gem install bundler -v 2.0.1</code>
4. cd to the jekyll directory and run <code>jekyll build</code> this should regenerate the Gemfile.lock file
5. Commit the changes to Git and Netlify should now build your site.

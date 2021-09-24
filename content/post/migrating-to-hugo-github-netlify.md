---
title: Migrating to Hugo, GitHub & Netlify
date: 2021-10-10T08:00:00+00:00
author: Rich
layout: ''
permalink: moving-to-netlify
categories:
- Personal
- Blog
tags:
- github
- netlify
- hugo
draft: true

---
# Contents

* Introduction
* Pre-Requisites
* Installing Hugo
  * Creating Your Site
    * Initilise with GitHub
    * Installing a theme
* 

## Introduction

## Pre Requisites

Before we get started, you are going to need a few things.

* GitHub Account
* Netlify Account
* Forestry Account (Optional)
* Content in markdown format

## Installing Hugo

I use a Windows desktop and the easiest way to install hugo is to use Chocolatey, if you have it installed you can simpley run

     choco install hugo -confirm -y  

If you don't have Chocolatey installed on your machine, you can follow the really [simple instructions](https://gohugo.io/getting-started/installing/) from the Hugo website to get Hugo installed on your machine

### Creating Your Site

Once Hugo is installed, open up a command prompt and navigate to the location you want your site to reside, for example

    D:\workspace

Once you are there, you can issue the following command to create a new site with Hugo

    hugo new site {site name}

Hugo will then create a new folder with whatever you put in place of {site name}

#### Initilise the folder with GitHub

#### Installing a theme

Now that the site is setup and ready to go, you are going to need to install a theme

#### Netlify Hugo Configuration

Becuase we are hosting our site on Netlify, we need to tell Netlify what configuration we want to use, to do this you will need to create a file in the root of the site directory called **netlify.toml** within this file you will need the following;

\[build\]

publish = "public"

command = "hugo --gc --minify"

\[context.production.environment\]

HUGO_VERSION = "0.81.0"

HUGO_ENV = "production"

HUGO_ENABLEGITINFO = "true"

\[context.split1\]

command = "hugo --gc --minify --enableGitInfo"

\[context.split1.environment\]

HUGO_VERSION = "0.81.0"

HUGO_ENV = "production"

\[context.deploy-preview\]

command = "hugo --gc --minify --buildFuture -b $DEPLOY_PRIME_URL"

\[context.deploy-preview.environment\]

HUGO_VERSION = "0.81.0"

\[context.branch-deploy\]

command = "hugo --gc --minify -b $DEPLOY_PRIME_URL"

\[context.branch-deploy.environment\]

HUGO_VERSION = "0.81.0"

\[context.next.environment\]

HUGO_ENABLEGITINFO = "true"

#### dsda
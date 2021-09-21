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

#### Initilise the folder with Git Hub

#### Installing a theme

Now that the site is setup and ready to go, you are going to need to install a theme 
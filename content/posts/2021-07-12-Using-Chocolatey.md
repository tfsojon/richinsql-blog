---
title: Using Chocolatey
date: 2021-07-12T09:00:00+00:00
author: Rich
layout: post
permalink: /using-chocolatey
categories:
  - powershell
tags:
  - powershell
  - personal
---

One of the most time consuming tasks when configuring a new computer is installing all of the applications that I use on a day to day basis, I am sure that everyone has this very same problem, but recently I came accross a package manager for Windows called Chocolatey, I am sure this has been around for absoloutely ages, looking at the amount of packages that are available but for me it is still pretty new. 

Basically what it allows you to do is specify a list of applications (you can see what is available here https://chocolatey.org/packages) that you would like to install via powershell and it will go download them and then install them for you automatically. 

I have put the script that I use to install my applications below, the basic syntax for installing an application with chocolatey is; 

```
    choco install {appname}
```

but I didn't want to type that in 20 odd times, so I created a loop to go through all the applications I wanted to install one by one.

```
Write-Host -ForegroundColor Gray "Attempting to install Chocolatey"

try {

Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://chocolatey.org/install.ps1'))

} catch {
Write-Host -ForegroundColor Red "Installing Chocolatey failed"
}

#https://chocolatey.org/packages

$chocolatePackaging = 
@("vscode","firefox","1password","sql-server-management-studio","github-desktop","paint.net","microsoft-windows-terminal","azure-data-studio","git","slack","office-tool","sqlsearch","sql-server-2017","microsoft-edge","7zip","teracopy","notepadplusplus","treesizefree")

foreach ($chocolate in $chocolatePackaging) {

    Write-Host -ForegroundColor Gray "Attempting to install " $chocolate 

    try {
        choco install $chocolate -y
    }
    catch {
        Write-Host -ForegroundColor Red "Installing " $chocolate "failed"
    }
}
```

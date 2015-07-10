---
author: thanapol
layout: post
featimg: laravel/laravel_intro.jpg
title: Introduction to Laravel
date:   2015-07-10 00:00:00
categories: primetime
tags: laravel php 
---

# Introduction
The PHP framework for Web Developer

# Installation

###### Prerequisite
* PHP >=5.5.9 
  * (For OSX, we uses php 5.5.26 which is installed using [Homebrew](http://brew.sh/)) 
* [Composer](https://getcomposer.org/download/) - The PHP dependency management 

###### Install Laravel framework
* Reference - http://laravel.com/docs/5.1
* Note from the document:
  * make sure you alias **composer** before run the command
  
    ``` composer global require "laravel/installer=~1.1" ```
    
    ``` alias composer="php /usr/bin/composer.phar" ```
    
    or just use ``` php <path/to/composer.phar> global require "laravel/installer=~1.1" ```
    
###### Create Laravel Project
* Add ```~/.composer/vendor/bin``` to your PATH
* Run ```laravel new <project_name>```


###### Project Structure
```
<project>
 | - app/
 | - conf/
 | - public/ 
 | - resources/views
 | - storage/
 ```
* app/ - contains Model, Controller
* public/ - contains the index.php and other public files such as css, js, image.
* resources/views - contains Html files
* storage/ - contains the internal files

###### Run The Project
* ```cd <project>/public```
* ```php -S localhost:8888``` run the project using built-in php web service

###### Useful 3rd library
* http://image.intervention.io/getting_started/installation#laravel

###### Next
* Create Model, Database (Migration, Seeding)
* Create Controller
* Create View to CRUD the model
* Integrate with Angular 
  

---
layout: post
title: Switching a team to a framework
date: '2015-05-31 21:57:30'
---

We recently embarked on a project where we new that things were a little different than our normal project:

1. The project needed some rapid development to push it into production as soon as possible
2. Whilst rapid development was necessary, the project was going to be maintained ongoing by our small team
3. We had only a very small number of external dependancies such as other codebases to interface with
4. We had no external database administrators to maintain

Previously we as a team had not used a "full-stack" framework within a team, writing mainly vanilla PHP code, often stemming back to PHP4 era; it was time for change!

---

From a long hard review at the myriad of frameworks available we settled on [Laravel](http://laravel.com/), primarily due to the LTS that is coming with [Version 5.1](https://laravel-news.com/2015/05/laravel-announces-v5-1-will-be-lts/) and the huge support from the community that Laravel is currently receiving.

We are building a large scale CRUD application, with a number of external applications hooking into this application (most of which are under our control) to follow our business needs.

## Writing less code to do more

We found that using Laravel meant that we wrote less code overall to achieve more. Our application is built around 2 main models which are very similar to each other, but with minor differences in their relationships with other models. Using a framework has allowed us to build reusable parts of our application which can be shared across these 2 models.

## Improving our Coding Quality

Using Laravel (or generally a framework) has massively improved our teams coding quality. Whilst using a framework does not instantly mean that the code is good quality, it requires you to think a little more about where your code sits in the stack, how it interacts with other parts of the application, and the chance for reusability and whether this is likely to be required and therefore should abstracted. We found this extra time to think, along with the ability to write less code meaning that our application is far better quality than it would have otherwise been.

## Better Source Control

We have always struggled with keeping code maintainable, and managing our application in source control (especially as previously we had not kept the database in source control, and had to rely on diff-ing database exports to compare changes).

With using [Migrations](http://laravel.com/docs/5.0/migrations) within Laravel we are now able to keep our Local, Staging & Production environments correctly up to date, and manage any changes across a team.

## Packages

Having the huge number of packages available from the community both in the form of ["normal" composer packages](https://packagist.org/) and [Laravel specific composer packages](http://packalyst.com/) has meant that we can often use community provided code (and spend our time helping with these rather than re-developing our own) and rapidly develop our application.

---

Overall the use of a framework (and specifically Laravel) has meant that our team has improved code quality, provided us with better management over our application and massively helped us rapidly develop our latest application!
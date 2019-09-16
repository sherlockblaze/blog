---
title: Coding Under Unix -- Daemon Processes
tags:
  - Unix
  - Processes
  - Coding Under Unix
date: 2019-03-05
---

> All the summaries are from the book named **[Advanced Programming in the Unix](https://www.amazon.com/Programming-Environment-Addison-Wesley-Professional-Computing/dp/0201563177/ref=sr_1_fkmrnull_1?crid=2YVJXTV3JD1HC&keywords=advance+programming+in+unix&qid=1551765355&s=gateway&sprefix=advance+unix%2Caps%2C467&sr=8-1-fkmrnull)**.

### What Is Daemon Processes

**We say that a process running without a controlling terminal in the background is a daemon process.**

Daemons are processes that live for a long time. They are often started when the system is bootstrapped and terminate only when the system is shut down. And Unix systems have numerous daemons that perform day-to-day activities.


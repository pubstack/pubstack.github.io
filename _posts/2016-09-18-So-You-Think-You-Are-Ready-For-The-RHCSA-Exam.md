---
layout: post
title: "So You Think You Are Ready For The RHCSA Exam?"
author: "Finnbarr P. Murphy"
categories:
  - blog
tags:
  - engineering
  - redhat
  - rhcsa
favorite: true
commentIssueId: 15
refimage: '/static/rhcsa1.jpg'
---

I just want to share this amazing blog post from Finnbarr P. Murphy,
originally from the [Musing of an OS plumber](http://blog.fpmurphy.com/2016/09/so-you-think-you-are-ready-for-the-rhcsa-exam.html)
blog.

Again, this is awesome!

--

So you have studied hard, maybe even attended a week or two of formal training,
for the Red Hat Certified System Administrator exam and now you think you are
ready to take the actual examination.

Before you spend your money (currently $400) on the actual examination,
why not download this custom [CentOS 7.2 VM](http://fpmurphy.com/public/RHCSA_SampleTest_1.ova)
and attempt a real world test of your RHCSA skills.

This VM, which is in the form of an OVA (Open Virtualization Archive),
will work with VMware Workstation 10 or later. Sorry, but if you want
to use the VM in other environments, you are going to have to figure
out how to do so, no support will be forthcoming from me. You will also
need network access from your host system to the default public CentOS repos.

There are twelve (12) tasks that you need to complete in 90 minutes from VM
power up. Most, if not all, of these tasks will probably appear in the real
exam. But first you must fix a problem booting the operating system and get
past the lost root password before you can read the file /TASKS which contains
the twelve tasks that you most complete. Oh, and by the way, networking and
package management are broken. You will have to get networking and package
management working in order to install some necessary packages.

Just like in the real examination, no answers are or will be provided.

If you cannot correctly complete all the tasks in 120 minutes or less.
You are absolutely NOT ready for the actual RHCSA exam.

Good luck!

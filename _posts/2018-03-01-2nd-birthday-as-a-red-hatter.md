---
layout: post
title: "My 2nd birthday as a Red Hatter"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - tripleo
  - openstack
  - reflections

favorite: true
commentIssueId: 49
refimage: '/static/bday.gif'
---

This post will be about to speak about my experience working in TripleO as
a Red Hatter for the last 2 years.

<div style="float: left; width: 230px; background: white;"><img width="230px" src="/static/bday.gif" alt="" style="border:15px solid #FFF"></div>


In my 2nd birthday as a Red Hatter, I have learned about many technologies,
really a lot... But the most intriguing thing is that here you never stop
learning. Not just because you just don't want to learn new things, instead,
is because of the project’s nature, this project... TripleO...

<div style="float: right; width: 230px; background: white;"><img width="230px" src="/static/tripleo_logo.png" alt="" style="border:15px solid #FFF"></div>

TripleO (Openstack On Openstack) is a software aimed to deploy OpenStack
services using the same OpenStack ecosystem, this means that we will deploy
a minimal OpenStack instance (Undercloud) and from there, deploy our production
environment (Overcloud)...  Yikes! What a mouthful, huh? Put simply, TripleO
is an installer which should make integrators/operators/developers lives
easier, but the reality sometimes is far away from the expectation.

TripleO is capable of doing wonderful things, with a little of patience,
love, and dedication, your hands can be the right hands to deploy complex environments at easy.

One of the cool things being one of the programmers who write TripleO, from now
on TripleOers, is that many of us also use the software regularly. We are writing
code not just because we are told to do it, but because we want to improve it for our own purposes.

Part of the programmers' motivation momentum have to do with TripleO’s open‐source
nature, so if you code in TripleO you are part of a community.

<div style="float: left; width: 230px; background: white;"><img width="230px" src="/static/community.gif" alt="" style="border:15px solid #FFF"></div>

Congratulations! As a TripleO user or a TripleOer, you are a part of our community and
it means that you're joining a diverse group that spans all age ranges, ethnicities,
professional backgrounds, and parts of the globe. We are a passionate bunch of crazy
people, proud of this "little" monster and more than willing to help
others enjoy using it as much as we do.

Getting to know the interface (the templates, Mistral, Heat, Ansible, Docker,
Puppet, Jinja, ...) and how all components are tight together, probably is one of
the most daunting aspects of TripleO for newcomers (and not newcomers).
This for sure will raise the blood pressure of some of you who tried using TripleO
in the past, but failed miserably and gave up in frustration when it did not behave
as expected. Yeah.. sometimes that "$h1t" happens...

Although learning TripleO isn't that easy, the architecture updates,
the decoupling of the role services "compostable roles", the backup and restore
strategies, the integration of Ansible among many others have made great strides
toward alleviating that frustration, and the improvements continue through to today.

So this is the question...

![](/static/fast_to.png)

Is TripleO meant to be "fast to use" or "fast to learn"?

There is a significant way of describing software products, but we need to know what
our software will be used for... TripleO is designed to work at scale, it might be
easier to deploy manually a few controllers and computes, but what about deploying
100 computes, 3 controllers and 50 cinder nodes, all of them configured to be integrated
and work as one single "cloud"? Buum!.
So there we find the TripleO benefits if we want to make it scale we need to make it fast to use...

This means that we will find several customizations,
hacks, workarounds, to make it work as we need it.

The upside to this approach is that TripleO evolved to be super-ultra-giga
customizable so operators are enabled to produce great environments blazingly fast..

The downside, Jaja, yes.. there is a downside "or several". As with most things that
are customized, TripleO became somewhat difficult for new people to understand.
Also, it's incredibly hard to test all the possible deployments, and when a user does
non-standard or not supported customizations, the upgrades are not as intuitive as they need...

This trade‐off is what I mean when I say "fast to use versus fast to learn."
You can be extremely productive with TripleO after you understand how it thinks "yes, it thinks".

However, your first few deployments and patches might be arduous. Of course,
alleviating that potential pain is what our work is about. IMHO the pros are more than the
cons and once you find a niche to improve it will be a really nice experience.

Also, we have the TripleO YouTube channel a place to push video tutorials and deep dive sessions
driven by the community for the community.

For the Spanish community we have a 100% translated TripleO UI, go to https://translate.openstack.org
and help us to reach as many languages as possible!!!

<div style="float: left; width: 230px; background: white;"><img width="230px" src="/static/logo.png" alt="" style="border:15px solid #FFF"></div>

www.pubstack.com was born on July 5th of 2016 (first GitHub commit), yeah is my way of expressing
my gratitude to the community doing some CtrlC + CtrlV recipes to avoid the frustration of working
with TripleO and not having something deployed and easy to be used ASAP.

Pubstack does not have much traffic but it reached superuser.openstack.org, the TripleO cheatsheets
were on devconf.cz and FOSDEM, so in general, is really nice. When people reference your writings
anywhere. Maybe in the future can evolve to be more related to ANsible and openSTACK ;) as TripleO
is adding more and more support for Ansible.

<div style="float: right; width: 230px; background: white;"><img width="230px" src="/static/red_hat.png" alt="" style="border:15px solid #FFF"></div>

What about Red Hat? Yeahp, I have a long time speaking about the project but haven't
spoken about the company making it all real.
Red Hat is the world's leading provider of open source solutions,
using a community-powered approach to provide reliable and high-performing
cloud, virtualization, storage, Linux, and middleware technologies.

There is a strong feeling of belonging in Red Hat, you are part of a team, a culture and you are able to
find a perfect balance between your work and life. Also, having all people from all over the globe makes
a perfect place for sharing ideas and collaborate. Not all of it is good, i.e. Working mostly remotely
in upstream communities can be really hard to manage if you are not 100%
sure about the tasks that need to be done.

Keep rocking and become part of the TripleO community!

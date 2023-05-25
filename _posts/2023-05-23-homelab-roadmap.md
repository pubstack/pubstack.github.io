---
layout: post
title: "Homelab roadmap"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - draft
  - cloud
  - kubernetes
  - engineering
  - hobbies
hidden: true
favorite: true
commentIssueId: 90
refimage: '/static/homelab/homelab.jpg'
---


## BOM

| Item             | Quantity | Price| Total |Image|
|------------------|----------|------|-------|-----|
| Countertop       | 1        |d| asdf| ![](/static/homelab/01_materials/01_tablero.jpg =50x50) |
| T-slot 2020 profiles  | 12       |d| asdf| ![](/static/homelab/01_materials/02_t-slot.jpg =50x50) |
| 3 way connectors      | 8 |d| asdf| ![](/static/homelab/01_materials/03_3_way_connector.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/04_m3_4mmlong_screw.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/06_90_degree_connector.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/07_90_degree_connector.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/08_t-slot_nut.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/09_spax_2mm_screw.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/12_L_bracket_30X80X55X2.5MM_SIMPSON.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/13_corner.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/16_feet.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/17_shock.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/18_wheels_1inch_nylon.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/19_m10_hexagon_connector.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/21_washers.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/22_hinge.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/23_glass_doors_5mm.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/81_Adam_Hall_19inch_61535.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/91_shelve.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/92_shelve_150mm.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/93_power.jpg =50x50) |
| Text  |s|d| asdf| ![](/static/homelab/01_materials/95_RDA2U.jpg =50x50) |

{% assign galleryImages = "/static/homelab/01_materials/01_tablero.jpg|/static/homelab/01_materials/02_t-slot.jpg|/static/homelab/01_materials/03_3_way_connector.jpg|/static/homelab/01_materials/04_m3_4mmlong_screw.jpg|/static/homelab/01_materials/06_90_degree_connector.jpg|/static/homelab/01_materials/07_90_degree_connector.jpg|/static/homelab/01_materials/08_t-slot_nut.jpg|/static/homelab/01_materials/09_spax_2mm_screw.jpg|/static/homelab/01_materials/12_L_bracket_30X80X55X2.5MM_SIMPSON.jpg|/static/homelab/01_materials/13_corner.jpg|/static/homelab/01_materials/16_feet.jpg|/static/homelab/01_materials/17_shock.jpg|/static/homelab/01_materials/18_wheels_1inch_nylon.jpg|/static/homelab/01_materials/19_m10_hexagon_connector.jpg|/static/homelab/01_materials/21_washers.jpg|/static/homelab/01_materials/22_hinge.jpg|/static/homelab/01_materials/23_glass_doors_5mm.jpg|/static/homelab/01_materials/81_Adam_Hall_19inch_61535.jpg|/static/homelab/01_materials/91_shelve.jpg|/static/homelab/01_materials/92_shelve_150mm.jpg|/static/homelab/01_materials/93_power.jpg|/static/homelab/01_materials/95_RDA2U.jpg" | split: "|" %}
{% assign galleryTitle = "Materials gallery" %}
{% include block_gallery.html galleryTitle=galleryTitle galleryImages=galleryImages %}


![](/static/homelab_intro.jpg)

The 993 was much improved over, and quite different from its predecessor.
According to Porsche, every part of the car was designed from the ground up,
including the engine and only 20% of its parts were carried over from the
previous generation. Porsche refers to the 993 as "a significant advance,
not just from a technical, but also a visual perspective." Porsche's engineers
devised a new light-alloy subframe with coil and wishbone suspension
(an all new multi-link system), putting behind the previous lift-off oversteer
and making significant progress with the engine and handling, creating a more
civilized car overall providing an improved driving experience. The 993 was
also the first 911 to receive a six speed transmission.

![](/static/964_side.jpg)

The external design of the Porsche 993, penned by English designer Tony Hatter,
retained the basic body shell architecture of the 964 and other earlier 911 models,
but with revised exterior panels, with much more flared wheel arches, a smoother
front and rear bumper design, an enlarged retractable rear wing and teardrop mirrors.
A 993 was promoted globally via its role of the safety car during the 1994 Formula One season.

Next, you can find several resources related to the 993 internals.

Please feel free to submit any pull request to add more resources here if you find them useful.
New files added to the folder [/static/993/](https://github.com/pubstack/pubstack.github.io/tree/master/static/993)
will be added automatically to this post after
the commits are merged.

{% assign galleryImages = "/static/964_front.jpg|/static/964_rear.jpg|/static/964_rear2.jpg|/static/993_inside.jpg" | split: "|" %}
{% assign galleryTitle = "asdf" %}
{% include block_gallery.html galleryTitle=galleryTitle galleryImages=galleryImages %}

Test

{% assign galleryImages = "/static/993_inside2.jpg|/static/964_inside2.jpg|/static/964_inside.jpg" | split: "|" %}
{% assign galleryTitle = "fdsa" %}
{% include block_gallery.html galleryTitle=galleryTitle galleryImages=galleryImages %}

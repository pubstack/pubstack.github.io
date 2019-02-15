---
layout: post
title: "Converting AAX audiobooks to MP3"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - hobbies
commentIssueId: 33
refimage: '/static/aax.jpeg'
---

This is a quick guide to convert AAX files (DRMed audiobooks) to it's MP3 equivalent.

I just got 'The Phoenix Project' book from [amazon.es](https://www.amazon.es/Phoenix-Project-DevOps-Helping-Business/dp/0988262509),
it was on sale together with it's audible audiobook version..

The thing is that I don't want to install any additional
software and I just want to listen the MP3 version in my phone,
and no it isn't a smartphone.. And to keep a sort of backup
in something different than this mumbo-jumbo AAX stuff.

So this is a small copy/edit/paste recipe to convert the AAX files
to MP3... Working fine and really easy to achieve...

```
  git clone https://github.com/inAudible-NG/tables.git
  git clone https://github.com/KrumpetPirate/AAXtoMP3.git
  wget http://project-rainbowcrack.com/rainbowcrack-1.7-linux64.zip
  unzip rainbowcrack-1.7-linux64.zip
  mv AAXtoMP3/* tables/
  mv rainbowcrack-1.7-linux64/* tables/
  mv <your_aax_file_name>.aax tables/
  cd tables
  make
  ffprobe <your_aax_file_name>.aax
  # Get the "[aax] file checksum"
  ./rcrack . -h <your_checksum>
  # Get the activation bytes hex
  bash AAXtoMP3 <your_activation_bytes> <your_aax_file_name>.aax

  # It will be generated an audiobook with the MP3 files in it.
  # *-*-Enjoy-*-*

```

This is basically all you need to convert your AAX file to MP3.

And no, I won't share with you my copy of the
'The Phoenix Project' DeDRMed audiobook...

> Disclaimer:
>
> The purpose of this recipe is to be able
> to create backups of your audio books and be able to
> play them on other non DRM capable players. Please do
> not share or upload your DeDRMed files with anyone else.

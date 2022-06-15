---
layout: post
title: "Automated weekly reports"
author: "Carlos Camacho"
categories:
  - blog
tags:
  - management
favorite: true
commentIssueId: 63
refimage: '/static/1and1s/its-time-for-weekly-reports.jpg'
---

Remote work is a trend among
Senior or hi-performant roles.

![](/static/1and1s/its-time-for-weekly-reports.jpg)

In general when working remotely it might be hard to measure or be
accountable for all the work we/you usually do.

Some of these tasks can be among user escalations,
preparing development environments, reviews, writing code,
checking unit tests, verifying CI systems status,
meetings, have 1and1 with your associates among many others...

And its really easy to miss the track or a log about these
tasks we want to actually report.

In this particular case I will explain how to generate automated
reports with the tasks you have logged in a Trello
board using a google docs template, also we will fetch data from
Bugzilla, Launchpad and Storyboard.


### Resources
* The main [google doc](https://docs.google.com/document/d/1qh7vuC8vPTum_BItCm5O0c6DmyXGXK-NABBXaoNuRMM/edit)
we will use for this how-to.
* A code repository in [GitHub](https://github.com/ccamacho/gdocsreport)
  with the JS/HTML code used inside the google docs
  (its all integrated to the google docs, so is not mandatory to use this, just read the [README](https://github.com/ccamacho/gdocsreport/blob/master/README.md)).

![](/static/1and1s/00_google_docs_menu.PNG)

### How to

If you open the [google document](https://docs.google.com/document/d/1qh7vuC8vPTum_BItCm5O0c6DmyXGXK-NABBXaoNuRMM/edit) and click
"Main menu" -> "Scrum" you will see the following options:

* Create today's agenda.
* Create tomorrow's agenda.
* Create today's agenda with Trello items.
* Generate activity report.

### Create today's and tomorrow's agenda

These two options are really simple to explain, we will just
generate an enumerated text section to write anything we need.

![](/static/1and1s/01_empty_agendas.PNG)

You will need to grant permissions to the script, the scripts are
located in
**"Main menu" -> "Tools" -> "Script editor"**.

There you will be able to see the code we can run from this
google document (we will speak about the code later).

![](/static/1and1s/02_tomorrow_empty_agenda.png)

### Create today's agenda with Trello items.

In this case we will fetch the content of three lists in a Trello board.

In particular I have created an example Trello board for this tutorial

![](/static/1and1s/03_trello_dashboard.PNG)

The board in question is private now (you can set it up as public, but is
private to show how to configure the Key and Token for Trello, otherwise
you will not be able to access it).

When you click on the option **Create today's agenda with Trello items.**

![](/static/1and1s/04_index_create_agenda_with_trello_items.png)

We will generate **In the place you have the cursor** the agenda pulling the
content of the board lists.

![](/static/1and1s/05_index_create_agenda_with_trello_items.png)

As you can see we have in the generated agenda the following information
based on each list.
In this case we have three lists, **To Do**, **Doing**, and **Done**, and in
each list we have:

* Assignees
* Description.
* Tags.
* Link to the Trello card.

Now, to customize this, please **Create your own copy of the [google document](https://docs.google.com/document/d/1qh7vuC8vPTum_BItCm5O0c6DmyXGXK-NABBXaoNuRMM/edit)!!**
and proceed to edit it's content by clicking:

**"Main menu" -> "Tools" -> "Script editor"**

![](/static/1and1s/06_google_script_editor.PNG)

The section in which we will focus to customize are the following parameters.

![](/static/1and1s/07_google_script_parameters.PNG)

**You can update and modify the script as you wish**

If you open your board appending a **.json** in the URL you will fetch
the IDs of the lists we need to display.

For example in this example will be:

```
https://trello.com/b/L8t9rCOz/1and1.json
```

In this case the variables **TRELLO KEY**, and **TRELLO_TOKEN** will be needed if the board is private,
otherwise you won't need them at all.

```
var TRELLO_KEY = '8b164347d9cf6a1026b5d20dc8556620';
var TRELLO_TOKEN = '4a64be415e2f7d128d2543fbbddd2b8ffd33d3ead6921803668ca39fe715f5cd';
```

The following parameters are:

* **TRELLO_LIST_ID**: The IDs of the lists we will fetch the content.

* **TRELLO_TITLES**:  The titles we will add to the report (the order should match, first list ID will be displayed under the first title and so on).

* **TRELLO_USER_FILTER**: Filter the cards based on the assignee name, here if this is a report, it might be a good idea to have your
manager name and your name. Use the name you have in Trello. (see the example)

```
var TRELLO_LIST_ID = ["5d20bd50eb63e24f7a0c8744", "5d20bd50eb63e24f7a0c8745","5d20bd50eb63e24f7a0c8746"]; //To Do, Doing, Done
var TRELLO_TITLES = ["To Do", "Doing","Done",]; //To Do, Doing, Done
var TRELLO_USER_FILTER = ["Carlos Camacho", "Pubstack"]; //Only display these people cards
```

You will need to generate a **Developer key** and a **Token**.

![](/static/1and1s/08_key.PNG)

Use the Key generated.

![](/static/1and1s/09_key_view.PNG)

Then you need to generate a token to be able to interact with Trello from the Google docs scripts.

![](/static/1and1s/10_token.PNG)

Copy the Token.

![](/static/1and1s/11_token_view.PNG)

Replace both **TRELLO KEY**, and **TRELLO_TOKEN** with the values you now have.

### Generate activity report.

This section will generate an activity report based on quarters, and this will have
a little bit more of information:

* Stackalytics.
* Launchpad.
* Storyboard.
* Bugzilla.

The information retrieved here is not private as is base in upstream metrics, so
you can just use mine as an example.

Basically based in the following configuration parameters:

```
/* Stackalytics variables */
var STACKALYTICS_USER= "ccamacho";
/* Bugzilla variables */
var BZ_HOST = "https://bugzilla.redhat.com";
var BZ_STATUS = "bug_status=NEW&bug_status=ASSIGNED&bug_status=POST&bug_status=MODIFIED&bug_status=ON_DEV&bug_status=ON_QA&bug_status=VERIFIED&bug_status=RELEASE_PENDING";
var BZ_EMAIL = "ccamacho%40redhat.com";
/* Launchpad variables */
var LAUNCHPAD_USER = "ccamacho";
/* Storyboard variables */
var STORYBOARD_USER = "3328";
```

Those parameters basically describe the data we will fetch, i.e. the BZ query, the BZ email and your Launchpad and Storyboard user ID.

Then when you generate the activity report you will filter by dates, in my particular case I based the reports on each quarter.

![](/static/1and1s/13_generate_quarterly_report_selection.png)

The code to configure the date filter is also stored in the google doc.

![](/static/1and1s/12_quarterly_report.png)

And then you can see the activity report generated correctly.

![](/static/1and1s/14_activity_report.png)

Again, if you don't trust to create a copy of the document
and use it directly because of the
google apps permissions requirement,
you can always get the code in [GitHub](https://github.com/ccamacho/gdocsreport)
and follow the [README](https://github.com/ccamacho/gdocsreport/blob/master/README.md)

This is a nice way of keeping up reported your contributions in the team in an easy and automated way.
Having a record of all the tasks you did is a really nice way of i.e. make a
yearly retrospective to share your achievements.

![](/static/1and1s/weekly_reports.jfif)

<div style="font-size:10px">
  <blockquote>
    <p><strong>Updated 2019/07/07:</strong> Initial version.</p>
    <p><strong>Updated 2019/07/08:</strong> Fixed some nits.</p>
  </blockquote>
</div>

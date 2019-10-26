---
title: "MPs commission attendances"
date: 2019-10-26T14:14:40+02:00
draft: false
---

## Scope of this post

### Article which inspired this post

I was triggered to write this post because of an [interesting article](https://multimedia.tijd.be/vlaamsparlement/) in *De Tijd* from some years ago which I recently stumbled upon again. [Maarten Lambrechts](<http://www.maartenlambrechts.com/](https://multimedia.tijd.be/vlaamsparlement/)>) investigated which members of the Flemish parliament are industrious and which are... less so. 

### Focus on commission attendances

To make this more concrete he looked at the number of initiatives and questions by parliamentarians in the Flemish Parliament. Based on this he was able to divide politicians in several categories according to their productivity. However I chose to approach the question of politician productivity from another angle. Instead of focusing on questions and initiatives I chose to focus on the commission attendances by MPs. On the one hand I chose to focus on attendances since it's a very binary variable (you're either there or you're not), on the other hand I chose to limit myself to the commission meetings. A large chunk of parliamentary work is done in the commissions which mostly remains invisible to the public.

## Why focus on attendances?

Originally I started off thinking to replicate the analysis exactly as it was done in the article in *De Tijd*. I was pleased to see an [accompanying article](http://www.maartenlambrechts.com/2016/10/03/how-i-built-a-scraper-to-measure-activity-of-mps.html) exists explaining the process of making the article. This way I learned the Flemish parliament has its [own API](http://ws.vlpar.be/e/opendata/api) in the spirit of open data. Unfortunately using this API turned out to be harder than expected. 

### Flemish parliament API is hard to use

Using the API was harder than expected because there's no obvious way to connect a member of parliament with their work. There are several ways to make the connection but none of them was satisfactory. This part of the post focuses on the inner workings of the API of the Flemish parliament. Feel free to skip this section if you're only interested in the results and not the process of making this post.

#### translate original UI scraping process to API

My first idea was to use the same process [as Maarten Lambrechts did](http://www.maartenlambrechts.com/2016/10/03/how-i-built-a-scraper-to-measure-activity-of-mps.html) but then using the API instead of [scraping](https://en.wikipedia.org/wiki/Web_scraping). This involves the following steps:

1. get all the MPs
2. get their details
3. extract questions and initiatives

The first 2 steps are easy (using the `/vv/op-datum` and `/vv/{persoonId}` endpoints). Unfortunately the MP details do not contain anything about questions and initiatives. Most of the fields aren't too interesting for analysis (honours, if MP is also senator, ...) or are interesting but difficult to parse (previous professions, other mandates, ...).

We get some attendance details but they're too general to do any useful analysis with:

```
"aanwezigheden-huidige-legislatuur": {
        "commissie-aanw": [],
        "plenaire-aanw": {
            "aanwezig": 1,
            "afwezig": 1
        }
    }
```

#### commissions as missing link

So sadly there's no straight link between the MP on the one hand and questions and initiatives on the other. However, I thought there might be a missing link between the two. The API provides all commission reports using `/comm/{commId}/verslagen`. Each report gives some info about questions and initiatives. Unfortunately there's no specific encoding of who asked the question or started the initiative as shown in the example of an initiative below:

```
"parl-initiatief": {
                "contacttype": [
                    {
                        "beschrijving": "Spreker",
                        "contact": []
                    },
                    {
                        "beschrijving": "Verslaggever",
                        "contact": []
                    }
                ],
                "titel": "Voorstel van resolutie van de heren Wouter Vanbesien en Björn Rzoska betreffende de organisatie van een hoorzitting met het middenveld over zijn visie op het beleid voor de komende zittingsperiode en over de gevraagde besparingsinspanningen.  Verslag namens de Commissie voor Algemeen Beleid, Financiën en Begroting uitgebracht door de heer Bart Tommelein"
        },
```


The MPs responsible for the initiative (Wouter Vanbesien and Björn Rzoska) are mentioned in the title field. This is not always the case so it's impossible to rely on this. In addition, to parse this info would take quite some natural language magic. And even if all this would go well the results would never be totally reliable.

Maarten Lambrechts in the end chose to just scrape the site of the parliament instead of using the API directly. However, I wanted to focus less on data collection and more on analysis in this article. Turning my mind to scraping would have meant doing a lot of (necessary) plumbing which I didn't feel like.

### What's an initiatives? What's a question?

Next to the data being hard to obtain through the API, there was a more fundamental reason I chose to move away from focusing on the questions and initiatives and focus on attendances instead. Another stumbling block turned out to be the concepts of questions, interpellations, interventions, themselves... Parliamentary work is [not straightforward](https://www.vlaamsparlement.be/over-het-vlaams-parlement/begrippenlijst). It feels like aggregating all this work into a single variable of "number of questions and initiatives" does not do justice to the subject.

So I was at a crossroads here: either I can embrace the full complexity of the topic or I can find another angle. If I go for the complexity route I have to take all the different kinds of parliamentary work (motions, oral questions, interpellations, resolutions, debates, written questions, ...) into account which would widely expand the scope of this article. Instead I chose to focus on simplicity and stick to attendances.

## Why focus on commissions?

The above explains why to focus on attendances. But why focus on the commission work and not the plenary? While researching this topic I felt the work done in commission doesn't get the attention it deserves. Most of the actual parliamentary work is not done in the plenary session (where all the members of parliament are present) but in the commissions beforehand. A commission is a group of members of parliaments specialized in a certain subject (finance, education, ...). So if you're not present in a commission, you're not there the moment the real work is done.

## Getting the data

To do a thorough analysis I needed these data sources:

- the commission meeting attendances
- a list of commission members

A third data source, [election results](https://vlaanderenkiest.be/verkiezingen2019/#/parlement/02000/verkozenen), was used to check if being a productive parliamentarian actually is rewarded by votes.

### commission meeting attendances

For the scope of this post data for only the last legislature (2014-2019) seemed more than enough.

To get the meeting notes we have to make calls to the API of the Flemish parliament:

- legislature (`/leg/alle`)
  - commission (`/comm/legislatuur`)
    - meeting notes (`/verg/volledig/zoek`)

The legislature call gives us the name of the last legislature, the commission call gives us the commission ids of the commission in that legislature and the last call gives us the meeting notes themselves.

From the meeting notes themselves I got more fields than needed since there's no almost no extra effort in doing so. The following fields were extracted:

- persons present
  - first name
  - last name
  - person id
  - party name
  - party id
  - gender
- meeting
  - meeting id
  - commission name
  - commission id
  - meeting start timestamp
  - meeting end timestamp
  - meeting description

The actual calls are divided in combinations of years and months (so January 2014 first, February 2014 second, and so on) in order not to overload the server. When trying to get all the meeting notes in a single call the server responds with a 500 HTTP status code meaning it could not handle the request.

What you end up with is a csv with each row being a unique combination of MP and meeting:

```
person_id | meeting_id
A         | X
B         | X
A         | Y
B         | Y
...
```

## Code

Code used in this post can be found on my [GitHub account](https://github.com/IsaacVerm/mps-activity).


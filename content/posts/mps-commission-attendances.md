---
title: 'MPs commission attendances'
date: 2019-10-26T14:14:40+02:00
draft: false
---

## Scope of this post

### Article which inspired this post

I was triggered to write this post because of an [interesting article](https://multimedia.tijd.be/vlaamsparlement/) in _De Tijd_ from some years ago which I recently stumbled upon again. [Maarten Lambrechts](<http://www.maartenlambrechts.com/](https://multimedia.tijd.be/vlaamsparlement/)>) investigated which members of the Flemish parliament are industrious and which are... less so.

### Focus on commission attendances

To make this more concrete he looked at the number of initiatives and questions by parliamentarians in the Flemish Parliament. Based on this he was able to divide politicians in several categories according to their productivity. However I chose to approach the question of politician productivity from another angle. Instead of focusing on questions and initiatives I chose to focus on the commission attendances by MPs. On the one hand I chose to focus on attendances since it's a very binary variable (you're either there or you're not), on the other hand I chose to limit myself to the commission meetings. A large chunk of parliamentary work is done in the commissions which mostly remains invisible to the public.

## Why focus on attendances?

Originally I started off thinking to replicate the analysis exactly as it was done in the article in _De Tijd_. I was pleased to see an [accompanying article](http://www.maartenlambrechts.com/2016/10/03/how-i-built-a-scraper-to-measure-activity-of-mps.html) exists explaining the process of making the article. This way I learned the Flemish parliament has its [own API](http://ws.vlpar.be/e/opendata/api) in the spirit of open data. Unfortunately using this API turned out to be harder than expected.

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
- election results

The third date source, [election results](https://vlaanderenkiest.be/verkiezingen2019/#/parlement/02000/verkozenen), was used to check if being a productive parliamentarian actually is rewarded by votes.

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

### list of commission members

Call is made to `/comm/{commId}` to get all the members in a commission. Acting members are not included.

### election results

In this case I didn't use the API but instead scraped the data like you would using a browser with `puppeteer`.

## Analysis

### Most productive members of parliament

First I have a look at what members of parliaments are the most productive by sorting them according to their presences. It goes beyond the raw number of presences in two ways:

- only obligatory presences are counted
- only members who completed the entire legislature (2014-2019) are retained

Because the official list of commission members is taken into account, only presences of official members of a commission are counted. Let's say politician A is a member of commission X but not Y. In case he goes to a meeting of commission Y this won't be counted as a presence.

Ended up with these 20 most productive members of parliament (with the party they belong to):

![top 20 most presences](/top-20-most-presences.png)

The most active members seem to be CD&V party members (5 CD&V MPs in the top 10 of most active parliamenterians). Nonetheless, the number 1 spot is taken by Bart Caron of Groen with about 600 presences blowing everyone else out of the water.

The same graph but for the least active members (who still were present in the entire legislature) paints a different picture:

![top 20 least presences](/top-20-least-presences.png)

Notice the spike for Hermes Sanctorum (striking name). He left Groen in 2016 over a [disagreement around animal rights](https://www.knack.be/nieuws/belgie/beste-hermes-sanctorum-mijn-sympathie-gaat-niet-uit-naar-je-ideologie-maar-naar-je-rechtlijnigheid/article-opinion-750903.html?cookie_check=1572513133). This messes somewhat with the sorting in descending order.

The MP with the least number of presences is Christian Van Eycken. Beginning of 2016 an [arrest warrant](https://en.wikipedia.org/wiki/Christian_Van_Eyken) was issued against him for murder. Because of procedural reasons he was released and somehow he kept attending commission meetings (I checked!). He was officialy found guilty in September 2019 which is after the end of the last legislature.

Based on the above there seems to be a difference in the number of presences between parties. Let's plot the average number of presences by party:

![average number of presences by party](/avg-presences-by-party.png)

UF is very low because there's only a single member, Christian Van Eycken, and his story has already been explained. What's more striking is the low number of presences by Vlaams Belang. On closer inspection it makes more sense. Turns out there's only a single Vlaams Belang commission member in our dataset ([Chris Janssens](https://www.vlaamsparlement.be/vlaamse-volksvertegenwoordigers/chris-janssens#parlementair)).

### Correlation productivity and electoral success

Looking at the data on presences is quite interesting in itself but does it mean anything? Is parliamentary work also rewarded by the voter? Election results are publicly available so we can correlate productivity and electoral success. Election results used are those of the 2019 election.

On a sidenote, only half of the members could be assigned a vote because:

- some members of parliament in legislature 2014-2019 didn't run for election in 2019
- names might be spelled differently in different datasets (making joins difficult)

The strategy used was to put names in lowercase. For example "De Meyer Jos" and "DE MEYER Jos" wouldn't match but de meyer jos would.

Let's start off with plotting the number of presences against the number of votes. In contrast to the productivity part above where we filtered to keep only those MPs which completed an entire legislature, in this electoral part we keep all MPs.

![presences vs votes](/presences-vs-votes.png)

It's hard to see any kind of relationship between presences in commissions and votes. This kind of means this analysis is rather without object. I look at the presences but apparently it has no impact at all on voter behavior.

Still, there are loose ends. For example, you can notice there are a few MPs getting votes without presences. How's that possible? I did some digging and there are different reasons for this:

- politicians like [Guy D'haeseleer](https://www.vlaamsparlement.be/vlaamse-volksvertegenwoordigers/guy-dhaeseleer#taken) only just joined the parliament.
- were minister in government ([Joke Schauvliege](https://www.vlaamsparlement.be/vlaamse-volksvertegenwoordigers/joke-schauvliege), after [resigning](https://www.nieuwsblad.be/cnt/dmf20190205_04154991)
- temporary replacements like [Jamila Hamddan Lachkar](https://nl.wikipedia.org/wiki/Jamila_Hamddan_Lachkar)

## Summary

Getting the data was way more difficult than expected (the Flemish parliament API could be made more user-friendly). Regarding the analysis itself, had a good look at which members of parliaments are the most and least productive. But in the end this matters little since voters don't seem to take this into account.

## Code

Code used in this post can be found on my [GitHub account](https://github.com/IsaacVerm/mps-activity).

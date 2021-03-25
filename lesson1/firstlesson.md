# Lesson 1: Wikipedia-based ontology induction

I want to construct a graph on Ancient Egyptian Culture, in particular, on Egyptian Deities and Mythology using Wikidata and DBpedia, and then formalize an ontology by remixing the semantic data of the two portals and (maybe) adding something else.

## What are the portals we are going to query?

### DBpedia

https://dbpedia.org/sparql

### Wikidata

https://query.wikidata.org/

Let's start from DBpedia, let's explore it using some general queries.

```sparql

ASK {dbr:Egypt ?something ?somethingelse} 
```



We use ASK instead of select if we want to check whether something exists or not, we don't care about what it is. It returns true so that means that dbr:Egypt exists.

```sparql

select ?objects {dbr:Egypt ?prop ?objects} 
```


Let's see every resource that is in the range of Egypt. There are a lot... let's try to count them


```sparql

select (count(?objects) as?tot) {dbr:Egypt ?prop ?objects} 

```

Ok they are a lot... but some of them are just label based. Let's focus on the uris.

```sparql

select distinct ?objects {dbr:Egypt ?prop ?objects .
?objects ?prop2 ?moreobjects } 

```

We do this because a non-uri will never take the role of the subject in a triple, so by doing this we are filtering them. Pay attention to the distinct!

```sparql

select (count(distinct ?objects)as ?tot) {dbr:Egypt ?prop ?objects .
?objects ?prop2 ?moreobjects } 

```

Ok they are still a lot. Let's try something different before looking directly at the Egypt page in dbpedia.

Let's see if there is something about culture in general!

```sparql

ask {dbr:Culture ?something ?somethingelse}

```

How would we check? Any Volunteer to tell me?

Let's now see what are the crossing between this

```sparql

select distinct ?egyptianculture { dbr:Egypt ?property ?egyptianculture .
dbr:Culture ?property ?egyptianculture }

```

Let's try to look at it from another point of view.

Let's try to get the concept of culture, concepts are generally expressed by the skos namespace

```sparql

select ?egyptianculture { ?egyptianculture ?prop dbr:Egypt ;
rdf:type skos:Concept }

```

No luck with this

```sparql

select distinct ?egyptianculture { dbr:Egypt ?something ?egyptianculture .
?egyptianculture rdf:type skos:Concept }

```

Or this. Let's try to look for a regex.

```sparql

select distinct ?egyptianculture { ?egyptianculture a skos:Concept .
filter(regex(?egyptianculture, "egypt", "i")) }

```

It takes a lot of time, too many results, let's try with a double regex.

```sparql

select distinct ?egyptianculture { ?egyptianculture a skos:Concept .
filter(regex(?egyptianculture, "egypt", "i"))
filter(regex(?egyptianculture, "culture", "i")) }

```

We finally got what we wanted. Here are some relevant results

* https://dbpedia.org/page/Category:Egyptian_culture
* https://dbpedia.org/page/Category:Egyptian_mythology_in_popular_culture
* https://dbpedia.org/page/Category:Ancient_Egyptian_culture
* https://dbpedia.org/page/Category:Ancient_Egypt_in_popular_culture



let's see all the properties and objects associates with these values

```sparql

select distinct ?egyptianculture ?prop ?obj { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture }
?egyptianculture ?prop ?obj .
?obj ?prop2 ?whatever}
GROUP BY ?egyptianculture

```

we can add these elements to the results:

* http://dbpedia.org/resource/Category:Egyptian_mythology
* http://dbpedia.org/resource/Category:Works_about_ancient_Egypt
* http://dbpedia.org/resource/Category:Ancient_Egypt

Let's take a look at the egyptian mythology page, maybe we can find other specific things

* dbc:Book_of_the_Dead
* dbc:Egyptian_deities
* dbc:Egyptian_legendary_creatures
* dbc:Locations_in_Egyptian_mythology

There is more we can do, for example we can see if these concepts are the object of another resource in dbpedia,
let's see, let's start with the original list and then we will add the other things later

``` sparql

select distinct ?sub ?prop ?egyptianculture   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture }
?sub ?prop ?egyptianculture .}
GROUP BY ?egyptianculture

```

There is way more now, but before getting them all we want to exclude the ones that use Skos:broader as we already have them

```sparql

select distinct ?sub ?prop ?egyptianculture   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture } }
GROUP BY ?egyptianculture

```

Now we also add the last 7 categories discovered after

``` sparql

select distinct ?sub ?prop ?egyptianculture   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Egyptian_mythology dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_deities dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture } }
GROUP BY ?egyptianculture

```

Let's count them

```sparql

select (count(distinct ?sub) as?tot)   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Egyptian_mythology dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_deities dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture } }

```

Now let's divide the count by the category

```sparql

select ?egyptianculture (count(distinct ?sub) as?tot)   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Egyptian_mythology dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_deities dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture } } group by ?egyptianculture

```

I'm still not convinced by the number of resources we have, let's say we want to get more elements by expanding what we we know of Egyptian Deities and Egyptian Mythology.

Let's look at egyptian deities first. We can see that it is related to 
http://dbpedia.org/resource/List_of_Egyptian_deities and we can see that here there are all the egyptian deities, fundamental for the graph we want to create. But are we sure that all these elements are Egyptian Deities? It seems that there is some noise here, we want to filter that noise out so let's look at one element that we know it's a deity. for example Seth. We can see that Seth has as subject dbc:Egyptian_gods that is a narrower concept for egyptian_deities

Ok, let's try to do this query then

```sparql

select distinct ?god where  {  dbr:List_of_Egyptian_deities dbo:wikiPageWikiLink  ?god .
?god dct:subject ?deity .
{?deity skos:broader dbc:Egyptian_deities} UNION {?deity skos:broader dbc:Egyptian_gods}
}

```

So now we have our list of deities (let's keep in mind that we can extract so much more from deities in the dbpedia page) but we will get to that later

Now let's focus on the Egyptian Mythology part. We can see that there is an egyptian mythology resource linked to the egyptian mythology concept. I wonder what gods are included here, let's check with a query.

```sparql

select ?commonentity where  { dbr:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity .
?commonentity dct:subject ?deity .
?deity skos:broader dbc:Egyptian_deities
 }

```

Ok there are some but we don't want to add more deities so let's filter them out

```sparql
select distinct ?commonentity where  { dbr:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity .
MINUS {?commonentity dct:subject ?deity .
{?deity skos:broader dbc:Egyptian_deities} UNION {?deity skos:broader dbc:Egyptian_gods} }
 }
```

This is a list of uris linked to the resource Egyptian_mythology, but what if we wanted to combine them with the uris linked to the concept Egyptian_mythology?

Let's try with a query.

```sparql
select distinct ?commonentity where  { {dbr:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity . } UNION {dbc:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity . }
MINUS {?commonentity dct:subject ?deity .
{?deity skos:broader dbc:Egyptian_deities} UNION {?deity skos:broader dbc:Egyptian_gods} }
 }

```

So now we have a list of elements that are somehow associated with Egyptian Mythology but are not deities.

We want to get a list of the elements associated to other categories that we have seen before, without actually going too much into details with them, let's do it with this query

``` sparql
select ?egyptianculture (group_concat(?sub;SEPARATOR=" ") as ?subs)   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub skos:broader ?egyptianculture } } group by ?egyptianculture
```

We want to exclude those concepts that we already have in the previous 2 lists so let's try to write this query:

```sparql

select ?egyptianculture (group_concat(?sub;SEPARATOR=" ") as ?subs)   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub skos:broader ?egyptianculture }
MINUS { {dbr:Egyptian_mythology dbo:wikiPageWikiLink ?sub . } UNION {dbc:Egyptian_mythology dbo:wikiPageWikiLink ?sub . } }
MINUS {?commonentity dct:subject ?sub .
{?sub skos:broader dbc:Egyptian_deities} UNION {?sub skos:broader dbc:Egyptian_gods} }
} group by ?egyptianculture

```



So now we have:

* a list of Egyptian Deities (with more details about them extractable from dbpedia)
* a list of general egyptian mythological related elements (with more details about them extractable from wikidata)
* a list of other more specific mythological elements (with more details about them extractable from wikidata)

Let's look at one page of a God to see what more details could be extracted.
Let's look at Anubis (https://dbpedia.org/page/Anubis)

we could just write

```sparql
describe dbr:Anubis
```

But as you can see it gets quite messy

let's just look at the properties that we have for Anubis


```sparql
select distinct ?prop where { {dbr:Anubis ?prop ?something} UNION {?something2 ?prop dbr:Anubis} } 
```

Let's select a few that we will use in our graph

* http://xmlns.com/foaf/0.1/depiction -> foaf:depiction
* http://dbpedia.org/property/parents -> dbp:parents
* http://dbpedia.org/property/symbol  -> dbp:symbol
* http://dbpedia.org/property/consort -> dbp:consort
* http://dbpedia.org/property/cultCenter -> dbp:cultCenter
* http://dbpedia.org/property/godOf -> dbp:godOf
* http://dbpedia.org/property/offspring -> dbp:offspring
* http://dbpedia.org/property/sibilings -> dbp:sibilings

Let's try to construct a graph on the deities using these information
```sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX myont: <http://hello.it/>
CONSTRUCT { 
?god foaf:depiction ?depiction ;
myont:belongsToMythology dbc:Egyptian_mythology ;
dbp:parents ?parent ;
dbp:symbol ?symbol ;
dbp:consort ?consort ;
dbp:cultCenter ?cultcenter ;
dbp:godOf ?godof ;
dbp:offspring ?offspring ;
dbp:sibilings ?sibilings}
WHERE {
dbr:List_of_Egyptian_deities dbo:wikiPageWikiLink  ?god .
?god dct:subject ?deity .
?deity skos:broader dbc:Egyptian_deities
OPTIONAL { ?god foaf:depiction ?depiction  }
OPTIONAL { ?god dbp:parents ?parent }
OPTIONAL {?god dbp:symbol ?symbol }
OPTIONAL {?god dbp:consort ?consort }
OPTIONAL {?god dbp:cultCenter ?cultcenter }
OPTIONAL {?god dbp:godOf ?godof }
OPTIONAL {?god dbp:offspring ?offspring }
OPTIONAL {?god dbp:sibilings ?sibilings}
 } 
```

```sparql
PREFIX myont: <helloworld.it/#> 
CONSTRUCT { ?god foaf:depiction ?depiction ;
myont:belongsToMythology dbc:Egyptian_mythology ;
dbp:parents ?parent ;
dbp:symbol ?symbol ;
dbp:consort ?consort ;
dbp:cultCenter ?cultcenter ;
dbp:godOf ?godof ;
dbp:offspring ?offspring ;
dbp:sibilings ?sibilings }
WHERE {  dbr:List_of_Egyptian_deities dbo:wikiPageWikiLink  ?god .
?god dct:subject ?deity .
{?deity skos:broader dbc:Egyptian_deities} UNION {?deity skos:broader dbc:Egyptian_gods}
OPTIONAL { ?god foaf:depiction ?depiction  }
OPTIONAL { ?god dbp:parents ?parent }
OPTIONAL {?god dbp:symbol ?symbol }
OPTIONAL {?god dbp:consort ?consort }
OPTIONAL {?god dbp:cultCenter ?cultcenter }
OPTIONAL {?god dbp:godOf ?godof }
OPTIONAL {?god dbp:offspring ?offspring }
OPTIONAL {?god dbp:sibilings ?sibilings} }
 
```




From all the other entities that we have, we just want to check their "type", as to say, what they are an instance of in wikidata. Because wikidata has more stable types for resources compared to dbpedia (for some reasons some gods are considered as type "psychological entities" in dbpedia). 
Let's try to see if we can get a Wikidata URI from these entities. Let's see the query to extract the wikidata uris.

```sparql

select distinct ?egyptianculture ?sub ?wikidata  { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub skos:broader ?egyptianculture }
MINUS { {dbr:Egyptian_mythology dbo:wikiPageWikiLink ?sub . } UNION {dbc:Egyptian_mythology dbo:wikiPageWikiLink ?sub . } }
MINUS {?commonentity dct:subject ?sub .
{?sub skos:broader dbc:Egyptian_deities} UNION {?sub skos:broader dbc:Egyptian_gods} }
OPTIONAL {?sub owl:sameAs ?wikidata .
FILTER (regex(?wikidata, "wikidata", "i"))} }
ORDER BY ?egyptianculture

```

We download the results of this query as a csv. We start to filter for each category. We take first ancient egypt, we copy those code and then go to http://removelinebreaks.net . We copy this list, go to a notepad and press ctrl + h (or go to edit, replace ) we replace "http://www.wikidata.org/entity/" with "wd:" . Then we can go on http://query.wikidata.org and do this query:

```sparql

select  ?entity ?entityLabel ?type ?typeLabel  WHERE { 
  VALUES ?entity {  wd:Q496002
  wd:Q253155
  wd:Q16887603  wd:Q1536350  wd:Q60744726  wd:Q10521987  wd:Q5445737  wd:Q3488951  wd:Q30715241  wd:Q7180697  wd:Q719699
  wd:Q4752832  wd:Q4752833  wd:Q22936469  wd:Q180927
  wd:Q4904119  wd:Q1975262  wd:Q1471686  wd:Q19903268  wd:Q193850
  wd:Q16843683  wd:Q1894673  wd:Q7368288  wd:Q435967
  wd:Q7738072  wd:Q17051361  wd:Q4118674  wd:Q4481150  wd:Q7852161  wd:Q7114179  wd:Q3518019  wd:Q7112549  wd:Q24027445  wd:Q16303552  wd:Q16839161  wd:Q1590754  wd:Q11041351  wd:Q10914393  wd:Q3375464  wd:Q56277528  wd:Q253181
  wd:Q4767870  wd:Q907879
  wd:Q7067835  wd:Q4920720  wd:Q7784429  wd:Q22936426  wd:Q3552228 }
 ?entity wdt:P31 ?type .
SERVICE wikibase:label { bd:serviceParam wikibase:language "en,[AUTO_LANGUAGE]". }}
group by  ?type ?typeLabel ?entityLabel

```
We download the results in csv. We analyze them and decide which ones can be generalized and which ones are not interesting at all.

The results are these 6 properties:

* isHistoricallyRelatedTo for entities of type: wd:Q625298 wd:Q3024240 wd:Q4204501 wd:Q11514315 wd:Q17524420 wd:Q11146759 which are  wd:Q4920720 wd:Q10914393 wd:Q3552228 wd:Q16843683 wd:Q10914393 wd:Q193850 wd:Q1894673 wd:Q253155
* isCulturalSiteRelatedTo for entities of type: wd:Q839954 wd:Q39614 which are  wd:Q1975262 wd:Q16303552 wd:Q16839161 wd:Q496002 wd:Q1536350 wd:Q7180697 wd:Q24027445
* isArtisticMovementOf for entities of type: wd:Q968159 which are wd:Q56277528
* isStylisticallyRelatedTo for entities of type: wd:Q96338860 which are wd:Q1471686 
* isControversyOf for entities of type: wd:Q1255828 which are wd:Q4752832
* isCulturalMovieAbout for entities of: wd:Q11424 which are wd:Q7738072



Let's modify the construct query by including both the values of Ancient Egypt and the new properties.

``` sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX myont: <http://hello.it/>
CONSTRUCT {
dbc:Egyptian_mythology myont:about dbc:Ancient_Egypt .
?history myont:isHistoricallyRelatedTo dbc:Ancient_Egypt .
?site myont:isCulturalSiteRelatedTo dbc:Ancient_Egypt .
?am myont:isArtisticMovementOf dbc:Ancient_Egypt .
?style myont:isStylisticallyRelatedTo dbc:Ancient_Egypt .
?contr myont:isControversyOf dbc:Ancient_Egypt .
?movie myont:isCulturalMovieAbout dbc:Ancient_Egypt .
?god foaf:depiction ?depiction ;
myont:belongsToMythology dbc:Egyptian_mythology ;
dbp:parents ?parent ;
dbp:symbol ?symbol ;
dbp:consort ?consort ;
dbp:cultCenter ?cultcenter ;
dbp:godOf ?godof ;
dbp:offspring ?offspring ;
dbp:sibilings ?sibilings}
WHERE {
VALUES ?historyW {wd:Q4920720 wd:Q10914393 wd:Q3552228 wd:Q16843683 wd:Q10914393 wd:Q193850 wd:Q1894673 wd:Q253155 }
VALUES ?siteW { wd:Q1975262 wd:Q16303552 wd:Q16839161 wd:Q496002 wd:Q1536350 wd:Q7180697 wd:Q24027445}
VALUES ?amW { wd:Q56277528}
VALUES ?styleW {wd:Q1471686 }
VALUES ?contrW { wd:Q4752832}
VALUES ?movieW {wd:Q7738072 }
?movie owl:sameAs ?movieW .
?contr owl:sameAs ?contrW .
?style owl:sameAs ?styleW .
?am owl:sameAs ?amW .
?site owl:sameAs ?siteW .
?history owl:sameAs ?historyW .
dbr:List_of_Egyptian_deities dbo:wikiPageWikiLink  ?god .
?god dct:subject ?deity .
?deity skos:broader dbc:Egyptian_deities
OPTIONAL { ?god foaf:depiction ?depiction  }
OPTIONAL { ?god dbp:parents ?parent }
OPTIONAL {?god dbp:symbol ?symbol }
OPTIONAL {?god dbp:consort ?consort }
OPTIONAL {?god dbp:cultCenter ?cultcenter }
OPTIONAL {?god dbp:godOf ?godof }
OPTIONAL {?god dbp:offspring ?offspring }
OPTIONAL {?god dbp:sibilings ?sibilings}
 } 
```

ACTIVITY


Group 1 Needs to do the same for category: **Ancient_Egyptian_culture**
Group 2 Needs to do the same for categories: **Book_of_the_Dead and Egyptian_legendary_creatures**
Group 3 Needs to do the same for category: **Egyptian_culture**
Group 4 Needs to do the same for categories: **Egyptian_mythology_in_popular_culture, Locations_in_Egyptian_mythology, Works_about_ancient_Egypt, Ancient_egypt_in_popular_culture**


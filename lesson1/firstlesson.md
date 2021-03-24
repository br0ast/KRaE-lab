# Lesson 1: Wikipedia-based ontology induction

I want to construct a graph on Ancient Egyptian Culture, in particular, on Egyptian Deities and Mythology using Wikidata and DBpedia, and then formalize an ontology by remixing the semantic data of the two portals and (maybe) adding something else.

## What are the portals we are going to query?

### DBpedia

https://dbpedia.org/sparql

### Wikidata

https://query.wikidata.org/

Let's start from DBpedia, let's explore it using some general queries.

@@@sparql

ASK {dbr:Egypt ?something ?somethingelse} 

@@@sparql

We use ASK instead of select if we want to check whether something exists or not, we don't care about what it is. It returns true so that means that dbr:Egypt exists.

@@@sparql

select ?objects {dbr:Egypt ?prop ?objects} 

@@@sparql

Let's see every resource that is in the range of Egypt. There are a lot... let's try to count them


@@@sparql

select (count(?objects) as?tot) {dbr:Egypt ?prop ?objects} 

@@@sparql

Ok they are a lot... but some of them are just label based. Let's focus on the uris.

@@@sparql

select distinct ?objects {dbr:Egypt ?prop ?objects .
?objects ?prop2 ?moreobjects } 

@@@

We do this because a non-uri will never take the role of the subject in a triple, so by doing this we are filtering them. Pay attention to the distinct!

@@@sparql

select (count(distinct ?objects)as ?tot) {dbr:Egypt ?prop ?objects .
?objects ?prop2 ?moreobjects } 

@@@

Ok they are still a lot. Let's try something different before looking directly at the Egypt page in dbpedia.

Let's see if there is something about culture in general!

@@@sparql

ask {dbr:Culture ?something ?somethingelse}

@@@

How would we check? Any Volunteer to tell me?

Let's now see what are the crossing between this

@@@sparql

select distinct ?egyptianculture { dbr:Egypt ?property ?egyptianculture .
dbr:Culture ?property ?egyptianculture }

@@@

MMmh there does not seem to be anything useful here, let's try to do it backwards.

@@@sparql

select distinct ?egyptianculture { ?egyptianculture ?property dbr:Egypt .
?egyptianculture ?property2 dbr:Culture }

@@@

or something like this

@@@sparql
select distinct ?egyptianculture { dbr:Egypt ?property ?egyptianculture .
?egyptianculture ?property2 dbr:Culture }
@@@

Let's try to look at it from another point of view.

Let's try to get the concept of culture, concepts are generally expressed by the skos namespace

@@@sparql

select ?egyptianculture { ?egyptianculture ?prop dbr:Egypt ;
rdf:type skos:Concept }

@@@

No luck with this

@@@ sparql

select distinct ?egyptianculture { dbr:Egypt ?something ?egyptianculture .
?egyptianculture rdf:type skos:Concept }

@@@

Or this. Let's try to look for a regex.

@@@sparql

select distinct ?egyptianculture { ?egyptianculture a skos:Concept .
filter(regex(?egyptianculture, "egypt", "i")) }

@@@

It takes a lot of time, too many results, let's try with a double regex.

@@@sparql

select distinct ?egyptianculture { ?egyptianculture a skos:Concept .
filter(regex(?egyptianculture, "egypt", "i"))
filter(regex(?egyptianculture, "culture", "i")) }

@@@

We finally got what we wanted. Let's explore some of these relevant results

* https://dbpedia.org/page/Category:Egyptian_culture
* https://dbpedia.org/page/Category:Egyptian_mythology_in_popular_culture
* https://dbpedia.org/page/Category:Ancient_Egyptian_culture
* https://dbpedia.org/page/Category:Ancient_Egypt_in_popular_culture



let's see all the properties and objects associates with these values

@@@sparql

select distinct ?egyptianculture ?prop ?obj { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture }
?egyptianculture ?prop ?obj .
?obj ?prop2 ?whatever}
GROUP BY ?egyptianculture

@@@

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

@@@ sparql

select distinct ?sub ?prop ?egyptianculture   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture }
?sub ?prop ?egyptianculture .}
GROUP BY ?egyptianculture

@@@

There is way more now, but before getting them all we want to exclude the ones that use Skos:broader as we already have them

@@@sparql

select distinct ?sub ?prop ?egyptianculture   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture } }
GROUP BY ?egyptianculture

@@@

Now we also add the last 7 categories discovered after

@@@ sparql

select distinct ?sub ?prop ?egyptianculture   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Egyptian_mythology dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_deities dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture } }
GROUP BY ?egyptianculture

@@@

Let's count them

@@@sparql

select (count(distinct ?sub) as?tot)   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Egyptian_mythology dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_deities dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture } }

@@@

Now let's divide the count by the category

@@@sparql

select ?egyptianculture (count(distinct ?sub) as?tot)   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Egyptian_mythology dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_deities dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture } } group by ?egyptianculture

@@@

I'm still not convinced by the number of resources we have, let's say we want to get more elements by expanding what we we know of Egyptian Deities and Egyptian Mythology.

Let's look at egyptian deities first. We can see that it is related to 
http://dbpedia.org/resource/List_of_Egyptian_deities and we can see that here there are all the egyptian deities, fundamental for the graph we want to create. But are we sure that all these elements are Egyptian Deities? It seems that there is some noise here, we want to filter that noise out so let's look at one element that we know it's a deity. for example Seth. We can see that Seth has as subject dbc:Egyptian_gods so let's filter our elements by saying that they must have this subject.

So now we have a list of gods, but... Can you notice something here?

we have no goddesses, might that be because there is another property for a goddess?
Let's look at a goddess page in dbpedia

Ok, let's try to do this query then

@@@sparql

select distinct ?god where  {  dbr:List_of_Egyptian_deities dbo:wikiPageWikiLink  ?god .
?god dct:subject ?deity .
{?deity skos:broader dbc:Egyptian_deities} UNION {?deity skos:broader dbc:Egyptian_gods}
}

@@@

So now we have our list of deities (let's keep in mind that we can extract so much more from deities in the dbpedia page) but we will get to that later

Now let's focus on the Egyptian Mythology part. We can see that there is an egyptian mythology resource linked to the egyptian mythology concept. I wonder what gods are included here, let's check with a query.

@@@sparql

select ?commonentity where  { dbr:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity .
?commonentity dct:subject ?deity .
?deity skos:broader dbc:Egyptian_deities
 }

@@@

Ok there are some but we don't want to add more deities so let's filter them out

@@@sparql
select distinct ?commonentity where  { dbr:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity .
MINUS {?commonentity dct:subject ?deity .
{?deity skos:broader dbc:Egyptian_deities} UNION {?deity skos:broader dbc:Egyptian_gods} }
 }
@@@

This is a list of uris linked to the resource Egyptian_mythology, but what if we wanted to combine them with the uris linked to the concept Egyptian_mythology?

Let's try with a query.

@@@sparql
select distinct ?commonentity where  { {dbr:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity . } UNION {dbc:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity . }
MINUS {?commonentity dct:subject ?deity .
{?deity skos:broader dbc:Egyptian_deities} UNION {?deity skos:broader dbc:Egyptian_gods} }
 }

@@@

So now we have a list of elements that are somehow associated with Egyptian Mythology but are not deities.

We want to get a list of the elements associated to other categories that we have seen before, without actually going too much into details with them, let's do it with this query

@@@ sparql
select ?egyptianculture (group_concat(?sub;SEPARATOR=" ") as ?subs)   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub skos:broader ?egyptianculture } } group by ?egyptianculture
@@@

We want to exclude those concepts that we already have in the previous 2 lists so let's try to write this query:

@@@sparql

select ?egyptianculture (group_concat(?sub;SEPARATOR=" ") as ?subs)   { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt dbc:Book_of_the_Dead dbc:Egyptian_legendary_creatures dbc:Locations_in_Egyptian_mythology }
?sub ?prop ?egyptianculture .
MINUS {?sub skos:broader ?egyptianculture }
MINUS { {dbr:Egyptian_mythology dbo:wikiPageWikiLink ?sub . } UNION {dbc:Egyptian_mythology dbo:wikiPageWikiLink ?sub . } }
MINUS {?commonentity dct:subject ?sub .
{?sub skos:broader dbc:Egyptian_deities} UNION {?sub skos:broader dbc:Egyptian_gods} }
} group by ?egyptianculture

@@@

Let's try to see if we can get a Wikidata URI from these entities

@@@sparql

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

@@@

Let's also get the wikidata uri of our general mythology entities

@@@sparql
select distinct ?commonentity ?wikidata where  { {dbr:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity . } UNION {dbc:Egyptian_mythology dbo:wikiPageWikiLink ?commonentity . }
MINUS {?commonentity dct:subject ?deity .
{?deity skos:broader dbc:Egyptian_deities} UNION {?deity skos:broader dbc:Egyptian_gods} }
OPTIONAL {?commonentity owl:sameAs ?wikidata .
FILTER (regex(?wikidata, "wikidata", "i"))}
 }

@@@

So now we have:

* a list of Egyptian Deities (with more details about them extractable from dbpedia)
* a list of general egyptian mythological related elements (with more details about them extractable from dbpedia)
* a list of other more specific mythological elements (with more details about them extractable from dbpedia)

Let's look at one page of a God to see what more details could be extracted.
Let's look at Set (https://dbpedia.org/page/Anubis)

we could just write

@@@sparql
describe dbr:Anubis
@@@

But as you can see it gets quite messy

let's just look at the properties that we have for Anubis


@@@sparql
select distinct ?prop where { {dbr:Anubis ?prop ?something} UNION {?something2 ?prop dbr:Anubis} } 
@@@

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

@@@sparql
PREFIX hello: < helloworld.it/# > 
CONSTRUCT { ?god foaf:depiction ?depiction ;
hello:belongsToMythology dbc:Egyptian_mythology ;
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
 
@@@



Let's take only the wikidata elements and query wikidata to see what kind of instances have those elements
we go here: https://removelinebreaks.net/ because we need the spaces removed to include them in the VALUES of wikidata.


Federated construct!!!!
@@@sparql
PREFIX wd: <http://www.wikidata.org/entity/>
PREFIX wdt: <http://www.wikidata.org/prop/direct/>
PREFIX wikibase: <http://wikiba.se/ontology#>
PREFIX p: <http://www.wikidata.org/prop/>
PREFIX ps: <http://www.wikidata.org/prop/statement/>
PREFIX pq: <http://www.wikidata.org/prop/qualifier/>
PREFIX bd: <http://www.bigdata.com/rdf#>
PREFIX owl: <http://www.w3.org/2002/07/owl#>
PREFIX rdfs: <http://www.w3.org/2000/01/rdf-schema#>
PREFIX foaf: <http://xmlns.com/foaf/0.1/>
PREFIX dct: <http://purl.org/dc/terms/>
PREFIX dbc: <http://dbpedia.org/resource/Category:>
PREFIX dbp: <http://dbpedia.org/property/>
PREFIX dbr: <http://dbpedia.org/resource/>
PREFIX dbo: <http://dbpedia.org/ontology/>
PREFIX hello: <http://hello.it/>
CONSTRUCT { ?god hello:xd hello:lul .
?god owl:sameAs ?parent .
?god foaf:depiction ?depiction ;
hello:belongsToMythology dbc:Egyptian_mythology ;
dbp:parents ?parent ;
dbp:symbol ?symbol ;
dbp:consort ?consort ;
dbp:cultCenter ?cultcenter ;
dbp:godOf ?godof ;
dbp:offspring ?offspring ;
dbp:sibilings ?sibilings}
WHERE { SERVICE <https://query.wikidata.org/sparql>
          { ?parent wdt:P361 wd:Q205740 }
SERVICE <https://dbpedia.org/sparql> {
?god owl:sameAs ?parent .
dbr:List_of_Egyptian_deities dbo:wikiPageWikiLink  ?god .
?god dct:subject ?deity .
OPTIONAL { ?god foaf:depiction ?depiction  }
OPTIONAL { ?god dbp:parents ?parent }
OPTIONAL {?god dbp:symbol ?symbol }
OPTIONAL {?god dbp:consort ?consort }
OPTIONAL {?god dbp:cultCenter ?cultcenter }
OPTIONAL {?god dbp:godOf ?godof }
OPTIONAL {?god dbp:offspring ?offspring }
OPTIONAL {?god dbp:sibilings ?sibilings}
 } 
              }@@@


@@@ sparql

select distinct ?wikidata  { 
VALUES ?egyptianculture { dbc:Egyptian_culture dbc:Egyptian_mythology_in_popular_culture dbc:Ancient_Egyptian_culture dbc:Ancient_Egypt_in_popular_culture dbc:Egyptian_mythology dbc:Works_about_ancient_Egypt dbc:Ancient_Egypt }
?sub ?prop ?egyptianculture .
MINUS {?sub <http://www.w3.org/2004/02/skos/core#broader> ?egyptianculture }
?sub owl:sameAs ?wikidata .
FILTER (regex(?wikidata, "wikidata", "i"))} 

@@@

Let's go into a notepad and remove "http://www.wikidata.org/resource/" with "wd:"
and do this query on https://query.wikidata.org/ 

@@@sparql

select ?type ?typeLabel ?resource  ?resourceLabel WHERE { VALUES ?type {  wd:Q16887603  wd:Q10521987  wd:Q4516292  wd:Q337344  wd:Q787057  wd:Q28246
  wd:Q6421966  wd:Q3824923  wd:Q1462523  wd:Q7531335  wd:Q3518019  wd:Q67889726  wd:Q7056762  wd:Q1153959  wd:Q1122064  wd:Q253155  wd:Q40535
  wd:Q1536350  wd:Q1684335  wd:Q3700645  wd:Q7180697  wd:Q432535  wd:Q4879280  wd:Q5692481  wd:Q47542643  wd:Q684109  wd:Q22936344  wd:Q9031060  wd:Q1099570  wd:Q4481150  wd:Q1648270  wd:Q2383661  wd:Q22907154  wd:Q2884882  wd:Q8064284  wd:Q6868659  wd:Q18122904  wd:Q15523428  wd:Q749371  wd:Q1303227  wd:Q7067851  wd:Q6754289  wd:Q766876  wd:Q1894673  wd:Q174361  wd:Q320461  wd:Q1784774  wd:Q4118876  wd:Q2332419  wd:Q253181  wd:Q5715
  wd:Q5348497  wd:Q211286  wd:Q2278032  wd:Q30715241  wd:Q1583849  wd:Q11768
  wd:Q552228  wd:Q2298292  wd:Q2655498  wd:Q2839173  wd:Q4959540  wd:Q816712  wd:Q16303552  wd:Q17011845  wd:Q187462  wd:Q1346230  wd:Q2242136  wd:Q16889471  wd:Q3488951  wd:Q6564382  wd:Q2600111  wd:Q4752834  wd:Q3064288  wd:Q2893435  wd:Q12811105  wd:Q193850  wd:Q18639717  wd:Q22935804  wd:Q32137
  wd:Q4413382  wd:Q4072072  wd:Q4118674  wd:Q7852161  wd:Q182863  wd:Q11041351  wd:Q254093  wd:Q2642660  wd:Q153263  wd:Q2724213  wd:Q4752830  wd:Q3332058  wd:Q5074373  wd:Q1471686  wd:Q34760668  wd:Q19903268  wd:Q582169  wd:Q853296  wd:Q17051361  wd:Q42235
  wd:Q1519552  wd:Q3104807  wd:Q16839161  wd:Q1590754  wd:Q7883361  wd:Q10914393  wd:Q3375464  wd:Q205740  wd:Q4767870  wd:Q907879  wd:Q189574  wd:Q17004916  wd:Q465768  wd:Q4752833  wd:Q22936469  wd:Q3312662  wd:Q1528093  wd:Q4904119  wd:Q129076  wd:Q435967  wd:Q180439  wd:Q6401088  wd:Q3814507  wd:Q7447170  wd:Q56277528  wd:Q19056138  wd:Q263516  wd:Q496002  wd:Q916292  wd:Q60744726  wd:Q2884880  wd:Q7752629  wd:Q5215223  wd:Q16843683  wd:Q997840  wd:Q16209288  wd:Q20649610  wd:Q7112549  wd:Q1430627  wd:Q1637382  wd:Q7735537  wd:Q233458  wd:Q7067835  wd:Q5348471  wd:Q618140  wd:Q871389  wd:Q5445737  wd:Q719699  wd:Q2172038  wd:Q4752832  wd:Q7368288  wd:Q2569896  wd:Q1131955  wd:Q1214882  wd:Q739278  wd:Q47542610  wd:Q24027445  wd:Q1520921  wd:Q17512042  wd:Q29919
  wd:Q22930228  wd:Q5178560  wd:Q2003175  wd:Q22935718  wd:Q853611  wd:Q2210354  wd:Q7575841  wd:Q180927  wd:Q1975262  wd:Q719063  wd:Q430514  wd:Q7738072  wd:Q16890528  wd:Q7114179  wd:Q3783773  wd:Q3576428  wd:Q4920720  wd:Q22909812  wd:Q4165845  wd:Q358209  wd:Q804596  wd:Q1117866  wd:Q772923  wd:Q18129813  wd:Q7784429  wd:Q22936426  wd:Q1413565  wd:Q3552228 }
  ?resource wdt:P31 ?type .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?type ?typeLabel ?resource  ?resourceLabel

  @@@

  As we can see there aren't many results, let's try to change the query a bit, let's say we want to see all the relationship between Paintings and the resources above, we also want a specific label for the relationship

@@@ sparql

select ?type ?typeLabel ?resource  ?resourceLabel ?prop ?propLabel WHERE { VALUES ?type {  wd:Q16887603  wd:Q10521987  wd:Q4516292  wd:Q337344  wd:Q787057  wd:Q28246
  wd:Q6421966  wd:Q3824923  wd:Q1462523  wd:Q7531335  wd:Q3518019  wd:Q67889726  wd:Q7056762  wd:Q1153959  wd:Q1122064  wd:Q253155  wd:Q40535
  wd:Q1536350  wd:Q1684335  wd:Q3700645  wd:Q7180697  wd:Q432535  wd:Q4879280  wd:Q5692481  wd:Q47542643  wd:Q684109  wd:Q22936344  wd:Q9031060  wd:Q1099570  wd:Q4481150  wd:Q1648270  wd:Q2383661  wd:Q22907154  wd:Q2884882  wd:Q8064284  wd:Q6868659  wd:Q18122904  wd:Q15523428  wd:Q749371  wd:Q1303227  wd:Q7067851  wd:Q6754289  wd:Q766876  wd:Q1894673  wd:Q174361  wd:Q320461  wd:Q1784774  wd:Q4118876  wd:Q2332419  wd:Q253181  wd:Q5715
  wd:Q5348497  wd:Q211286  wd:Q2278032  wd:Q30715241  wd:Q1583849  wd:Q11768
  wd:Q552228  wd:Q2298292  wd:Q2655498  wd:Q2839173  wd:Q4959540  wd:Q816712  wd:Q16303552  wd:Q17011845  wd:Q187462  wd:Q1346230  wd:Q2242136  wd:Q16889471  wd:Q3488951  wd:Q6564382  wd:Q2600111  wd:Q4752834  wd:Q3064288  wd:Q2893435  wd:Q12811105  wd:Q193850  wd:Q18639717  wd:Q22935804  wd:Q32137
  wd:Q4413382  wd:Q4072072  wd:Q4118674  wd:Q7852161  wd:Q182863  wd:Q11041351  wd:Q254093  wd:Q2642660  wd:Q153263  wd:Q2724213  wd:Q4752830  wd:Q3332058  wd:Q5074373  wd:Q1471686  wd:Q34760668  wd:Q19903268  wd:Q582169  wd:Q853296  wd:Q17051361  wd:Q42235
  wd:Q1519552  wd:Q3104807  wd:Q16839161  wd:Q1590754  wd:Q7883361  wd:Q10914393  wd:Q3375464  wd:Q205740  wd:Q4767870  wd:Q907879  wd:Q189574  wd:Q17004916  wd:Q465768  wd:Q4752833  wd:Q22936469  wd:Q3312662  wd:Q1528093  wd:Q4904119  wd:Q129076  wd:Q435967  wd:Q180439  wd:Q6401088  wd:Q3814507  wd:Q7447170  wd:Q56277528  wd:Q19056138  wd:Q263516  wd:Q496002  wd:Q916292  wd:Q60744726  wd:Q2884880  wd:Q7752629  wd:Q5215223  wd:Q16843683  wd:Q997840  wd:Q16209288  wd:Q20649610  wd:Q7112549  wd:Q1430627  wd:Q1637382  wd:Q7735537  wd:Q233458  wd:Q7067835  wd:Q5348471  wd:Q618140  wd:Q871389  wd:Q5445737  wd:Q719699  wd:Q2172038  wd:Q4752832  wd:Q7368288  wd:Q2569896  wd:Q1131955  wd:Q1214882  wd:Q739278  wd:Q47542610  wd:Q24027445  wd:Q1520921  wd:Q17512042  wd:Q29919
  wd:Q22930228  wd:Q5178560  wd:Q2003175  wd:Q22935718  wd:Q853611  wd:Q2210354  wd:Q7575841  wd:Q180927  wd:Q1975262  wd:Q719063  wd:Q430514  wd:Q7738072  wd:Q16890528  wd:Q7114179  wd:Q3783773  wd:Q3576428  wd:Q4920720  wd:Q22909812  wd:Q4165845  wd:Q358209  wd:Q804596  wd:Q1117866  wd:Q772923  wd:Q18129813  wd:Q7784429  wd:Q22936426  wd:Q1413565  wd:Q3552228 }
  ?resource wdt:P31 wd:Q3305213 ;
          ?p ?type .
  ?prop wikibase:directClaim ?p .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?type ?typeLabel ?resource  ?resourceLabel ?prop ?propLabel

  @@@

  Notice the "culture" property, seems interesting, let's try to look at all the wikidata resources related to the wikidata uris from dbpedia that are linked using this property. 

  @@@sparql

  select ?type ?typeLabel ?resource  ?resourceLabel WHERE { VALUES ?type {  wd:Q16887603  wd:Q10521987  wd:Q4516292  wd:Q337344  wd:Q787057  wd:Q28246
  wd:Q6421966  wd:Q3824923  wd:Q1462523  wd:Q7531335  wd:Q3518019  wd:Q67889726  wd:Q7056762  wd:Q1153959  wd:Q1122064  wd:Q253155  wd:Q40535
  wd:Q1536350  wd:Q1684335  wd:Q3700645  wd:Q7180697  wd:Q432535  wd:Q4879280  wd:Q5692481  wd:Q47542643  wd:Q684109  wd:Q22936344  wd:Q9031060  wd:Q1099570  wd:Q4481150  wd:Q1648270  wd:Q2383661  wd:Q22907154  wd:Q2884882  wd:Q8064284  wd:Q6868659  wd:Q18122904  wd:Q15523428  wd:Q749371  wd:Q1303227  wd:Q7067851  wd:Q6754289  wd:Q766876  wd:Q1894673  wd:Q174361  wd:Q320461  wd:Q1784774  wd:Q4118876  wd:Q2332419  wd:Q253181  wd:Q5715
  wd:Q5348497  wd:Q211286  wd:Q2278032  wd:Q30715241  wd:Q1583849  wd:Q11768
  wd:Q552228  wd:Q2298292  wd:Q2655498  wd:Q2839173  wd:Q4959540  wd:Q816712  wd:Q16303552  wd:Q17011845  wd:Q187462  wd:Q1346230  wd:Q2242136  wd:Q16889471  wd:Q3488951  wd:Q6564382  wd:Q2600111  wd:Q4752834  wd:Q3064288  wd:Q2893435  wd:Q12811105  wd:Q193850  wd:Q18639717  wd:Q22935804  wd:Q32137
  wd:Q4413382  wd:Q4072072  wd:Q4118674  wd:Q7852161  wd:Q182863  wd:Q11041351  wd:Q254093  wd:Q2642660  wd:Q153263  wd:Q2724213  wd:Q4752830  wd:Q3332058  wd:Q5074373  wd:Q1471686  wd:Q34760668  wd:Q19903268  wd:Q582169  wd:Q853296  wd:Q17051361  wd:Q42235
  wd:Q1519552  wd:Q3104807  wd:Q16839161  wd:Q1590754  wd:Q7883361  wd:Q10914393  wd:Q3375464  wd:Q205740  wd:Q4767870  wd:Q907879  wd:Q189574  wd:Q17004916  wd:Q465768  wd:Q4752833  wd:Q22936469  wd:Q3312662  wd:Q1528093  wd:Q4904119  wd:Q129076  wd:Q435967  wd:Q180439  wd:Q6401088  wd:Q3814507  wd:Q7447170  wd:Q56277528  wd:Q19056138  wd:Q263516  wd:Q496002  wd:Q916292  wd:Q60744726  wd:Q2884880  wd:Q7752629  wd:Q5215223  wd:Q16843683  wd:Q997840  wd:Q16209288  wd:Q20649610  wd:Q7112549  wd:Q1430627  wd:Q1637382  wd:Q7735537  wd:Q233458  wd:Q7067835  wd:Q5348471  wd:Q618140  wd:Q871389  wd:Q5445737  wd:Q719699  wd:Q2172038  wd:Q4752832  wd:Q7368288  wd:Q2569896  wd:Q1131955  wd:Q1214882  wd:Q739278  wd:Q47542610  wd:Q24027445  wd:Q1520921  wd:Q17512042  wd:Q29919
  wd:Q22930228  wd:Q5178560  wd:Q2003175  wd:Q22935718  wd:Q853611  wd:Q2210354  wd:Q7575841  wd:Q180927  wd:Q1975262  wd:Q719063  wd:Q430514  wd:Q7738072  wd:Q16890528  wd:Q7114179  wd:Q3783773  wd:Q3576428  wd:Q4920720  wd:Q22909812  wd:Q4165845  wd:Q358209  wd:Q804596  wd:Q1117866  wd:Q772923  wd:Q18129813  wd:Q7784429  wd:Q22936426  wd:Q1413565  wd:Q3552228 }
  ?resource wdt:P2596 ?type
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?type ?typeLabel ?resource  ?resourceLabel

  @@@

  This is a really nice finding, we can see that all these object are related to Ancient Egyptian culture somehow, but let's try to get their type (what are they an instance of)


  @@@sparql
select ?resource  ?resourceLabel ?type ?typeLabel WHERE {
  ?resource wdt:P2596 wd:Q11768;
          wdt:P31 ?type
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?type ?typeLabel ?resource  ?resourceLabel


  @@@

  Let's try to get the unique types so that we can start to have a hierarchy of classes for our ontology.

  @@@sparql

  select distinct ?type ?typeLabel WHERE {
  ?resource wdt:P2596 wd:Q11768;
          wdt:P31 ?type .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?type ?typeLabel ?resource  ?resourceLabel

  @@@

  Let's go even more generic and include the type of the types!

  @@@sparql

  select distinct ?type ?typeLabel ?typetype ?typetypeLabel WHERE {
  ?resource wdt:P2596 wd:Q11768;
          wdt:P31 ?type .
  OPTIONAL {?type wdt:P31 ?typetype }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?type ?typeLabel ?typetype ?typetypeLabel

  @@@


Now I want also to have a sample of these resources to be attached to the query

@@@sparql

select distinct (SAMPLE(?resource) as ?resourceSample) ?type ?typeLabel ?typetype ?typetypeLabel WHERE {
  ?resource wdt:P2596 wd:Q11768;
          wdt:P31 ?type .
  OPTIONAL {?type wdt:P31 ?typetype }
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?resourceSample ?type ?typeLabel ?typetype ?typetypeLabel

@@@

Let's get back to the query about paintings i want to see something related to a deity

@@@sparql

select ?type ?typeLabel ?resource  ?resourceLabel ?propLabel WHERE { VALUES ?type {  wd:Q16887603  wd:Q10521987  wd:Q4516292  wd:Q337344  wd:Q787057  wd:Q28246
  wd:Q6421966  wd:Q3824923  wd:Q1462523  wd:Q7531335  wd:Q3518019  wd:Q67889726  wd:Q7056762  wd:Q1153959  wd:Q1122064  wd:Q253155  wd:Q40535
  wd:Q1536350  wd:Q1684335  wd:Q3700645  wd:Q7180697  wd:Q432535  wd:Q4879280  wd:Q5692481  wd:Q47542643  wd:Q684109  wd:Q22936344  wd:Q9031060  wd:Q1099570  wd:Q4481150  wd:Q1648270  wd:Q2383661  wd:Q22907154  wd:Q2884882  wd:Q8064284  wd:Q6868659  wd:Q18122904  wd:Q15523428  wd:Q749371  wd:Q1303227  wd:Q7067851  wd:Q6754289  wd:Q766876  wd:Q1894673  wd:Q174361  wd:Q320461  wd:Q1784774  wd:Q4118876  wd:Q2332419  wd:Q253181  wd:Q5715
  wd:Q5348497  wd:Q211286  wd:Q2278032  wd:Q30715241  wd:Q1583849  wd:Q11768
  wd:Q552228  wd:Q2298292  wd:Q2655498  wd:Q2839173  wd:Q4959540  wd:Q816712  wd:Q16303552  wd:Q17011845  wd:Q187462  wd:Q1346230  wd:Q2242136  wd:Q16889471  wd:Q3488951  wd:Q6564382  wd:Q2600111  wd:Q4752834  wd:Q3064288  wd:Q2893435  wd:Q12811105  wd:Q193850  wd:Q18639717  wd:Q22935804  wd:Q32137
  wd:Q4413382  wd:Q4072072  wd:Q4118674  wd:Q7852161  wd:Q182863  wd:Q11041351  wd:Q254093  wd:Q2642660  wd:Q153263  wd:Q2724213  wd:Q4752830  wd:Q3332058  wd:Q5074373  wd:Q1471686  wd:Q34760668  wd:Q19903268  wd:Q582169  wd:Q853296  wd:Q17051361  wd:Q42235
  wd:Q1519552  wd:Q3104807  wd:Q16839161  wd:Q1590754  wd:Q7883361  wd:Q10914393  wd:Q3375464  wd:Q205740  wd:Q4767870  wd:Q907879  wd:Q189574  wd:Q17004916  wd:Q465768  wd:Q4752833  wd:Q22936469  wd:Q3312662  wd:Q1528093  wd:Q4904119  wd:Q129076  wd:Q435967  wd:Q180439  wd:Q6401088  wd:Q3814507  wd:Q7447170  wd:Q56277528  wd:Q19056138  wd:Q263516  wd:Q496002  wd:Q916292  wd:Q60744726  wd:Q2884880  wd:Q7752629  wd:Q5215223  wd:Q16843683  wd:Q997840  wd:Q16209288  wd:Q20649610  wd:Q7112549  wd:Q1430627  wd:Q1637382  wd:Q7735537  wd:Q233458  wd:Q7067835  wd:Q5348471  wd:Q618140  wd:Q871389  wd:Q5445737  wd:Q719699  wd:Q2172038  wd:Q4752832  wd:Q7368288  wd:Q2569896  wd:Q1131955  wd:Q1214882  wd:Q739278  wd:Q47542610  wd:Q24027445  wd:Q1520921  wd:Q17512042  wd:Q29919
  wd:Q22930228  wd:Q5178560  wd:Q2003175  wd:Q22935718  wd:Q853611  wd:Q2210354  wd:Q7575841  wd:Q180927  wd:Q1975262  wd:Q719063  wd:Q430514  wd:Q7738072  wd:Q16890528  wd:Q7114179  wd:Q3783773  wd:Q3576428  wd:Q4920720  wd:Q22909812  wd:Q4165845  wd:Q358209  wd:Q804596  wd:Q1117866  wd:Q772923  wd:Q18129813  wd:Q7784429  wd:Q22936426  wd:Q1413565  wd:Q3552228 }
  ?resource wdt:P31 wd:Q178885;
          ?p ?type .
  ?prop wikibase:directClaim ?p .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?type ?typeLabel ?resource  ?resourceLabel ?propLabel

  @@@

  There must be more, let's see if using wikidata complete resource function we can find something more specific

  @@@sparql

  select ?type ?typeLabel ?resource  ?resourceLabel ?propLabel WHERE { VALUES ?type {  wd:Q16887603  wd:Q10521987  wd:Q4516292  wd:Q337344  wd:Q787057  wd:Q28246
  wd:Q6421966  wd:Q3824923  wd:Q1462523  wd:Q7531335  wd:Q3518019  wd:Q67889726  wd:Q7056762  wd:Q1153959  wd:Q1122064  wd:Q253155  wd:Q40535
  wd:Q1536350  wd:Q1684335  wd:Q3700645  wd:Q7180697  wd:Q432535  wd:Q4879280  wd:Q5692481  wd:Q47542643  wd:Q684109  wd:Q22936344  wd:Q9031060  wd:Q1099570  wd:Q4481150  wd:Q1648270  wd:Q2383661  wd:Q22907154  wd:Q2884882  wd:Q8064284  wd:Q6868659  wd:Q18122904  wd:Q15523428  wd:Q749371  wd:Q1303227  wd:Q7067851  wd:Q6754289  wd:Q766876  wd:Q1894673  wd:Q174361  wd:Q320461  wd:Q1784774  wd:Q4118876  wd:Q2332419  wd:Q253181  wd:Q5715
  wd:Q5348497  wd:Q211286  wd:Q2278032  wd:Q30715241  wd:Q1583849  wd:Q11768
  wd:Q552228  wd:Q2298292  wd:Q2655498  wd:Q2839173  wd:Q4959540  wd:Q816712  wd:Q16303552  wd:Q17011845  wd:Q187462  wd:Q1346230  wd:Q2242136  wd:Q16889471  wd:Q3488951  wd:Q6564382  wd:Q2600111  wd:Q4752834  wd:Q3064288  wd:Q2893435  wd:Q12811105  wd:Q193850  wd:Q18639717  wd:Q22935804  wd:Q32137
  wd:Q4413382  wd:Q4072072  wd:Q4118674  wd:Q7852161  wd:Q182863  wd:Q11041351  wd:Q254093  wd:Q2642660  wd:Q153263  wd:Q2724213  wd:Q4752830  wd:Q3332058  wd:Q5074373  wd:Q1471686  wd:Q34760668  wd:Q19903268  wd:Q582169  wd:Q853296  wd:Q17051361  wd:Q42235
  wd:Q1519552  wd:Q3104807  wd:Q16839161  wd:Q1590754  wd:Q7883361  wd:Q10914393  wd:Q3375464  wd:Q205740  wd:Q4767870  wd:Q907879  wd:Q189574  wd:Q17004916  wd:Q465768  wd:Q4752833  wd:Q22936469  wd:Q3312662  wd:Q1528093  wd:Q4904119  wd:Q129076  wd:Q435967  wd:Q180439  wd:Q6401088  wd:Q3814507  wd:Q7447170  wd:Q56277528  wd:Q19056138  wd:Q263516  wd:Q496002  wd:Q916292  wd:Q60744726  wd:Q2884880  wd:Q7752629  wd:Q5215223  wd:Q16843683  wd:Q997840  wd:Q16209288  wd:Q20649610  wd:Q7112549  wd:Q1430627  wd:Q1637382  wd:Q7735537  wd:Q233458  wd:Q7067835  wd:Q5348471  wd:Q618140  wd:Q871389  wd:Q5445737  wd:Q719699  wd:Q2172038  wd:Q4752832  wd:Q7368288  wd:Q2569896  wd:Q1131955  wd:Q1214882  wd:Q739278  wd:Q47542610  wd:Q24027445  wd:Q1520921  wd:Q17512042  wd:Q29919
  wd:Q22930228  wd:Q5178560  wd:Q2003175  wd:Q22935718  wd:Q853611  wd:Q2210354  wd:Q7575841  wd:Q180927  wd:Q1975262  wd:Q719063  wd:Q430514  wd:Q7738072  wd:Q16890528  wd:Q7114179  wd:Q3783773  wd:Q3576428  wd:Q4920720  wd:Q22909812  wd:Q4165845  wd:Q358209  wd:Q804596  wd:Q1117866  wd:Q772923  wd:Q18129813  wd:Q7784429  wd:Q22936426  wd:Q1413565  wd:Q3552228 }
  ?resource wdt:P31 wd:Q146083;
          ?p ?type .
  ?prop wikibase:directClaim ?p .
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?type ?typeLabel ?resource  ?resourceLabel ?propLabel

  @@@

  This list is interesting

  Let's dive deep into Ancient Egypt Mythology and see everything related to that!

  @@@ sparql
  select  ?something ?somethingLabel ?propLabel ?typeLabel WHERE { 
  ?something ?p wd:Q205740 ;
             wdt:P31 ?type .
  ?prop wikibase:directClaim ?p .
  
  SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }} GROUP BY ?something ?somethingLabel ?propLabel ?typeLabel
  @@@




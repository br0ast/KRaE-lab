# Hackathon: Semantic Virtual Curation and Interaction

## Main objective of the hackathon
Develop two ontologies: one about the curation of a virtual exhibition, the second about the possible interactions between a user and the items in the exhibition.

## Requirements

### Requirements about the virtual exhibition ontology

It must say which artistic movements are present in the exhibition, which items are in the exhibition.

Add at least one more requirement of your choice.

### Requirements about the user interaction ontology

The user might express an emotion in this iteraction.

The emotions must follow the DHDK Emotion Theory:
According to this ontology there are 4 primary emotions: Love, Excitement, Sadness, Dislike. There are also 8 secondary emotions: Adoration and Obsession are a particular kind of love. Hype and Enthusiasm are a particular kind of Excitement. Depression and Melancholy are a particular kind of Sadness. Hate and Disgust are a particular kind of Dislike.

Other kinds of interations can be seen in the json below

```json
{ 
    "interaction1":{"text":"User: George. Exhibition: exhibition1. Associates artwork1 to: Family. Associates artwork2 to: The Lord of the Rings. Associates artwork3 to: Digital Humanities"},
    "interaction2":{"text":"User: George. Exhibition: exhibition2. Associates artwork4 to: Trip. Associates artwork5 to: History"},
    "interaction3":{"text":"User: Mary. Exhibition: exhibition1. Associates artwork1 to: Beach. Associates artwork2 to: Forest. Associates artwork3 to: Fire."},
    "interaction4":{"text":"User: Mary. Exhibition: exhibition1. Associates artwork4 to: Hell. Associates artwork5 to: Heaven."}
}
```

***

## Steps to do:

### Create the two TBOXes

Looking at both requirements and the json example: create a TBOX for each ontology in Protégé. Put restrictions if you think they are necessary.

### Create the two ABOXes

#### ABOX of the virtual exhibition ontology

For the ABOX of the virtual exhibition, you'll need to create 2 exhibitions, one of 3 paintings and one of 2 paintings. The requirement is that the items of an exhibition must have the same artistic movement. Here is a query that can help you take a look at the artistic movements in Wikidata:

```sparql
SELECT DISTINCT ?movements ?movementsLabel WHERE { ?painting wdt:P31 wd:Q3305213 ; wdt:P135 ?movements .
                                                  ?movements wdt:P31 wd:Q968159 .
    SERVICE wikibase:label { bd:serviceParam wikibase:language "en". }
}
```
You can modify the query to select the paintings according to the artistic movements you choose.

(Save the query you create for the presentation)

To create the ABOX you will do two **CONSTRUCT** queries (one for each exhibition) in wikidata, that links the artworks you have chosen to your exhibition (including other metadata from wikidata, such as the title and others of your choice), along with the other properties that you have created. An example of a query of that type can be found here

```sparql
prefix ex: <http:example.org/>
CONSTRUCT { ?artwork ex:something ex:Exhibition1 ;
                     ex:somethingelse ?artworkLabel . }
WHERE { VALUES ?artwork { wd:Q12418 wd:Q3211030 }
       SERVICE wikibase:label { bd:serviceParam wikibase:language "en". } }
```

After you download your results as **csv**, you can convert them to a turtle using the function here: [click to open colab](https://colab.research.google.com/github/br0ast/KRaE-lab/blob/main/hackathon1/HACKATHON_auxiliaryfunctions.ipynb)

#### ABOX of the interaction ontology

Once you have finished the abox of the exhibition ontology, **IF YOU PLAN ON CREATING THIS ABOX USING PYTHON**, you should modify the content of the json so that it matches your other abox.

For example this

```json
{ 
    "interaction1":{"text":"User: George. Exhibition: exhibition1. Associates artwork1 to: Family. Associates artwork2 to: The Lord of the Rings. Associates artwork3 to: Digital Humanities"} }
```

could become something like this (you should also think about replacing the dots. Also, you can replace the associations with anything you want)
```json
{ 
    "interaction1":{"text":"User: George; Exhibition: http://exhibitionontology.com/ex1 ; Associates http://www.wikidata.org/entity/Q1234 to: Family; Associates http://www.wikidata.org/entity/Q12345 to: The Lord of the Rings; Associates http://www.wikidata.org/entity/Q123432 to: Digital Humanities"} }
```

Then you should create an algorithm in Python that can convert the content of the modified json into a more structured format, so that then you can create your turtle using rdflib.

After that you can add more triples regarding the information that are not included in the json (for example the emotions, you are free to add the emotions as you like) using the function in the same colab file linked above.


If not you can manually encode the turtle using both the information about emotions and the ones from the json (modifying the URIs and changing the associations...)

### Merging the two ABOXes

Once you have created the ABOX for the interaction ontology and the ABOX for the exhibition ontology, you can merge using the algorithm in the same colab file linked above.

### The rest of the steps

Will be shown next lesson!

# Bottom up Ontology Development

Today, we will use a bottom up approach to create an ontology about people. This is a json file with the starting data for our ontology. 

(Parts of this development methodology are taken directly from known ontology development methodologies such as [SAMOD](https://essepuntato.it/samod/) or [XD](http://extremedesign.sourceforge.net/))

This is the data we are dealing with:

```json
{"Artemis": {"text":"partner: Jeff- offspring: rosa, Vera, Sky"},
 "Jeff": {"text":"partners: Artemis; offspring: Rosa, Vera, sky"},
 "Luna": {"text":"parents: Ember and Rain; siblings: cloud; partner: Stella; offspring: Diamond; dog: Lulu; cats: Sun"},
 "Rain": {"text":"partner: Ember- offspring: Luna and Cloud"},
 "vera": {"text":"parents: Artemis, Jeff; dogs: Fire, Grass and Water- snakes: Happy; siblings: Rosa, Sky"},
"Cloud": {"text":"partner: sky; siblings: Luna"} }
```
***

## Step 1: Look at the data

In this step we will look at the data, both to understand the domain we are dealing with and also to check the syntax of the data,  look for errors or look for the possibility of standardizing something.

### Step 1.1: Content
As for the content, we can say that we are dealing with these types of entities (which might become our classes)
* Person
* Cat – Dog – Snake … All could be subclasses of the class Pet/Animal

Both Person and Cat, Dog, Snakes could be subclasses of Biological Being.

We can also say from the data, that a person can have a partner, siblings, offspring, parents, pets. And that both People and Pets can have names

### Step 1.2: Understanding the syntax
Every key in the first layer of the Json contains a Name of a person, in the second layer there is a text inside the “text” key.
In this text there are some forms of syntax, there are a series of lists introduced by a term and then “:”. 
Lists separators seem to be either a “-“ or a “;”, they can be standardized into one separator, “;” seems to be more fitting. 
Inside the lists there are only Entity names, which are separated by either a “,” or by “and”, this can be standardized to “,”. 
Sometimes the singular/plural is used to represent the same kind of lists: look at “dog” or “dogs”, “partners” or “partner”. We can just keep the plural form. 

We can see that some names don’t always have a Capital Letter, we can standardize it to have everything start with a capital letter.***

*** _Every decision that has been taken about the standardization or choices is subjective, you could replace every separator with “@” or used the singular form or lowercase, it does not matter how you plan to standardize your data, just choose one way that then makes it easier to process it and correct it automatically!_

***
## Step 2  Competency Questions Formulation.
In this step we will create the competency questions considering the things we have noticed in the content analysis.
Competency questions are useful to explain what kind of knowledge you can/want to extract using your ontology.

These are just some competency questions that can be formulated from the data:

**CQ1)** Which people have siblings who have offspring?
**CQ2)** What is the name of the dogs whose owner has parents?
**CQ3)** Which partners don’t have offspring?
**CQ4)** Which offspring have parents who have parents as well?
By looking at the data, you can imagine some expected results:

**CQ1)** Cloud
**CQ2)** “Lulu”, “Fire”, “Grass”, “Water”
**CQ3)** Cloud, Sky
**CQ4)** Diamond

***
## Step 3: Create the TBOX

We can open Protégé and start modeling what we have conceptualized so far
Let’s start from the classes: the main one would be _BiologicalBeing_, superclass of _Person_ and _Pet_. _Pet_ can be the superclass of _Dog_, _Snake_, _Cat_.
Now let’s define some properties.
_Person_ can be linked to _Person_ through many properties:
_hasOffspring/ hasParent_ (inverse), _hasPartner_, _hasSibling_.

Then, _Person_ can be linked to _Pet_ with _hasPet/ hasOwner_ (inverse).
Finally, _BiologicalBeing_ can have a data property _name_ with a literal or string range.
We can choose the prefix _per:_ for our ontology.



***

## Create the ABOX

Now we have our **TBOX**, to get our ABOX we can either encode it manually or we can use a Python Algorithm along with the cleaning of the data algorithm, if you are brave enough to take a look at it it’s in this Colab File.

If not once you are done manually creating you ABOX with the ontology, you can load it and try to query it on the same colab file

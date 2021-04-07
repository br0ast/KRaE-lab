# Assignment

Today you will be asked to develop an ontology using the bottom up approach. You'll start from the data in a json format, and have to do the following things.


***

## First Look

Look at the data, try to imagine possible classes and properties that emerge from the data

***

## Understand the syntax of the data

What are the separators used in the data? in what context are they use? Can you generalize them to process the data automatically? How? Ask yourselves this questions, and it will be easier to adapt the algorithm proposed in class to your data

***

## Develop Competency Questions

By looking at the data, come up with **at least** 3 competency questions. Try to create simple and complex competency questions.

Also try to check what would be the answers to your competency questions if applied to your data.

***

## Take another look at the first conceptualization

See if your conceptualization "theoretically" can answer those competency questions

***

## Start developing the TBOX

Use Protégé to formalized the defined classes, properties and data properties conceptualized before. Pay attention to range and domain! Choose a prefix for the ontology

***

## Create the ABOX

Encode the data that you've worked on at the beginning using your ontology. You can do that manually or re-adapt the algorithm that I have given you during the class. 
**No need to clean the data, it should be clean, but you will need to process it if you want to create the ABOX automatically, by adapting the scripts I have created to the syntax of the new data**

***

## Formalize the competency questions

Create the SPARQL version of your competency questions based on your ontology using the algorithm I have given you during the class. Check whether the results you obtained are the same as the expected results.

***

## Data:

### Group 1:

Here is your data:

```json
{
  "0": {
    "text": "General: latency; potentiality; eternity; non-being; beginning; nothingness; failure; associated with the the infinite, eternity, unity, ether, the archangel Lumiel @ Buddhist: the Void; non-being @ Islamic: divine essence @ Kabalistic: boundlessness; limitless light @ Mesopotamian: associated with totality, the universe @ Pythagorean: the perfect form; the Monad @ Taoist: the Void; non-being @ tarot: the Fool"
  },
  "7": {
    "text": "General: completeness; totality; perfection; reintegration; conflict; pain; the moon @ alchemic: the seven metalsinvolved in the Work @ Assyrian: specialization -> pine with seven branches and seven buds: Tree of Life; completeness @ astrologic: rays of the sun @ tarot: the Chariot"
  }
}
```

### Group 2:

Here is your data:

```json
{
  "6": {
    "text": "General: associated with divine power, majesty, justice, creation, perfection, balance, material comfort, education, marriage, institutions, harmony, responsibility, beauty, art, time, happiness, health, luck, evolution @ Kabalistic: experiment; creation; beauty @ Mayan: an unlucky number; associated with death @ Occidental: winning number in dice @ Pythagorean: chance; @ tarot: the Lovers"
  },
  "13": {
    "text": "General: unlucky; associated with the archangel Raphael, betrayal, faithlessness, misfortune, death, birth, purification @ Aztecan: time; the days in a week; associated with divination, the skies @ Christian: Judas Iscariot; the 13 Tenebrae, people at the Last Supper @ Kabalistic: associated with snakes, dragons, murderers, Satan @ tarot: Death"
  }
}


```

### Group 3:

Here is your data:

```json
{
  "10": {
    "text": "General: the perfect number; associated with the cosmos, return to unity, balance, completeness, kingship, infinite power, perfection, finality, androgyny, marriage, spiritual achievement, order, involution and evolution, the archangel Lumiel @ Buddhist: the ten moral duties of the code of Manou @ Babylonian: associated with Marduk @ tarot: Wheel of Fortune"
  },
  "20": {
    "text": "General: continuity; dynasty; an intensifier; the whole man; an indefinite number; shares the general symbolism of 10 @ Kabalistic : physical strength @ Mayan: associated with the sun god; days of a month in the religious calendar; the base of their number system; Primal Oneness @ tarot : Judgment"
  }
}
```

### Group 4:

Here is your data:

```json
{
  "16": {
    "text": "General: associated with happiness, luxury, love, sensuality, fertility, increase, the archangel Samael, Lucifer @ Buddhist: the 16 Arhats @ Hindu: the 16 petals of the Visuddha chakra @ Jainism: the 16 goddesses @ tarot: House of God"
  },
  "19": {
    "text": "General: lucky; associated with the archangel Michael @ Japanese: misfortune @ tarot: Sun"
  },
  "21": {
    "text": "General: absolute truth; a lucky number; a sacred number; traditional age of majority; associated with the archangel Cassiel, excellence, harmony of creation @ Christian: union of the Trinity @ Italian: letters in the alphabet @ Jewish: perfection @ tarot: World"
  }
}
```

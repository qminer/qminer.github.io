---
layout: page
title: Anomaly detection
---

# Anomaly detection

Various types of anomaly detection can be created using QMiner platform. 

## Anomalies in text stream

Prototype for novelty detection in a news-stream, based on data from Event Registry. The prototype is using QMiner's anomaly detection stream aggregate based on k-Nearest Neighbor algorithm.

We firstly import all relevant libraries.

````javascript
let qm = require('qminer');
let fs = require('fs');
let readline = require('readline');
let eachline = require('eachline');
````

Define the storage schema. We define one store called 'articles' ...

````javascript
let base = new qm.Base({
    mode: 'createClean',
    schema:[
        {
            name: 'Articles',
            fields: [
                { name: 'Time', type: 'datetime' },
                { name: 'Text', type: 'string' },
                { name: 'Title', type: 'string' },
                { name: 'Number', type: 'float' }
            ]
        }
    ]
});

let articles = base.store('Articles');
````

... a feature space aggregator on the article store.

````javascript
let aggrFSA = {
   name: "ftrSpaceAggr",
   type: "featureSpace",
   initCount: 2,
   update: true, full: false, sparse: true,
   featureSpace: [
        { 
            type: "text", source: "Articles", field: 'Text',
            weight: 'tfidf', // none, tf, idf, tfidf
            tokenizer: {
                type: 'simple',
                stopwords: 'en', // none, en, [...]
                stemmer: 'porter' // porter, none
            },
            ngrams: 2,
            normalize: true
        }
    ]
}; 

let ftrSpaceAggr = articles.addStreamAggr(aggrFSA);
````

We use a 'fake' time series tick stream aggregator for the 'Articles' store to trigger feature space aggregate. This is why articles store has a 'fake' numeric field 'Number'.

````javascript
let aggrT = {
    name: "tickAggr",
    type: "timeSeriesTick",
    store: "articles",
    timestamp: "Time",
    value: "Number"
};
//create the tick aggregator
let tickAggr = articles.addStreamAggr(aggrT);
````

Novelty detection is implemented using Nearest Neighbors anomaly detector aggregator.  Aggregator is used on the articles store and takes timestamped features as input. The time stamp is provided by the tick aggregator while the feature vector is provided by the feature space aggregator.

````javascript
let aggrAD = {
    name: 'AnomalyDetectorAggr',
    type: 'nnAnomalyDetector',
    inAggrSpV: 'ftrSpaceAggr',
    inAggrTm: 'tickAggr',
    rate: [0.2, 0.05, 0.01],
    windowSize: 200
};

// Create the anomaly detection aggregator
let anomaly = articles.addStreamAggr(aggrAD);
````
 
Here we define our monitoring stream aggregate.

````javascript
// Define monitoring stream aggregate.
let monitoringAggr = articles.addStreamAggr({
    onAdd: (rec) => {        
        if (anomaly.getInteger() > 2) console.log("Rate " + anomaly.getInteger() + ": " + rec["Title"]);        
    }
});
````
Articles are loaded from our server. 6 different sets are available. Note that stemmer and stopwords need to be changed for Slovenian language.

````javascript
// Define global variable for storing results.
let results = [];
let rates = [];
let count = [0, 0, 0, 0];

// load articles into the store via simulated stream
let ARTICLES_URL = "http://atena.ijs.si/data/novelty/PeterPrevcENG.json";
// let ARTICLES_URL = "http://atena.ijs.si/data/novelty/PeterPrevcSLV.json";
// let ARTICLES_URL = "http://atena.ijs.si/data/novelty/BorutPahorENG.json";
// let ARTICLES_URL = "http://atena.ijs.si/data/novelty/BorutPahorSLV.json";
// let ARTICLES_URL = "http://atena.ijs.si/data/novelty/MicrosoftENG.json";
// let ARTICLES_URL = "http://atena.ijs.si/data/novelty/EuropeanComissionENG.json";

let got = URL => require("got")(URL, { json : false });
let response = await got(ARTICLES_URL);
let lines = response['body'].split('\n');

lines.forEach(line => {
    if (line != "") {
        let currentArticle = JSON.parse(line);
        articles.push({ Time: currentArticle["date"] + "T" + currentArticle["time"], Text: currentArticle["body"], Title: currentArticle["title"], Number: 1});
        results.push({ rate: anomaly.getInteger(), title: currentArticle["title"] });
        rates.push(anomaly.getInteger());
    }
});
````

Display all the results ...

````javascript
console.log(JSON.stringify(results));
````

Display counts (you can visualize histogram/chart);

````javascript
rates.forEach(rate => count[rate]++);
console.log(JSON.stringify(count));
````

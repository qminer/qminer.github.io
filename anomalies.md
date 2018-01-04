---
layout: page
title: Anomaly detection
---

# Anomaly detection

By anomaly, we refer to a pattern that is unusual, unexpected or rare in the data under analysis. QMiner can detect anomalies on several types of static or dynamic data.

## Anomalies in text streams

Let's try to detect novel topics published in the media. Novel (or "anomalous") topics are topics that were never or rarely observed in the past. We assume that there is a stream of articles being pushed to the system. 

Let's first import all the relevant libraries.

```javascript
let qm = require('qminer');
let fs = require('fs');
let readline = require('readline');
let eachline = require('eachline');
```

Then we define the store called "Articles" having a schema with four fields.

```javascript
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
```

Next we define the pipeline that processes the news articles. The pipelines has several processing modules: the model, the trigger, the anomaly detection algorithm and the actual alert.

To keep a model of the past articles, we need feature space aggregator on the article store. The model will use the text from the "Text" of the article and will use "tfidf" weight to compute the importance of various words. The model will need at least 2 articles to be initialized and will consider pairs of two words (the n-grams) parameter.

```javascript
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
```

This prototype examines articles at a constant rate. We use an artificial time series to simulate this constant rate. We implement it using the tick stream aggregator for the 'Articles' store. The model defined before updates at this rate.

```javascript
let aggrT = {
    name: "tickAggr",
    type: "timeSeriesTick",
    store: "articles",
    timestamp: "Time",
    value: "Number"
};
//create the tick aggregator
let tickAggr = articles.addStreamAggr(aggrT);
```

The novelty detection is implemented using the Nearest Neighbors anomaly detector aggregator. The aggregator takes timestamped features as input. The time stamp is provided by the previously defined tick aggregator and feature space aggregator. It considers a window of 200 most recent articles and computes the distances between these. The most distant articles are considered as anomalies. The rate of anomalies can be tuned using the "rate" parameter - play with it on RunKit!

```javascript
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
```
 
The alert monitoring stream aggregate is a custom defined stream aggregate that outputs alerts with severity levels above 2. 

```javascript
// Define monitoring stream aggregate.
let monitoringAggr = articles.addStreamAggr({
    onAdd: (rec) => {        
        if (anomaly.getInteger() > 2) console.log("Rate " + anomaly.getInteger() + ": " + rec["Title"]);        
    }
});
```

Now that the processing pipeline is defined, we can start pushing the stream of data in the system Articles are loaded from our server.

```javascript
// Define a global variable for storing the results.
let results = [];
let rates = [];
let count = [0, 0, 0, 0];

// For the example, we use English articles from the intyernational media about the sky champion Peter Prevc. Other data sets are also available.
let ARTICLES_URL = "http://atena.ijs.si/data/novelty/PeterPrevcENG.json";
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
```

Display all the results.

```javascript
console.log(JSON.stringify(results));
```

Display counts (you can visualize histogram/chart).

```javascript
rates.forEach(rate => count[rate]++);
console.log(JSON.stringify(count));
```

# Stream Processing Patterns

## Source

Produces to Kafka, does not consume from Kafka.

## Sink

Consumes from Kafka, does not produce to Kafka.

## Processor

Consumes from Kafka and produces back to a different topic.

### Enrichment Processor

Consumes from Kafka, enriches the data (typically by calling out to an API), and produces back to a different topic.

An enrichment processor can often cache the responses from the API, especially if the data is not subject to change.  
If the API response can be different based on when the call is made, it's recommended to avoid caching or to be extremely conscious of evicting the cache when it makes sense.

## Single-trigger combination processor

Consumes from multiple Kafka topics, and produces back to a different topic.

Whenever there is a new record on topic 2, it is stored for later use. Typically, new records would overwrite previous records.  
Whenever there is a new record on topic 1, it is combined with the latest data from topic 2 and emitted.  

![k latest from topic 1](https://user-images.githubusercontent.com/7855170/139717378-bda88b11-6810-4aa5-bc84-480c0015705c.png)

## Any-trigger combination processor

Whenever there is a new record on topic 1 or 2, it is stored for later use. Typically, new records would overwrite previous records.  
Whenever there is a new record on topic 1 or 2, it is combined with the latest data from the other topic and emitted.  

![k combine](https://user-images.githubusercontent.com/7855170/139717575-9c93f49f-9f0b-4902-b5df-0e865e86924e.png)

## Paired-trigger combination processor

Whenever there is a new record on topic 1 or 2, it is stored for later use. New records would be stored along side previous records without overwriting.  
Whenever there is a match, the data is combined and emitted. The records used in the match are then removed from the store.

![k zip](https://user-images.githubusercontent.com/7855170/139717554-a4ac63fa-09d3-4003-920c-21ce524bf1ab.png)

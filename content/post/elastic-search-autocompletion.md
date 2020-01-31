---
Title: "Elasticsearch Typeahead 101"
Date: 2019-09-23
Lastmod: 2019-09-23
Author: Wickstargazer
Description: Concepts and practice on How to Elasticsearch Typeahead (Auto-complete) 101 with the Thai Language.
image: post/images/elasticsearch_typeahead.jpg
Categories: ["Elasticsearch", "Typeahead", "Autocomplete"]
Tags: ["Web Development","Elasticsearch", "Typeahead", "Autocomplete"]
Draft: false
MinuteRead: 4 MIN READ
---

*In this article, we will talk about talk about how to Elasticsearch Typeahead (Auto-complete) 101  with the Thai Language*

In lean development of a product we never plan for scale, but when things are going well performance starts to become a technical debt that we cannot look over and overcome with quick fixes.

#### Journey of the noob!
When starting off your first startup its never in the skill-set to plan just enough to finish a product that scale. We ended up with quick fixes, un-optimize code and infrastructure all across the system. Someone has told me that it takes a master skill-set to plan the right amount of work to be done to achieve an 80% scalable code/infrastructure. When its your first startup its always rough and tough, count me in as one of those noobs who does not know any better!

One of the nubie bomb is when the database starts to explode...So we turn around, there is no one exactly to help out except Google and the magic word "Caching". Users are knocking away and nagging away while we do the rough and tough research and development to come up with a sustainable solution!
![db_explide](/post/images/db_explode.jfif)

#### Time to grow up.
So we got googling and try to every other code tune-up in hopes of to live another day until eventually we know that Caching &Searching infrastructure is the dang thing we'd need to get things back on track, but we have never graze by the search engine technology before so we get all confused with the documentations and the emmense power that it can bring and do not know where to start.

*So we do the getting started tutorial. We got started!*
*Then do it the startup way, one iteration at a time.*

#### The basic indexing
Never hearing of the  **Token Filters** and **Tokenizers** we made the most common mistakes ... using the **Ngram Filter + Standard tokenizer** to construct the index setting thinking it will be similar to the `Like %{xx}%` query and then Bang!. Users are searching for their products or contacts and getting blanks or terms showing and then disappearing after the next character.

Take for instance the word **น้ำเฉาก๊วย**, when searching we get this behaviour.
```
"my_first_tokenizer": {
          "type": "ngram",
          "min_gram": 1,
          "max_gram": 10,
          "token_chars": [
            "letter",
            "digit",
            "whitespace"
          ]
        }
```
```
"น้ำเ" ==> "น้ำเฉาก๊วย"
"น้ำเฉ" ==> ""
"น้ำเฉา" ==> ""
"น้ำเฉาก" ==> "น้ำเฉาก๊วย"
```
![noob](/post/images/noob.jpg)
### The journey that completes
So yes I am a noob. After few sleepless nights and a bunch of kibana queries later ... I have gone over the edge and came up with a workable solution. here it is!

**The Settings**

Basically I found out that there are a gazillion settings we can use on elasticsearch and it depends on how you use them to achieve your goal! There are suggestion specific settings as well such as the **"suggest"** query but it still has it difficulty in configuration and I personally could not find a way for the **Contain** query to work the way we want to. To contain search a term in the phrase/word. So I went with the following settings instead.
```
"analysis": {
      "filter": {
         "autocomplete_filter": {
          "type": "edge_ngram",
          "min_gram": 1,
          "max_gram": 255,
          "side": "front"
        }
      },
      "analyzer": {
        "autocomplete": { 
          "type": "custom",
          "tokenizer": "standard",
          "filter": [
           "lowercase",
            "autocomplete_filter"
          ]
        }
        "autocomplete_thai": { 
          "type": "custom",
          "tokenizer": "icu_tokenizer",
          "filter": [
            "lowercase",
            "autocomplete_filter"
          ]
        }
      }
    }
  },
  "mappings": {
    "productesmodel": {
      "properties": {
        "name": {
          "type": "text",
          "analyzer": "autocomplete",
          "search_analyzer": "standard",
          "fields": {
            "keyword": { 
              "type":  "keyword",
              "doc_values": true,
              "normalizer": "lowercase_normalizer"
            }
          }
        },
        "nameth": {
          "type": "text",
          "analyzer": "autocomplete_thai",
          "search_analyzer": "standard",
          "fields": {
            "keyword": { 
              "type":  "keyword",
              "doc_values": true,
              "normalizer": "lowercase_normalizer"
            }
          }
        }
      }
    }
  }
```
**The Query**

By defining the two fields that has different tokenizers, one is the **icu_tokenizer** while the other **standard** with an **edge_gram** filter, we achieve a quiet nice autocomplete for our fields. However the query have to search through two fields otherwise the conjoining character won't return any search results. Like below if i remove the **"name"** field, there wont be any result because the **icu_tokenizer** already decides there is only two words in the **"name_th"** field.

```
POST /index/productesmodel/_search
{
  "query" : {
    "multi_match" : {
      "query":  "น้ำเ",
      "type":   "phrase",
      "fields": ["nameth","name"]
    } 
  }
}
```
**Valhalla!! The Result**
```
"น้ำเ" ==> "น้ำเฉาก๊วย"
"น้ำเฉ" ==> "น้ำเฉาก๊วย"
"น้ำเฉา" ==> "น้ำเฉาก๊วย"
"น้ำเฉาก" ==> "น้ำเฉาก๊วย"
```

#### Conslusion

There are a million things you can do with Elasticsearch engine, this is just scratching the surface of its power. Hence its a 101 tutorial! Hope you enjoy reading and get digging deeper becuase the new power is Data. Next step is to get the score and the weights into play with the query and suggestions to show more relevancy matching!
---
Title: "Building a generic search/sort"
Date: 2019-01-20
Lastmod: 2019-01-20
Author: Wickstargazer
Description: How we make it generic for multiple models across the platform
Image: /post/images/search_sort.jpeg
Cover: https://wickstargazer.com/post/images/earch_sort.jpeg
Categories: ["Typescript", "Generics"]
Tags: ["Typescript", "ES6", "Generics"]
Draft: false
MinuteRead: 3 MIN READ
---

*Searching and sorting is the basic building blocks for a web-application. Lets look at how we make it generic for multiple models across the platform. I will focus on using Typescript and .Net technology in this one.*

<!--more-->

#### Technology Stack
1. Typescript ES6
2. C# Linq Expression Builder

------------

Before jumping into the technical parts of developing a search sort functionality, lets list out the aspects on which how it should work and to what generic coverage.

1. The data is persisted in the database (for relational DB, we will also generalize the search in FK navigational properties)
2. The data will be cached on a search engine somewhere
3. The data will also be cached partially on the front-end

This reveals an important design factor, i.e the name of the columns/fields must be mutable between the front-end models, search engine models and database models. I can tell you that I have made this mistake many a times because of sheer laziness! 😞

------------

#### OK lets start!
Lets pick a common example, e-commerce products and its sales. I am going to go with a relational DB here just for the sake of me googling lesser 😏
![db design](/post/images/search_sort_db_design.jpeg)

So we got some models to play with, and after building a simple grid in the front-end .. we can start writing up the models that will do the dirty work.

We usually use a query model which is mutable from json to a generic query model in the back-end (API).
```
export class Query {
    totalRecords: number;
    pageSize: number;
    currentPage: number;
    filter: Array<FilterOption>
    sortBy: Array<SortOption>
}
```

With the following model for **FilterOptions** and **SortOptions**

```javascript
export class FilterOption {
    columnName: string;
    columnValue: string;
    columnPredicateOperator: string;
}
export class SortOption {
    name: string;
    sortOrder: string;
}
```

Simple ain’t it? 😛 Note: The array allows multiple sorts/search.

Now its time to setup the options so we can allow user to search or sort something from the product/category/order data! Anything!

```javascript
query = new Query();
query.filter.push(new FilterOption("PName", "Cookie", "and"));
query.filter.push(new FilterOption("Sale.DeliveryAddress", "Cookie, "or"));
query.filter.push(new FilterOption("BarCode.BarCode", "Cookie, "or"));
```

Now we send this request to our front-end collection or the back-end (Who is responsible to choose if it is to be selected from elastic-search or DB)

In this Log i am going to show only going to show how we implemented the expression builder, why? Because its so cool! 😆
So once we de-serialize our request, and get the Query as a C# Object, we can pass it onto the query builder. which goes like this.

#### Expression Builder for FilterOptions
<script src="https://gist.github.com/wickstargazer/3ead75ca16449694ae3871840a218865.js"></script>

#### Expression Builder for SortOptions
<script src="https://gist.github.com/wickstargazer/58dd8869095692616b60672c54885649.js"></script>

We use the dynamic LINQ library for the sort option as it just need the order by functionality unlike the filter where we need to build a where clause using the predicate and also to build a where query for the correct strong-typed properties. Hence we end up with the Expression tree.

------------

#### In conclusion
The main focus of the technique is at the expression builder which can be written for other types of data persistent layers and in other languages using the same concept. In this one we used LINQ and C# as an example.
The front-end plays the part of sending in the right combination of search queries entered by the user.

Please do not hesitate to share or comment on what you think should be added to the technique or if there is a better expression tree builder out there that conforms with the .net framework! 😄

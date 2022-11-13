Title: Datalog
Date: 2022-11-13
Tags: clojure

In the world of databases, Datalog is clearly not the most famous.
However, it’s almost the biggest selling point of people selling _Clojure_: Datomic.
I had a hard time understanding what Datomic _was_ exactly.
I grew up with relational and document oriented databases, and I did not get where Datomic fit in all of this.

Is it a technology? Is it a database? What’s an «ion»? Do I query it with SQL? What does the data _look like_ there?

It does not help that I did not know Clojure and that it was not a `brew install datomic` away.
You have to go to the downloads page and it’s not clear if you have to pay for it or not.

Comparing it with Postgres, where the documentation is comprehensive and it’s easy to install on macOS:
```
brew install postgresql
brew services start postgresql
```

And you’re basically there.

However, it’s fitting the OneTrain project I am currently working on, so I thought it would be a good idea to explore.

## History
[Rich Hickey](https://en.wikipedia.org/wiki/Rich_Hickey) is the creator of Clojure and guru of the community, and he is also the creator of [Datomic](https://www.datomic.com).
With Clojure he thought he would solve programming (at least for him) and with Datomic databases.

According to [Wikipedia](https://en.wikipedia.org/wiki/Datomic), the project was first released in 2012 and is:
> [...] a distributed database and implementation of Datalog.
It has ACID transactions, joins, and a logical query language, Datalog.
A distinguishing feature of Datomic is that time is a basic feature of data entities.

## Notes on Max-Datom
It’s a very good tutorial overall.

> Datomic is built from atomic facts called datoms.
A datom is a finite ordered list of 5 elements referred to as a tuple.
The five values in every datom are the following: [<entity-id> <attribute> <value> <transaction-id> <operation>].
datomic.client.api/q performs the query described by the provided query and args then returns a collection of tuples.
– https://max-datom.com

```clj
(ns leve-x
  (:require [datomic.client.api :as d]
            [max-datoms.connections :refer [db])))

(d/q '[:find pull(?e [*])
       :where [?e :author/id #uuid "foobar"]])
;; Retrieves (pulls) all attributes for the entity that has the author ID "foobar".
```

```clj
(ns level-8
  (:require
    [datomic.client.api :as d]
    [max-datom.connections :refer [db]]))
​
(def author-id #uuid "35636B79-EE46-4447-8AA7-3F0FB351C45C")
​
(d/q '[:find (pull ?e [:author/first-name :author/last-name])
       :in $ ?author-id ;; This is the part where we parameterize the query.
       :where [?e :author/id ?author-id]] db author-id)
```

```clj
(ns level-9
  (:require
    [datomic.client.api :as d]
    [max-datom.connections :refer [db]]))

(def author-ids [#uuid "0955EDF7-FF8F-4EC2-AFB2-380E7E5D48D7"
                 #uuid "B7761785-79F9-49FA-97AF-13B4F5C2BCC2"])

(d/q '[:find (pull ?e [:author/first-name :author/last-name])
       :in $ [?author-id ...]
       :where [?e :author/id ?author-id]] db author-ids)
```

```cljs
(ns level-10
  "Pulls the user-name and counts their posts."
  (:require
    [datomic.client.api :as d]
    [max-datom.connections :refer [db]]))
​
(d/q '[:find  ?user-name (count ?posts)
       :where [?user :user/id #uuid "1B341635-BE22-4ACC-AE5B-D81D8B1B7678"]
              [?user :user/first+last-name ?user-name]
              [?posts :post/author ?user]] db)
```

```clj
(ns level-11
  "The not clause"
  (:require
    [datomic.client.api :as d]
    [max-datom.connections :refer [db]]))

(d/q '[:find  (count ?post) ?user-name
       :where [?user :user/first+last-name ?user-name]
              [?post :post/author ?user]
              (not [?post :post/dislikes])] db)
```

```clj
(ns level-12
  "J’en aurais chié pour celui-là."
  (:require
    [datomic.client.api :as d]
    [max-datom.connections :refer [db]]))
​
(defn comment-count-str [x]
  (str "Comment Count: " (count x)))
​
(d/q '[:find  (pull ?posts [{:post/author [:user/first+last-name]}
                            [:post/comments :xform level-12/comment-count-str]])
              
       :where [?posts :post/author _]] db)
```

Stopped at level 13: custom aggregates.

### Datalog
Datalog is the query language, it’s comparable to SQL. It’s the language we use to talk to the database.

## References
* [Max-Datom](https://max-datom.com/#/E3875311-58D1-497C-9AA7-94CEF884561E): an interactive tutorial on Datomic
* [Datomic](https://www.datomic.com)’s webpage
* [A YouTube talk on Datalog](https://www.youtube.com/results?search_query=datalog+demo+conference) by Norbert Wojtowicz

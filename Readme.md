# Purpose

This plugin can be used to

 * update all or selected documents of an index, e.g. after you change the settings of a type
 * to change the index settings like shard count: create a new index with that config and reindex all documents into that index
 * grab all or selected documents from another elasticsearch cluster and feed your local machine with that
 * and more like a filtered backup etc

# License

Apache License 2.0


# Installation

> mvn -DskipTests clean package

> // TODO since github remove download TAB we need to find an alternative hosting solution!

> // sudo $ES_HOME/bin/plugin install karussell/elasticsearch-reindex

> // or the local one: sudo $ES_HOME/bin/plugin -url ./target -install reindex

> sudo service elasticsearch restart

you should see 'loaded [reindex], sites []' in the logs. Or use the reinstall.sh script for development purposes


# Deinstallation

> sudo $ES_HOME/bin/plugin remove reindex

> sudo service elasticsearch restart


# Usage

## WARNINGs / TODOs:

 * Please try this on your local machine before using it in production - especially the case searchHost!=localhost could be problematic for your performance/IO
 * The call is not async and not stopable (except you stop the requested server) => The plugin should probably be better a river
 * If you have two servers on localhost and the queried server port is 9201 and you want to search
   the different server at 9200 => then you have to use e.g. searchHost=127.0.0.1&searchPort=9200

## Same cluster 

> curl -XPUT 'http://localhost:9200/indexnew/typenew/_reindex?searchIndex=indexold&searchType=typeold' -d '
>  { "term" : { "count" : 2 } }'

This refeeds all documents in index 'indexold' with type 'typeold' into the index 'indexnew' with type 'typenew'.
But only documents matching the specified filter will be refeeded. The internal Java API will be used which should
should be efficient.

## Different cluster 

Now JSONObjects and the HttpClient will be used. TODO that is probably not efficient in terms of RAM/CPU?!:

> curl -XPUT 'http://localhost:9200/indexnew/typenew/_reindex?searchIndex=indexold&searchType=typeold&searchHost=yourElasticsearchHost.com&searchPort=9200' -d '
>  { "term" : { "count" : 2 } }'

Further parameters:
 * hitsPerPage - used as search limit and at the same time for bulk indexing (default 100)
 * keepTimeInMinutes - the maximum time in minutes a scroll search is valid (default 30) increase if you have more data
 * withVersion - if the version of a document should be respected (default false)
 * waitInSeconds - pause the specified time after every request pair (one search+one bulkIndex). 
   This avoids heavy load on the search or on the indexing server/cluster. This way it is very easy
   e.g. to grab even a massive amount of data from your production servers into your local machine.

Hints:
 * the index 'indexnew' and the type 'typenew' should exist.
 * the parameters 'searchIndex' and 'searchType' are optional and the new ones will be used if not provided
 * the filter is also optional

---
layout: post
title: Migrating an Elasticsearch index
date: 2014-09-10
categories:
 - nerd
tags:
 - elasticsearch
 - elastic4s
 - scala
---

Migrating an [ElasticSearch](http://elasticsearch.org) index can be challenging. 
Updating mappings is important as they are the foundation for doing more than 'Hello World!' when doing search.
ElasticSearch is quite good at guessing the type of a field, but it's just a start. 

The ``UpdateIndex`` class uses [Elastic4s](http://elastic4.sgithub.com) as a scala DSL on top of the regular ElasticSearch Java API.
Elastic4s is a lot closer to the ElasticSearch REST API than the regular API. 
It also handles moving data from the old index over to the new.  

Code
----

```
package bootstrap

import scala.annotation.implicitNotFound
import scala.concurrent.Await
import scala.concurrent.ExecutionContext
import scala.concurrent.Future
import scala.concurrent.duration.DurationInt

import org.elasticsearch.indices.IndexMissingException
import org.ladderframework.logging.Loggable

import com.sksamuel.elastic4s.ElasticClient
import com.sksamuel.elastic4s.ElasticDsl.aliases
import com.sksamuel.elastic4s.ElasticDsl.create
import com.sksamuel.elastic4s.ElasticDsl.deleteIndex
import com.sksamuel.elastic4s.ElasticDsl.matchall
import com.sksamuel.elastic4s.ElasticDsl.search
import com.sksamuel.elastic4s.mappings.MappingDefinition
import com.sksamuel.elastic4s.mappings._

class UpdateIndex(client: ElasticClient, indexName: String, types: Seq[MappingDefinition])(implicit ec: ExecutionContext){
	
	def info(msg: String){
          println(msg) // You can do better :-)
        }

	def initIndex(): Future[Unit] = {
		info("init index")
		client.execute{
			search (indexName) query matchall
		}.map(_.getHits.getTotalHits).recover{case _: IndexMissingException => 0}.flatMap(numberOfDocs => {
			info("number of docs: " + numberOfDocs)
			info("number of docs: " + numberOfDocs)
				client.execute{
					info("create index")
					create index indexName mappings (types:_*)
				}.map(_ => {})
			} else {
				val date = System.currentTimeMillis()
				val newIndex = indexName + date
				for{
					_ <- client.execute{
						info("create brand new index")
						create index newIndex mappings (types:_*) 
					}
					_ <- {
						info("reindex")
						client.reindex(indexName, newIndex)
					}
					_ <- {
						client.execute(deleteIndex(indexName))
					}
					_ <- {
						info("create aliases")
						client.execute{aliases add indexName on newIndex}
					}
					newSt <- client.execute{search (indexName) query matchall}
				} yield {
					info("done update index - Moved docs: " + newSt.getHits().getTotalHits())
				}
			}
		})
	}
	
	def createOrUpdateIndexes(): Unit = {
		try{
			val res = Await.result(initIndex(), 3 minutes)
			info("DONE: " + res)
		} catch{
			case t: Throwable => t.printStackTrace()
		}
		
	}
	
}
```

Disclaimer
----------

This code is used on a limited index on [is it down.no](http://isitdown.no) but it workes great there..
So handle with care.
Good feedback is appreciated.

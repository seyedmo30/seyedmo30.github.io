
PUT /test13
{
  "mappings": {
    "dynamic": "false",
    "_source": {
      "includes": [
        "authors",
        "category",
        "content.main_content",
        "content.summary",
        "content.title",
        "country",
        "create_date",
        "docsentiment.negative",
        "docsentiment.neutral",
        "docsentiment.positive",
        "grade",
        "annotation_types.count",
        "annotation_types.name",
        "annotation_types.name_fa",
        "annotation_types.wiki_id",
        "annotation_types.entity_type",
        "ner_done",
        "nerlocation.coordinate",
        "nerlocation.name",
        "orientation",
        "site",
        "summary",
        "summary_fa",
        "title",
        "title_fa"
      ]
    },
    "properties": {
      "authors": {
        "type": "keyword"
      },
      "category": {
        "type": "keyword"
      },
      "content": {
        "properties": {
          "main_content": {
            "type": "text"
          },
          "main_pic": {
            "type": "keyword",
            "index": false
          },
          "summary": {
            "type": "text"
          },
          "title": {
            "type": "text"
          }
        }
      },
      "country": {
        "type": "keyword"
      },
      "create_date": {
        "type": "date"
      },
      "docsentiment": {
        "properties": {
          "negative": {
            "type": "float"
          },
          "neutral": {
            "type": "float"
          },
          "positive": {
            "type": "float"
          }
        }
      },
      "grade": {
        "type": "byte"
      },
      "annotation_types": {
        "properties": {
          "count": {
            "type": "byte" ,
            "index": false
          },
          "name": {
            "type": "keyword"
          },
          "name_fa": {
            "type": "keyword"
          },
          "wiki_id": {
            "type": "keyword"
          },
          "entity_type": {
            "type": "keyword"
          }
        }
      },
      "ner_done": {
        "type": "boolean"
      },
      "nerlocation": {
        "properties": {
          "coordinate": {
            "type": "geo_point"
          },
          "name": {
            "type": "keyword"
          }
        }
      },
      "orientation": {
        "type": "keyword"
      },
      "site": {
        "type": "keyword"
      },
      "summary": {
        "type": "text"
      },
      "summary_fa": {
        "type": "text"
      },
      "title": {
        "type": "text"
      },
      "title_fa": {
        "type": "text"
      },
      "url": {
        "type": "keyword",
        "index": false
      }
    }
  }
}


GET test13/_mapping

POST backup_feeds/_delete_by_query
{
  "query": {
    "match": {
      "common_key.keyword": "02d28e38-45c0-435a-9fc4-d4fb9031eecf"
    }
  }
}

DELETE test13

POST _reindex
{
  "source": {
    "size": 1000, 
        "index": "feeds",
      "query": {
          "query_string": {
            "fields": ["category"],
            "query": "*"
          }
        }
  },
    "dest": {
    "index": "test1_backup"
  }
  
}


POST _reindex
{
  "source": {
    "size": 1000,
    "index": "backup_feeds"
  },
  "dest": {
    "index": "test13"
  },
  "script": {
     "lang": "painless",
    "source": """
      // Remove common_key if present and use it as the document ID
      if (ctx._source.containsKey('common_key')) {
        ctx._id = ctx._source.remove('common_key');
      }
      
      // Rename 'title' to 'title_fa' and remove 'title' field
      if (ctx._source.containsKey('title')) {
        ctx._source.title_fa = ctx._source.remove('title');
      }
      
      // Convert 'create_date' from Unix timestamp to ISO 8601 string
      if (ctx._source.containsKey('create_date')) {
        long millis = (long) (ctx._source.create_date * 1000);
        Instant instant = Instant.ofEpochMilli(millis);
        ctx._source.create_date = instant.toString();
      }
      
    
      // Transform 'nerlocation' field
      if (ctx._source.containsKey('nerlocation')) {
        
        
  
        def nerlocation = ctx._source.nerlocation;
        
        // Check if 'coordinate' exists and has valid lat and lon
        if (nerlocation.containsKey('coordinate')) {
          
               def coordinate = ctx._source.nerlocation.coordinate;
        
        
          if (coordinate.containsKey('lat') && coordinate.containsKey('long')) {
          
          def lat = nerlocation.coordinate.lat   != null ? nerlocation.coordinate.lat : 0.0;
     
            def lon = nerlocation.coordinate.long != null ? nerlocation.coordinate.long : 0.0;
            
            
         if (lat != null && lon != null && lat instanceof Number && lon instanceof Number) {
      
          
   
             ctx._source.nerlocation.coordinate = [
              "lat": lat,
              "lon": lon
            ];
  
       
                if (nerlocation.containsKey('name')) {
                  ctx._source.nerlocation.name = nerlocation.name;
                }
                    
          }
          
          
    
      
        }
        
    
        }
      }
      
      
          
      if (ctx._source.containsKey('ner_stats') &&
          ctx._source.ner_stats.containsKey('most_mentioned_ents') &&
          ctx._source.ner_stats.most_mentioned_ents != null &&
          ctx._source.ner_stats.most_mentioned_ents.size() > 0 &&
          ctx._source.ner_stats.most_mentioned_ents.containsKey('name') &&
          ctx._source.ner_stats.most_mentioned_ents.containsKey('count')) {
          
          ctx._source.annotation_types.count = ctx._source.ner_stats.most_mentioned_ents.count;
          ctx._source.annotation_types.name = ctx._source.ner_stats.most_mentioned_ents.name;
          
         
      } 
      
    """
  }
}



PUT /tika
{
  "mappings": {
    "properties": {
      "FileName": { "type": "text" },
      "BucketName": { "type": "text" },
      "EventTime": { "type": "text" },
      "size": { "type": "integer" },
      "Content": { "type": "text" },
      "Application-Name": { "type": "keyword" },
      "Application-Version": { "type": "keyword" },
      "Creation-Date": { "type": "date"},
      "Content-Type": { "type": "keyword" },
      "Content-Encoding": { "type": "text" },
      "X-TIKA:origResourceName": { "type": "text" },
      "creator": { "type": "keyword" },
      "date": { "type": "date"},
      "modified": { "type": "date" },
      "Keywords": { "type": "keyword" },
      "dc:title": { "type": "text" },
      "dc:subject": { "type": "text" },
      "Exif IFD0:Make": { "type": "keyword" },
      "Exif IFD0:Model": { "type": "keyword" },
      "geo:lat": { "type": "float" },
      "geo:long": { "type": "float" },
      "xmpDM:duration": { "type": "float" },
      "Message-From": { "type": "keyword" },
      "Message-To": { "type": "keyword" },
      "Message:From-Email": { "type": "keyword" },
      "Message:Raw-Header": { "type": "text" },
      "Message:Raw-Header:FCC": { "type": "text" },
      "Content-Disposition": { "type": "text" },
      "machine:machineType": { "type": "keyword" },
      "machine:platform": { "type": "keyword" }
    }
  }
}

GET news_reindex


PUT /news_reindex
{
  "mappings": {
    "dynamic": "false",
    "_source": {
      "includes": [
        "authors",
        "category",
        "content.main_content",
        "content.summary",
        "content.title",
        "country",
        "create_date",
        "docsentiment.negative",
        "docsentiment.neutral",
        "docsentiment.positive",
        "grade",
        "annotation_types.count",
        "annotation_types.name",
        "annotation_types.name_fa",
        "annotation_types.wiki_id",
        "annotation_types.entity_type",
        "ner_done",
        "nerlocation.coordinate",
        "nerlocation.name",
        "orientation",
        "site",
        "summary",
        "summary_fa",
        "title",
        "title_fa"
      ]
    },
    "properties": {
      "authors": {
        "type": "keyword"
      },
      "category": {
        "type": "keyword"
      },
      "content": {
        "properties": {
          "main_content": {
            "type": "text"
          },
          "main_pic": {
            "type": "keyword",
            "index": false
          },
          "summary": {
            "type": "text"
          },
          "title": {
            "type": "text"
          }
        }
      },
      "country": {
        "type": "keyword"
      },
      "create_date": {
        "type": "date"
      },
      "docsentiment": {
        "properties": {
          "negative": {
            "type": "float"
          },
          "neutral": {
            "type": "float"
          },
          "positive": {
            "type": "float"
          }
        }
      },
      "grade": {
        "type": "byte"
      },
      "annotation_types": {
        "properties": {
          "count": {
            "type": "byte" ,
            "index": false
          },
          "name": {
            "type": "keyword"
          },
          "name_fa": {
            "type": "keyword"
          },
          "wiki_id": {
            "type": "keyword"
          },
          "entity_type": {
            "type": "keyword"
          }
        }
      },
      "ner_done": {
        "type": "boolean"
      },
      "nerlocation": {
        "properties": {
          "coordinate": {
            "type": "geo_point"
          },
          "name": {
            "type": "keyword"
          }
        }
      },
      "orientation": {
        "type": "keyword"
      },
      "site": {
        "type": "keyword"
      },
      "summary": {
        "type": "text"
      },
      "summary_fa": {
        "type": "text"
      },
      "title": {
        "type": "text"
      },
      "title_fa": {
        "type": "text"
      },
      "url": {
        "type": "keyword",
        "index": false
      }
    }
  }
}


GET /news

GET general_mobin/_search
{

 "size": 10,
      "query": {
        "bool": {
          "should": [
            {
              "match": {
                
                "description": {"query": "Arabia persian","operator": "and","fuzziness": 2}
              }
            },
            
           
            
                            {
              "match": {
                
                "fa_description": {"query": "Arabia persian","operator": "and","fuzziness": 2}
              }
            }
            
            ,
            
                        
                            {
              "match": {
                
                "name": {"query": "Arabia persian","operator": "and","fuzziness": 2}
              }
            }
            
            ,
            
           
           
           
            
                            {
              "match": {
                
                "fa_name": {"query": "Arabia","operator": "and","fuzziness": 2}
              }
            }
            
            
            ]
        }
      }
}

GET news


GET news/_search
{"size": 0,
  "aggs": {
    "published_by": {
      "terms": {
        "field": "category"
       
      }
    
      
      
    }
   
  }
}


GET /news


GET /news/_search
{
  "query": {
    "bool": {
      "should": [
        {
          "wildcard": {
            "category": {
              "value": "*tekh*"
            }
          }
        },
        {
          "fuzzy": {
            "category": {
              "value": "tekh",
              "fuzziness": "AUTO"
            }
          }
        }
      ]
    }
  },
  "size": 0,
  "aggs": {
    "aggs_result": {
      "terms": {
        "field": "category"
      }
    }
  }
}



GET /nvds/_search
{
  "size": 5,
  "query": {
    "bool": {
      "must": [
        {
          "match": {
            "nvd.impact.baseMetricV3.cvssV3.attackVector.keyword": "ADJACENT_NETWORK"
          }
        }
        
        ,
        
        
        
        
        
        {
          "bool": {
            "should": [
              {
                "match": {
                  "nvd.impact.baseMetricV3.cvssV3.attackVector.keyword": "ADJACENT_NETWORK"
                }
              }
            ]
          }
        }
      ]
    }
  }
}




















DELETE test16

PUT test16

GET test111/_search



GET /test16/_doc/1702123835777335



GET feeds/_doc/1704001470141345


GET feeds/_search
{
  "query": {
    "match_all": {}  
  },
  "sort": [
    {
      "@timestamp": {
        "order": "asc"  
      }
    }
  ],
  "size": 100
}


POST feeds/_search
{
  "query": {
    "bool": {
      "filter": {
        "script": {
          "script": {
            "source": "doc['common_key.keyword'].value.length() > 60", 
            "lang": "painless"
          }
        }
      }
    }
  }
}
GET /feeds/_search
{"size": 10,

  "query": {
    "match": {
      "common_key.keyword": "01fc95a325487be0119d93ae46f3e73ec2da582c486648058df3b15e93fd2d0d"
    }
  }
  
}


GET /nvds/_search
{"size": 20,
  "sort": [
    {
      "nvd.impact.baseMetricV3.cvssV3.baseScore": {
        "order": "desc"
      }
    }
  ], 
  "query": {
    "terms": {
      "nvd.impact.baseMetricV3.cvssV3.attackVector.keyword": [
      "ADJACENT_NETWORK", "LOCAL"
      ]
    }  }
  
}

GET /news/_search
{"size": 0,

  "query": {
    "match": {
      "category.keyword": "tech"
    }
  },
     "aggs": {
        "aggs1": {
            "terms": {
                "field": "Category.keyword",
             
                "size": 1000000  
            },
            "aggs": {
                "bucket_sort": {
                    "bucket_sort": {                                 
                        "sort": [{
                            "_key": {
                                "order": "asc"
                            }
                        }],
               
                        "from": 0,
                        "size": 10
                    }
                }
            }
        }
    }
}






PUT mobin
{
  "mappings": {
    "dynamic": "false",
    "properties": {
      "description": {
        "type": "text"
      },
      "fa_description": {
        "type": "text"
      },
      
      "fa_name": {
        "type": "text"
      },
      
      "name": {
        "type": "text"
      },
      
      "status": {
        "type": "keyword"
      },
      
      "target_location": {
        "type": "keyword"
      },
      
      "tlp": {
        "type": "keyword"
      },
      
      
      "labels": {
        "type": "keyword"
      },
            
            
            
      "published_by": {
        "type": "keyword"
      },
        
      "published": {
        "type": "date"
      }
 
    }
  }
}






GET general_mobin/_search?_source
{"size": 0,
  "aggs": {
    "published_by": {
      "terms": {
        "field": "target_location.keyword"
       
      },
      
      "aggs": {
        "agg2": {
          "terms": {
            "field": "fa_target_location.keyword"
          }
        }
        
      }
      
    
      
      
    }
   
  }
}


POST test11/_delete_by_query
{
  "query": {
          "exists": {
            "field": "hash"
          }
  }
}

GET feeds/_mapping


GET /feeds/_search
{
  "query": {
    "range": {
      "docsentiment.positive": {
        "gte": 0.9,
        "lte": 1
      }
    }
  }
}



GET test/_search
		{
		  "query": {
		  "term": {
			"idd.keyword": "test5"
		  }
		}
		
		
		}
		
		
		

GET news/_search
{
  "query": {
    "range": {
      "create_date": {
                  "gte": 1704866366000,
              "lte": 1912051171000
      }
    }
  }
}




GET news/_search
{
  "query": {
    "range": {
      "create_date": {
        "gte": "now-2M/M",
        "lte": "now-1M/M"
      }
    }
  }
}

PUT test7/
{
  "mappings": {
     "dynamic": "strict",
    "properties": {
      "description":    { "type": "text" },  
      "hash":  { "type": "keyword"  }, 
         "news_agancy":  { "type": "keyword"  }, 
      
      "link":  { "type": "keyword",
          "index": false  },      
        "create_time" : {
          "type" : "date"
        },
         "text":   { "type": "text"  } ,
      "title":   { "type": "text"  }     
    }
  }
}

GET news/_doc/e640759d1b802e7468994fb43f890a9e3913e47e5e45e4911a296c69fe47c38d


GET news









POST test/_update_by_query
{
		"script": {
		  "source": "ctx._source.name = params.name;ctx._source.com = params.com;ctx._source.idd = params.idd",
		  "lang":   "painless",
		  "params": {
			"name": "qqqqqqqqqqqqqqqqqqqq",
			"idd": "qqqqqqqqqqqqqqqqqq",
			
			"com": 1111111111111
		  }
		},
		
		"query": {
		  "term": {
			"_id": "sss"
		  }
		}
}



POST feeds/_update_by_query
{
    "script": {
        "source": "ctx._source.docsentiment = params.docsentiment",
        "lang": "painless",
        "params": {
            "docsentiment": {
                "negative": 0.08,
                "neutral": 0.75,
                "positive": 0.17
            }
        }
    },
    "query": {
        "term": {
            "common_key.keyword": "b9e0c6de-9613-404d-9377-bdc43175d3b1"
        }
    }
}


PUT test14/_mapping
{
    "properties": {
        "authors" : {
          "type" : "keyword"
        }
    }
  
}

GET test2/_search

GET test1


POST /test1/_doc/tt
{
  "field1": "tt"
}


PUT nvds/_settings
{
  "index": {
    "default_pipeline": "append-abcd"
  }
}

POST _reindex
{
  "source": {
    "index": "test1"
  },
  "dest": {
    "index": "test2",
    "pipeline": "append-abcd" 
  }
}

POST test1/_doc
{
  "field1": "333333"
}


PUT _ingest/pipeline/append-abcd
{
  "description": "Append 'abcd' to field1 and fill field2",
  "processors": [
    {
      "script": {
        "source": """
          ctx.field2 = ctx.field1 + 'abcd';
        """,
        "lang": "painless"
      }
    }
  ]
}



POST _bulk
{"update":{"_index":"news","_id":"9549692561f6515a68940683fd8b29a054f466fc382f8695723f90350e1a389c"}}
{"doc_as_upsert":true,"doc":{"content":{"main_pic":"https://news.cgtn.com/news/2024-02-07/CGTN-The-Vibe-20240207-1r0fos52rhS/img/c4935283842e42d4b2c9b1642b66061f/c4935283842e42d4b2c9b1642b66061f-1280.png","summary":"【Rising Dragons, Leaping Tigers】Spring Festival symphony… performers in various world cities get in the groove for Year of the Dragon.  【Guzheng Artist】Hither and zither… guzheng maestro believes much more lies in store for ancient stringed instrument.【Virtual","main_content":"Open in CGTN APP for better experience","title":"CGTN The Vibe 20240207"},"url":"https://news.cgtn.com/news/2024-02-07/CGTN-The-Vibe-20240207-1r0fos52rhS/p.html","site":"CGTN","time":1707832939.232168,"create_date":1707264000.000001}}

DELETE your_index
GET your_index/_count


POST feeds/_update_by_query
{"script": {
      "source":"ctx._source.summary_fa = params.summary_fa;ctx._source.doc_sentiment = params.doc_sentiment",
      "lang": "painless",
      "params": {"summary_fa":"not ffff","doc_sentiment":{"negative":0.26,"neutral":0.7,"positive":0.05}}
    },
    "query": {
      "term": {
            "common_key.keyword": "51c04a79-83a1-49d4-88eb-3cfc16cb0287"
      }
    }
    }



POST /nvds/_search
{
  "size": 0,
  "aggs": {
    "substring_agg": {
      "terms": {
        "script": {
          "source": """
            def substrings = [];
            for (node in params._source.nvd.configurations.nodes) {
              for (cpe_match in node.cpe_match) {
                if (cpe_match.cpe23Uri != null && cpe_match.cpe23Uri != '') {
                  def parts = cpe_match.cpe23Uri.split(':');
                  if (parts.size() >= 4) {  // Ensure there are at least 4 parts
                    substrings.addAll(parts.subList(1, 4)); // Extract parts [1], [2], [3]
                  }
                }
              }
            }
            return substrings;
          """,
          "lang": "painless"
        }
      }
    }
  }
}

POST /nvds/_search
{
  "size": 0,
  "aggs": {
    "substring_agg": {
      "terms": {
        "script": {
          "source": """
            def substrings = [];
            for (node in params._source.nvd.configurations.nodes) {
              if (node.cpe_match != null) {
                for (cpe_match in node.cpe_match) {
                  if (cpe_match != null && cpe_match.cpe23Uri != null && cpe_match.cpe23Uri != '') {
                    def startIdx = cpe_match.cpe23Uri.indexOf(':') + 1; // Find the first colon
                    def endIdx = cpe_match.cpe23Uri.indexOf(':', startIdx); // Find the second colon
                    if (startIdx != -1 && endIdx != -1) { // Check if both colons are found
                      substrings.add(cpe_match.cpe23Uri.substring(startIdx, endIdx)); // Extract substring between first and second colon
                    }
                  }
                }
              }
            }
            return substrings;
          """,
          "lang": "painless"
        }
      }
    }
  }
}




POST /nvds/_search
{
  "size": 0,
  "aggs": {
    "vendor": {
      "terms": {
        "field": "nvd.configurations.nodes.cpe_match.vendor.keyword",
        "include": "tec.*",  
        "size": 10 
      }
    }
  }
}
GET test3/_search


PUT /_ingest/pipeline/lang
{
  "description": "A pipeline to automatically fill a field",
  "processors": [
    {
      "set": {
        "field": "lang",
        "value": "faa"
      }
    }
  ]
}




PUT _ingest/pipeline/cpe2
{
  "description": "Split cpe23Uri by ':' and extract required substrings with null check",
  "processors": [
    {
      "script": {
        "source": """
         
            def tokenizer = new java.util.StringTokenizer(ctx['cpe23Uri'].toString(), ":");
            def parts = [];
            while (tokenizer.hasMoreTokens()) {
              parts.add(tokenizer.nextToken());
            }
            ctx.part = parts.size() > 2 ? parts[2] : '';
            ctx.vendor = parts.size() > 3 ? parts[3] : '';
            ctx.product = parts.size() > 4 ? parts[4] : '';
            ctx.version = parts.size() > 5 ? parts[5] : '';
            
             ctx.vendor_product = ctx.vendor + ':' +ctx.product;
          
        """,
        "lang": "painless"
      }
    }
  ]
}





PUT _ingest/pipeline/split-cpe-uri
{
  "description": "Split nvd.configurations.nodes.cpe_match.cpe23Uri by ':' and extract required substrings with null check",
  "processors": [
    {
      "script": {
        "source": """
          if (ctx.containsKey('nvd') && ctx['nvd'] != null && ctx['nvd'].containsKey('configurations') && ctx['nvd']['configurations'] != null && ctx['nvd']['configurations'].containsKey('nodes') && ctx['nvd']['configurations']['nodes'] != null && ctx['nvd']['configurations']['nodes'].containsKey('cpe_match')) {
            def cpeMatch = ctx['nvd']['configurations']['nodes']['cpe_match'];
            if (cpeMatch instanceof List) {
              for (def match : cpeMatch) {
                if (match.containsKey('cpe23Uri') && match['cpe23Uri'] != null && match['cpe23Uri'] instanceof String) {
                  def tokenizer = new java.util.StringTokenizer(match['cpe23Uri'].toString(), ":");
                  def parts = [];
                  while (tokenizer.hasMoreTokens()) {
                    parts.add(tokenizer.nextToken());
                  }
                  match['vendor'] = parts.size() > 3 ? parts[3] : '';
                  match['product'] = parts.size() > 4 ? parts[4] : '';
                  match['version'] = parts.size() > 5 ? parts[5] : '';
                }
              }
            }
          }
        """,
        "lang": "painless"
      }
    }
  ]
}



PUT /news
{

   "mappings" : {
      "dynamic" : "false",
      "properties" : {
        "authors" : {
          "type" : "keyword"
        },
        "category" : {
          "type" : "keyword"
        },
        "create_date" : {
          "type" : "date"
        },
        "docsentiment" : {
          "properties" : {
            "negative" : {
              "type" : "float"
            },
            "neutral" : {
              "type" : "float"
            },
            "positive" : {
              "type" : "float"
            }
          }
        },
        "content" : {
        "properties" : {
            "main_content" : {
              "type" : "text"
            },
            "summary" : {
              "type" : "text"
            },
            "title" : {
              "type" : "text"
            },
                    "main_pic" : {
          "type" : "keyword",
          "index" : false
        }
          }
        },
        "most_mentioned_ents" : {
          "properties" : {
            "count" : {
              "type" : "byte"
            },
            "name" : {
              "type" : "keyword"
            }
          }
        },
        "ner_done" : {
          "type" : "boolean"
        },
        
        "nerlocation" : {
          "properties" : {
            "coordinate" : {
              
        "type": "geo_point"
     
            },
            "name" : {
              "type" : "keyword"
            }
          }
        },
        "site" : {
          "type" : "keyword"
        },
        "summary" : {
          "type" : "text"
        },
        "summary_fa" : {
          "type" : "text"
        },

        "title" : {
          "type" : "text"
        },
        "title_fa" : {
          "type" : "text"
        },
        "url" : {
          "type" : "keyword",
          "index" : false
        }
      }
    }
  }


GET test3/_doc/25


POST /test3/_doc/25?pipeline=nvds-slpit-cpe
  { "nvd": {
          "configurations": {
        "CVE_data_version": "4.0",
        "nodes": [
          {
            "cpe_match": [
              {
                "vulnerable": true,
                "cpe23Uri": "cpe:2.3:o:linux:ubunutu:*:*:*:*:*:*:*:*",
                "versionEndExcluding": "5.12",
                "cpe_name": [],
                "versionStartIncluding": "5.10"
              }
            ],
            "children": [],
            "operator": "OR"
          },
          {
            "cpe_match": [],
            "children": [
              {
                "cpe_match": [
                  {
                    "vulnerable": true,
                    "cpe23Uri": "cpe:2.3:o:netapp:h410c_firmware:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              },
              {
                "cpe_match": [
                  {
                    "vulnerable": false,
                    "cpe23Uri": "cpe:2.3:h:netapp:h410c:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              }
            ],
            "operator": "AND"
          },
          {
            "cpe_match": [],
            "children": [
              {
                "cpe_match": [
                  {
                    "vulnerable": true,
                    "cpe23Uri": "cpe:2.3:o:netapp:h300s_firmware:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              },
              {
                "cpe_match": [
                  {
                    "vulnerable": false,
                    "cpe23Uri": "cpe:2.3:h:netapp:h300s:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              }
            ],
            "operator": "AND"
          },
          {
            "cpe_match": [],
            "children": [
              {
                "cpe_match": [
                  {
                    "vulnerable": true,
                    "cpe23Uri": "cpe:2.3:o:netapp:h500s_firmware:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              },
              {
                "cpe_match": [
                  {
                    "vulnerable": false,
                    "cpe23Uri": "cpe:2.3:h:netapp:h500s:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              }
            ],
            "operator": "AND"
          },
          {
            "cpe_match": [],
            "children": [
              {
                "cpe_match": [
                  {
                    "vulnerable": true,
                    "cpe23Uri": "cpe:2.3:o:netapp:h700s_firmware:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              },
              {
                "cpe_match": [
                  {
                    "vulnerable": false,
                    "cpe23Uri": "cpe:2.3:h:netapp:h700s:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              }
            ],
            "operator": "AND"
          },
          {
            "cpe_match": [],
            "children": [
              {
                "cpe_match": [
                  {
                    "vulnerable": true,
                    "cpe23Uri": "cpe:2.3:o:netapp:h410s_firmware:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              },
              {
                "cpe_match": [
                  {
                    "vulnerable": false,
                    "cpe23Uri": "cpe:2.3:h:netapp:h410s:-:*:*:*:*:*:*:*",
                    "cpe_name": []
                  }
                ],
                "children": [],
                "operator": "OR"
              }
            ],
            "operator": "AND"
          }
        ]
      }
    }
  }

POST /nvds/_update_by_query?pipeline=nvds-slpit-cpe
{
  "query": {
    "match_all": {} 
  }
}

PUT _ingest/pipeline/nvds-slpit-cpe
{
  "description": "Split nvd.configurations.nodes.cpe_match.cpe23Uri by ':' and extract required substrings with null check",
  "processors": [
    {
      "script": {
        "source": """
          if (ctx.containsKey('nvd') && ctx['nvd'] != null && ctx['nvd'].containsKey('configurations') && ctx['nvd']['configurations'] != null && ctx['nvd']['configurations'].containsKey('nodes') && ctx['nvd']['configurations']['nodes'] != null) {
            def nodes = ctx['nvd']['configurations']['nodes'];
            for (def node : nodes) {
              if (node.containsKey('cpe_match')) {
                def cpeMatch = node['cpe_match'];
                if (cpeMatch instanceof List) {
                  for (def match : cpeMatch) {
                    if (match.containsKey('cpe23Uri') && match['cpe23Uri'] != null && match['cpe23Uri'] instanceof String) {
                      def tokenizer = new java.util.StringTokenizer(match['cpe23Uri'].toString(), ":");
                      def parts = [];
                      while (tokenizer.hasMoreTokens()) {
                        parts.add(tokenizer.nextToken());
                      }
                      match['part'] = parts.size() > 2 ? parts[2] : '';
                      match['vendor'] = parts.size() > 3 ? parts[3] : '';
                      match['product'] = parts.size() > 4 ?match['vendor']+ ':' + parts[4] : '';
                      match['version'] = parts.size() > 5 ? parts[5] : '';
                     
                    }
                  }
                }
              }
            }
          }
        """,
        "lang": "painless"
      }
    }
  ]
}


PUT /_snapshot/my_backup
{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/data/"
  }
}


GET /test3/_doc/14

POST /test3/_doc/14?pipeline=cpe
{
  "cpe23Uri": "cpe:2.3:a:telepad-app:telepad:*:*:*:*:*:*:*:*"
}

PUT _ingest/pipeline/cpe
{
  "description": "Split cpe23Uri by ':' and extract required substrings with null check",
  "processors": [
    {
      "script": {
        "source": """
          if (ctx.containsKey('cpe23Uri') && ctx['cpe23Uri'] != null && ctx['cpe23Uri'] instanceof String) {
            def tokenizer = new java.util.StringTokenizer(ctx['cpe23Uri'].toString(), ":");
            def parts = [];
            while (tokenizer.hasMoreTokens()) {
              parts.add(tokenizer.nextToken());
            }
            ctx.part = parts.size() > 2 ? parts[2] : '';
            ctx.vendor = parts.size() > 3 ? parts[3] : '';
            ctx.product = parts.size() > 4 ? parts[4] : '';
            ctx.version = parts.size() > 5 ? parts[5] : '';
            
             ctx.vendor_product = ctx.vendor + ':' +ctx.product;
          }
        """,
        "lang": "painless"
      }
    }
  ]
}


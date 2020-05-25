# ElasticSearch Cheatsheet
## CURL to Elastic Cloud
`curl -H "Content-Type: application/x-ndjson" -XPOST -u username:password https://elastic-cloud-endpoint.com:9243/products/_bulk --data-binary "@products-bulk.json"`
 
## Cluster & Shard Info
`GET _cluster/health`
`GET _cat/nodes?v`
`GET _cat/indices?v`
`GET _cat/shards?v`

## Indices
`PUT /index_name
{
  "settings": {
		"number_of_shards: #,
		"number_of_replicas: #
   }
}`
`DELETE /index_name`

## Documents
`GET /index_name/_doc/id`

###### Create
`POST /index_name/_doc
{
  "field_name": value
}`

**Create with ID/Replace:**
`PUT /index_name/_doc/id
{
  "field_name": value
}`

###### Update
`POST /index_name/_update/id
{
	"doc": {
		"field_name": value
  }
}`

`POST /index_name/_update/id
{
	"script" {
		"source": """
			if (ctx._source.field_name > 0) {
				ctx._source.field_name(++|--|=|op) params.param_field_name"
			}
			""",
		"params": {
			"param_field_name": #
		}
  }
}`

`POST /index_name/_update_by_query												#will reindex all matches
{
	"script": {
		"source": "ctx._source.in_stock(op)"
	},
  "query": {
		"match_all": {}
  }
}`

###### Upsert (Conditional Update)
`POST /index_name/_update/id
{
	"script" {
		"source": "ctx._source.field_name(++|--|=|op)",
		"upsert": {
			"field_name": value
		}
  }
}`

###### Delete
`DELETE /index_name/_doc/id`
`POST /index_name/_delete_by_query
{
  "query": {
		"match_all": {}
	}
}`

## Bulk API
`POST /index_name//_bulk
{ "index: {"_id": # } }
{ "field_name: value, "field2_name": value2 }
{ "create": {"_id":# } }
{ "field_name: value, "field2_name": value2 }
{ "update": {"_id":# } }
{ "doc": {field_name: value, "field2_name": value2 } }
{ "delete": {"_id":# } }
`
**NOTE:** Bulk file must end with newline char (\n or \r\n)

## Analyze API
`POST /_analyze 
{
  "text": "some_string",
  "analyzer": "(standard|stemming_analyzer|pattern_analyzer|english)",
  "char_filter": [html_strip],
  "tokenizer": "standard"
  "filter": [
		"lowercase".
		"stop",
		"asciifolding"																			#Translates weird chars
}`

## Mappings
`GET /index_name/_mapping`

`PUT /index_name 
{
	"settings": {
		"index.mapping.coerce": false"
  },
  "mappings": {
		"properties": {
			"field_name": { "type": "integer|float|text|date|boolean|keyword},
			"field_name": {
				"properties": {																	#for object data type
					"field_name": { "type": "date" }
					"format": "dd/MM/yyyy|epoch_second",
					"doc_values": false,													#save space when not aggregating, scripting or sorting large indices
					"norms": false, 															#save space when not using for relevance scoring (e.g. filtering/aggregations)
					"indexing": false,														#save space non-filtering e.g. time series
					"null_value": "NULL",													#set null val for searching																			
					"copy_to": concatenation_field_name,
					"ignore_above": #															#ignores > length
				} 
			},
			"field_name": { "type": "nested"},								#also for object data type
			"field_name.subfield" { "type": "some_type" }     #dot notation
		}
  }
}

###### Index Templates
Will apply to all new Indexes created matching the pattern
`PUT /_template/index_name
{
	"index_patterns": ["indexes_to_match*"],
	"mappings": {
		"properties": {
			"field_name": {
				"type": type
			}
		}
	}
}`

###### Dynamic Templates
Will apply to multiple fields/types
`PUT /_template/index_name
{
	"mappings": {
		"dynamic_templates": {
			"field_name": {
				"match_mapping_type": (type|*),
				"match": "regex",
				"match_pattern": "regex",
				"mapping": {
					"type": (desired_type|[dynamic_type])          #dynamic_type will leave type as is
				}
			}
		}
	}
}`


## Re-Indexing
`POST /_reindex
{
	"source":{
		"index": "old_index_name",
		"query": {
			"range": {
			}
		}
	},
	"dest": {
		"index": "new_index_name"
  }
}`

## Field Aliasing
`PUT /index_name/_mapping
{
	"properties": {
		"field_name": {
			"type": "alias",
			"path": "alias_name"
		}
	}
}`

## Querying
`GET /index_name/_search?q=field:value AND field2:value2``

`GET /index_name/_search?size=#
{
	"query": {
		"_source": ["field_name", "field_name"],		#Specify result fields 
		"term|match|match_phrase": {         				#Term  = exact match in index,
																							  #Match = preprocess using the analyzer befo    re searching in index
			"field_name": {
				"value": "value",
				"slop": edit_distance,   								#Define proximity search
				"fuzziness": auto|edit_distance					#Max 2, calculated on a per-term basis	
			}
		},
		"sort": [
			{ "field_name": "desc|asc" }
		]
  }
}`

`GET /index_name/_search
{
  "query": {
    "query_string" : {
      "query" : "term AND term OR term",
      "default_field" : "field_name"
    }
  }
}`

`GET /index_name/_search
{
	"query": {
		"multi_match": {
			"query":"value",
			"fields": ["field_name", "field2_name"]
		}
	}
}`

**Ranges**
`GET /index_name/_search
{
	"query": {
		"range": {										
			"field_name": {
				"gt|gte": "upper_bound_value||-#y-#M-#d",  #|| defines relative date anchor
				"lt|lte": "lower_bound_value",
				"format": "dd-MM-yyyy"										 #Optional - specify a date format
			}
		}
  }
}`

**Prefixes**
`GET /index_name/_search
{
	"query": {
		"prefix":
			"field_name.keyword": "value"
		}
  }
}`

**Wildcard & Regex**
`GET /index_name/_search
{
	"query": {
		"wildcard|regexp":
			"field_name": "match_pattern*?|regex_pattern"
		}
  }
}`

###### Compound Queries
`GET /index_name/_search
{
	"query": {
		"bool": {
			"must|must_not|should": { [
				{
					"match":{
						"field_name": {
						"query": "value",
						"_name": "name_for_pattern"						#optional param to show if matched in results
					}
				}
			],
			"filter": [
				{
					"range": {
						"field_name": {
							"lte|gte|gt|lt": value
						}
					}
				}
			] }
		}
	}
}

`

###### Notes
**Remember:** Inverse Document Frequency is calculated on a per-shard basis by default for relevance scoring


###### Debugging Queries
`GET /index_name/_explain
{
	"query": {
		"term|match": {
			"field_name": value
		}
	}
}`

## Aggregations
`GET /index_name/_search
{
  "size": {
    "aggs" : {
      "name_for_aggregation": {
				"sum|avg|min|max|cardinalit|value_count|stats": {
					"field": "field_to_aggregate"
				}
			}
    }
  }
}`

###### Buckets
`GET /index_name/_search
{
  "size": {
    "aggs" : {
      "name_for_buckets": {
				"filters: {
					"name_for_bucket": {
						"match|missing": {
							"field": "value"
						}
					}
        }
			}
    }
  }
}`

**Range Buckets**
`GET /index_name/_search
{
  "size": {
    "aggs" : {
      "name_for_buckets": {
				"range|date_range|histogram": {
					"field": "field_name",
					"ranges": [
					{
						"to: value
					},
					{
						from": value,
						"to": value
					},
					{
					  "from": value
					}
					]
        }
			}
    }
  }
}`

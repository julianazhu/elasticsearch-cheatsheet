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

`POST /index_name/_update_by_query
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


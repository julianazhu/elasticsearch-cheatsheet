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
`POST /_bulk
{ "index: {"_index": "index_name", "_id": #} }
{ "field_name: value, "field2_name": value2 }
`

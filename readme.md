# Integration tests for the expert scoring script example plugin
#
---
install:
  - do: bin/elasticsearch-plugin install file:///path/to/your/plugin

setup
  - do:
      indices.create:
          index:  test

  - do:
      index:
        index:  test
        id:     1
        body:   { "important_field": "foo" }
  - do:
        index:
          index:  test
          id:     2
          body:   { "important_field": "foo foo foo" }
  - do:
        index:
          index:  test
          id:     3
          body:   { "important_field": "foo foo" }

  - do:
      indices.refresh: {}
---
"document scoring":
  - do:
      search:
        rest_total_hits_as_int: true
        index: test
        body:
          query:
            function_score:
              query:
                match:
                  important_field: "foo"
              functions:
                - script_score:
                    script:
                      source: "pure_df"
                      lang: "expert_scripts"
                      params:
                        field: "important_field"
                        term: "foo"

  - length: { hits.hits: 3 }
  - match: {hits.hits.0._id: "2" }
  - match: {hits.hits.1._id: "3" }
  - match: {hits.hits.2._id: "1" }
  
  Demo:
    
    POST /_search
    {
      "query": {
        "function_score": {
          "query": {
            "match": {
              "body": "foo"
            }
          },
          "functions": [
            {
              "script_score": {
                "script": {
                    "source": "pure_df",
                    "lang" : "expert_scripts",
                    "params": {
                        "field": "body",
                        "term": "foo"
                    }
                }
              }
            }
          ]
        }
      }
    }
get /_cat/indices

get /_template/tosca-template

get /tosca-g001-2020-05/_mapping

# Update mapping
put /tosca-g001-*/_mapping
{
  "properties": {
    "Folder": {
      "type": "text",
      "fields": {
        "raw": {
          "type": "keyword",
          "ignore_above": 2048
        }
      }
    }
  }
}

# Re-index
POST _reindex
{
  "source": {
    "remote": {
      "host": "https://opensearch-cluster-master.g0-auth-enabled.svc.cluster.local:9200",
      "username": "admin",
      "password": "Test@Password01"
    },
    "index": "tosca-g001-2025-04"

  },
  "dest": {
    "index": "tosca-g001-2025-04"
  }
}


# UpdateFolder 
POST tosca-g001*/_update_by_query
{
  "query": {
    "bool": {
      "must": [
        {
          "exists": {
            "field": "NodePath"
          }
        }
      ],
      "should": [],
      "must_not": [
        {
          "exists": {
            "field": "Folder"
          }
        }
      ]
    }
  },
  "script": {
    "lang": "painless",
    "source": """
      def m = /^(.*)(\/.*?)$/.matcher(ctx._source.NodePath);
      
      if ( m.matches() ) { 
        ctx._source.Folder = m.group(1)
      }
    """
  }
}
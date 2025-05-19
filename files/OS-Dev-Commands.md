get /_cat/indice
get /_template/tosca-template
get /tosca-g001-2023-03/_mapping


put /tosca-g001-*/_mapping
{
  "properties": {
    "Folder": {
      "type": "text",
      "fields": {
        "raw": {
          "type": "keyword",
          "ignore_above": 2048
        }s
      }
    }
  }
}



POST tosca-g001*/_update_by_query
{
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
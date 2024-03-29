# Custom Entity Lookup skill in Azure AI Search

Applies to: AI enrichment capabilities of Azure AI Search

Azure AI Search provides AI-enriched indexing that invokes skills for analyzing, transforming, and enriching raw content to make it full-text searchable in a search index. This sample demonstrates one of the entity-related built-in skills from Microsoft, [Custom Entity Lookup](https://docs.microsoft.com/azure/search/cognitive-search-skill-custom-entity-lookup).

In contrast with [Entity Recognition](https://docs.microsoft.com/azure/search/cognitive-search-skill-entity-recognition-v3), which finds whatever the model tells it find, [Custom Entity Lookup](https://docs.microsoft.com/azure/search/cognitive-search-skill-custom-entity-lookup) finds whatever YOU tell it to find. You can provide the skill with a simple list of entities, or you can further define those entities by type and subtype. You can also assign aliases for variations of an entity name. In the context of a search solution, custom entities could be useful for filters, facets, and sorting.

In this sample, you'll learn the following techniques:

+ Create a [custom entity definition](https://docs.microsoft.com/azure/search/cognitive-search-skill-custom-entity-lookup#custom-entity-definition-format) that defines the entities of interest
+ Create a skillset that includes the [Custom Entity Lookup skill](https://docs.microsoft.com/azure/search/cognitive-search-skill-custom-entity-lookup)
+ Create a search index that models the complex types of Custom Entity Lookup skill output
+ Create a search indexer that specifies the output field mappings

## Content

The sample data used for this example consists of famous speeches in PDF file format, uploaded to a blob container in Azure Storage. These files can be found at [Azure-Samples/azure-search-sample-data/famous-speeches-pdf](https://github.com/Azure-Samples/azure-search-sample-data/tree/master/famous-speeches-pdf). You can upload these files to a blob container in Azure Storage and then reference the container and connection string in your indexer data source.

The code is a Postman collection that calls the AI Search REST APIs to create and run the objects. You can import this collection and fill in its variables to create these objects on your search service.

The custom entity definition is an external JSON file with entities that match the sample data. This file is named "custom-entities-famous-speeches.json" and is located in the same folder as this readme. You can upload this file to Azure Storage and then reference its fully-qualified path name in your skillset.

## Prerequisites

+ Azure AI Search (free or billable version)
+ Azure Storage 
+ Sample data and sample custom entity definition, both uploaded to blob containers in Azure Storage
+ Postman desktop app

To authenticate to your Azure services, you'll need the API key for Azure AI Search and a full access connection string for Azure Storage. A full access connection string incudes an account key used for accessing your content. If you're unfamiliar with setting up REST calls in Postman, see [Quickstart: Create an Azure AI Search index using REST APIs](https://docs.microsoft.com/azure/search/search-get-started-rest) for help.

## Create a custom entity definition

You'll set up the enrichment pipeline with all of the usual components: source data in a supported format, a data source object that provides the connection, a skillset that specifies enrichments, an indexer that defines the execution, and an index that receives both unchanged and enriched data for full text search. The new piece in this example is the custom entity definition, which is where you provide the custom entities you'd like to recognize, including entity variants (aliases).

You can provide the custom entity definition as an external CSV or JSON file, or as an inline definition which is useful for proof-of-concept testing and small workloads. Since you're more likely to use an external file, this sample includes a JSON file with entity mappings that align to the sample data. This custom entity definition is simple. For more advanced scenarios, see [Custom entity definition format](https://docs.microsoft.com/azure/search/cognitive-search-skill-custom-entity-lookup#custom-entity-definition-format).

If you are running the sample, upload this JSON file to its own blob container in Azure Storage.

1. Identify entities of interest. Because this data set consists of famous American civic speeches, the entities we'll look for will include themes and concepts that are common to this genre.

   + equal (alias "equality")
   + founding fathers (alias "fathers")
   + freedom (alias "liberty")
   + god
   + justice
   + nation (alias "country")
   + revolution
   + rights

1. Save the following as a JSON file and then upload it to a blob container in Azure Storage. Under blob properties, get the URL to the file. In a later step, you'll specify it in the Custom Entity Lookup skill definition.

  ```
  [ 
      { 
          "name" : "equality",
          "aliases" : [ 
              { "text" : "equal", "caseSensitive" : false }
          ]
      }, 
      { 
          "name" : "founding fathers",
          "aliases" : [ 
              { "text" : "fathers", "caseSensitive" : false }
          ]
      }, 
      { 
          "name" : "freedom",
          "aliases" : [ 
              { "text" : "liberty", "caseSensitive" : false }
          ]
      },
      { 
          "name" : "god"
      }, 
      { 
          "name" : "justice"
      }, 
      { 
          "name" : "nation",
          "aliases" : [ 
              { "text" : "country", "caseSensitive" : false }
          ]
      },
      { 
          "name" : "revolution"
      }, 
      { 
          "name" : "rights"
      }
  ]
  ```

## Create a data source

The data source specifies a full access connection string to Azure Storage. The container should hold the four famous speech PDF files.

```
PUT https://<YOUR-SEARCH-SERVICE>.search.windows.net/datasources/famous-speeeches?api-version=2020-06-30
{
  "description": null,
  "type": "azureblob",
  "subtype": null,
  "credentials": {
    "connectionString": "DefaultEndpointsProtocol=https;AccountName=<YOUR-STORAGE-ACCOUNT>;AccountKey=..."
  },
  "container": {
    "name": "famous-speeches",
    "query": null
  },
  "dataChangeDetectionPolicy": null,
  "dataDeletionDetectionPolicy": null,
  "encryptionKey": null,
  "identity": null
}
```

## Create a skillset 

This skillset specifies a combination of Custom Entity Lookup skill and Entity Recognition. The skill definition includes a pointer to the blob container that stores the custom entity definition.

```
PUT https://YOUR-SEARCH-SERVICE>.search.windows.net/skillsets/famous-speeches?api-version=2020-06-30
{
   "description":"",
   "skills":[
      {
         "@odata.type":"#Microsoft.Skills.Text.CustomEntityLookupSkill",
         "name":"#1",
         "description":null,
         "context":"/document/content",
         "defaultLanguageCode":"en",
         "entitiesDefinitionUri":"https://<YOUR-STORAGE-ACCOUNT>.blob.core.windows.net/famous-speeches-entity-def/custom-entities-famous-speeches.json",
         "globalDefaultCaseSensitive":null,
         "globalDefaultAccentSensitive":null,
         "globalDefaultFuzzyEditDistance":null,
         "inputs":[
            {
               "name":"text",
               "source":"/document/content"
            }
         ],
         "outputs":[
            {
               "name":"entities",
               "targetName":"entities"
            }
         ],
         "inlineEntitiesDefinition":[
            
         ]
      },
      {
         "@odata.type":"#Microsoft.Skills.Text.V3.EntityRecognitionSkill",
         "name":"#2",
         "description":null,
         "context":"/document/content",
         "categories":[
            "Product",
            "PhoneNumber",
            "Person",
            "Quantity",
            "Organization",
            "IPAddress",
            "URL",
            "Email",
            "Event",
            "Skill",
            "Location",
            "PersonType",
            "Address",
            "DateTime"
         ],
         "defaultLanguageCode":"en",
         "minimumPrecision":null,
         "modelVersion":null,
         "inputs":[
            {
               "name":"text",
               "source":"/document/content"
            }
         ],
         "outputs":[
            {
               "name":"persons",
               "targetName":"people"
            },
            {
               "name":"organizations",
               "targetName":"organizations"
            },
            {
               "name":"locations",
               "targetName":"locations"
            }
         ]
      }
   ],
   "cognitiveServices":null,
   "knowledgeStore":null,
   "encryptionKey":null
}
```

## Create an index

This index includes blob metadata, fields for entity recognition output, and fields for custom entity lookup.

```
PUT https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/famous-speeches?api-version=2020-06-30
{
  "fields": [
    {
      "name": "content",
      "type": "Edm.String",
      "facetable": false,
      "filterable": false,
      "retrievable": true,
      "searchable": true,
      "sortable": false,
    },
    {
      "name": "metadata_storage_name",
      "type": "Edm.String",
      "facetable": false,
      "filterable": false,
      "retrievable": false,
      "searchable": false,
      "sortable": false,
    },
    {
      "name": "metadata_storage_path",
      "type": "Edm.String",
      "facetable": false,
      "filterable": false,
      "key": true,
      "retrievable": true,
      "searchable": false,
      "sortable": false,
    },
    {
      "name": "people",
      "type": "Collection(Edm.String)",
      "facetable": false,
      "filterable": false,
      "retrievable": true,
      "searchable": true,
    },
    {
      "name": "organizations",
      "type": "Collection(Edm.String)",
      "facetable": false,
      "filterable": false,
      "retrievable": true,
      "searchable": true,
    },
    {
      "name": "locations",
      "type": "Collection(Edm.String)",
      "facetable": false,
      "filterable": false,
      "retrievable": true,
      "searchable": true,
    },
    {
      "name": "entities",
      "type": "Collection(Edm.ComplexType)",
      "fields": [
        {
          "name": "name",
          "type": "Edm.String",
          "facetable": false,
          "filterable": false,
          "retrievable": true,
          "searchable": true,
          "sortable": false,
        },
        {
          "name": "id",
          "type": "Edm.String",
          "facetable": false,
          "filterable": false,
          "retrievable": true,
          "searchable": false,
          "sortable": false,
        },
        {
          "name": "description",
          "type": "Edm.String",
          "facetable": false,
          "filterable": false,
          "retrievable": true,
          "searchable": true,
          "sortable": false,
        },
        {
          "name": "type",
          "type": "Edm.String",
          "facetable": true,
          "filterable": true,
          "retrievable": true,
          "searchable": false,
          "sortable": false,
        },
        {
          "name": "subtype",
          "type": "Edm.String",
          "facetable": true,
          "filterable": true,
          "retrievable": true,
          "searchable": false,
          "sortable": false,
        },
        {
          "name": "matches",
          "type": "Collection(Edm.ComplexType)",
          "fields": [
            {
              "name": "text",
              "type": "Edm.String",
              "facetable": false,
              "filterable": false,
              "retrievable": true,
              "searchable": true,
              "sortable": false,
            },
            {
              "name": "offset",
              "type": "Edm.Int32",
              "facetable": true,
              "filterable": true,
              "retrievable": true,
              "sortable": false,
            },
            {
              "name": "length",
              "type": "Edm.Int32",
              "facetable": true,
              "filterable": true,
              "retrievable": true,
              "sortable": false,
            },
            {
              "name": "matchDistance",
              "type": "Edm.Double",
              "facetable": true,
              "filterable": true,
              "retrievable": true,
              "sortable": false,
            }
          ]
        }
      ]
    }
  ],
  "suggesters": [],
  "scoringProfiles": [],
  "defaultScoringProfile": "",
  "corsOptions": null,
  "analyzers": [],
  "charFilters": [],
  "tokenFilters": [],
  "tokenizers": []
}
```

## Create and run the indexer

The indexer is where you configure execution settings. By default, creating the indexer also runs it on the search service. Output field mappings are used to route enriched content, such as predefined and custom entities, to fields in a search index.

```
PUT https://<YOUR-SEARCH-SERVICE>.windows.net/indexers/famous-speeches?api-version=2020-06-30
{
  "dataSourceName": "famous-speeches",
  "skillsetName": "famous-speeches",
  "targetIndexName": "famous-speeches",
  "disabled": null,
  "schedule": null,
  "parameters": {
     "batchSize": null,
     "maxFailedItems": 0,
     "maxFailedItemsPerBatch": 0,
     "base64EncodeKeys": true,
     "configuration": {
       "dataToExtract": "contentAndMetadata",
       "parsingMode": "default"
    }
  },
  "fieldMappings" : [
    {
      "sourceFieldName" : "metadata_storage_name",
      "targetFieldName" : "metadata_storage_name"
    },
    {
      "sourceFieldName" : "metadata_storage_path",
      "targetFieldName" : "metadata_storage_path"
    },
    {
      "sourceFieldName" : "content",
      "targetFieldName" : "content"
    }
  ],
"outputFieldMappings": [
    {
        "sourceFieldName": "/document/content/entities",
        "targetFieldName": "entities"
    },
    {
      "sourceFieldName": "/document/content/people",
      "targetFieldName": "people"
    },
    {
      "sourceFieldName": "/document/content/organizations",
      "targetFieldName": "organizations"
    },
    {
      "sourceFieldName": "/document/content/locations",
      "targetFieldName": "locations"
    }
  ]
 }
```

## Check results

You can switch to the Azure portal and use Search Explorer to query results, or run the following GET verbs to view indexer status, followed by a basic query once indexer execution is finished to view the content. Indexer and enrichment processing takes a few minutes to complete.

This command gets indexer status:

```
GET https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexers/famous-speeches/status?api-version=2020-06-30
```

You can query the search index as soon as the first document is processed. Although you can use GET, using POST allows you to set query parameters in the body of the request, which is easier to modify and re-run.

```
POST https://<YOUR-SEARCH-SERVICE>.search.windows.net/indexes/famous-speeches/docs/search?api-version=2020-06-30
{
    "search": "*",
    "queryType": "simple",
    "select": "entities, people, organizations, locations",
    "searchFields": "",
    "filter": "",
    "orderby": "",
    "count": true
}
```

Note that the custom entity lookup skill does not aggregate or consolidate entity instances within a document. If a document has multiples of the same entity, each occurrence will be listed in the results, along with its match details, which might be more verbose than what you're expecting.

Here are the results for the Gettysburg Address, where the entity "nation" occurs five times in the document.

```
{
    "@odata.count": 4,
    "value": [
        {
            "@search.score": 1.0,
            "people": [
                "Abraham Lincoln"
            ],
            "organizations": [
                "Lit2Go"
            ],
            "locations": [
                "Gettysburg",
                "National Cemetery",
                "Pennsylvania",
                "continent",
                "battle-field"
            ],
            "entities": [
                {
                    "name": "founding fathers",
                    "id": null,
                    "description": null,
                    "type": null,
                    "subtype": null,
                    "matches": [
                        {
                            "text": "fathers",
                            "offset": 199,
                            "length": 7,
                            "matchDistance": 0.0
                        }
                    ]
                },
                {
                    "name": "nation",
                    "id": null,
                    "description": null,
                    "type": null,
                    "subtype": null,
                    "matches": [
                        {
                            "text": "nation",
                            "offset": 246,
                            "length": 6,
                            "matchDistance": 0.0
                        },
                        {
                            "text": "nation",
                            "offset": 406,
                            "length": 6,
                            "matchDistance": 0.0
                        },
                        {
                            "text": "nation",
                            "offset": 421,
                            "length": 6,
                            "matchDistance": 0.0
                        },
                        {
                            "text": "nation",
                            "offset": 649,
                            "length": 6,
                            "matchDistance": 0.0
                        },
                        {
                            "text": "nation",
                            "offset": 1494,
                            "length": 6,
                            "matchDistance": 0.0
                        }
                    ]
                },
                {
                    "name": "freedom",
                    "id": null,
                    "description": null,
                    "type": null,
                    "subtype": null,
                    "matches": [
                        {
                            "text": "Liberty",
                            "offset": 267,
                            "length": 7,
                            "matchDistance": 0.0
                        },
                        {
                            "text": "freedom",
                            "offset": 1540,
                            "length": 7,
                            "matchDistance": 0.0
                        }
                    ]
                },
                {
                    "name": "equality",
                    "id": null,
                    "description": null,
                    "type": null,
                    "subtype": null,
                    "matches": [
                        {
                            "text": "equal",
                            "offset": 335,
                            "length": 5,
                            "matchDistance": 0.0
                        }
                    ]
                },
                {
                    "name": "god",
                    "id": null,
                    "description": null,
                    "type": null,
                    "subtype": null,
                    "matches": [
                        {
                            "text": "God",
                            "offset": 1508,
                            "length": 3,
                            "matchDistance": 0.0
                        }
                    ]
                }
            ]
        },
```

## Next steps

Remember to delete any Azure services and data that you're no longer using. If you'd like to continue with other skills or features, see the [official docs](https://docs.microsoft.com/azure/search/).

# Introduction à MongoDB


## Quick start


download for local or create an atlas account for cloud:\
https://www.mongodb.com/try/download/community \
https://www.mongodb.com/fr-fr/cloud/atlas


installation
```py
pip install 'pymongo[srv]'
```

import
```py
from pymongo import MongoClient
```

connection
```py
client = MongoClient()
```

Lister les bases de données \
pour les méthodes de type list ou find mongo retourne un curseur qui peut être parcouru dans une boucle ou dérouler directement dans une liste.
```py
db_names = list(client.list_databases())
```

Lister les collections
```py
collection_names = db.list_collection_names()
for name in collection_names:
    print(name)
```

creation d'une base si elle n'existe pas
```py
db = client.disney
```

## Requêtes


insertion à partir d'un fichier ou d'une liste de documents avec insert_many:
```py
with open('movies.json') as json_file:
    movies = json.load(json_file)
db.movies.insert_many(movies)
```

insertion simple
```py
db.movies.insert_one({"title":"The Lion King", "years":1994})
```

trouver tous les films et les convertir en dataframe
```py
movies = db.movies.find()
df = pd.DataFrame(movies)
```

trouver avec des filtres\
La premier argument est ce que l'on cherche, le second argument est ce que l'on veut comme champs.\
$gt (greater than) est un opérateur, liste: https://docs.mongodb.com/manual/reference/operator/query/ 
```py
db.movies.find({"years":{"$gt":2004}}, {"title":1, "date":1})
```

limiter et trier les résultats
```py
db.movies.find({"years":{"$gt":2004}}, {"title":1, "date":1}).sort("years").limit(30)
```

update
```py
db.movies.update_one({"title": "Fantasia"}, {"$set":{"budget": 2.28}})
```

update many
```py
db.movies.update_many({"years": {"$lt":2002}},  { "$push": { "tags": "classics" } })
```

delete one
```py
db.movies.delete_one({"title": "pokemon"})
```

delete many
```py
db.movies.delete_many({"years": {"$gt":datetime.now().year}})
```

supprimmer une collection
```py
db.movies.drop()
```

supprimmer une base de données
```py
client.drop_database('disney')
```

fermer la connection
```py
client.close()
```


## Document Validation

* Pas obligatoire
* Mongosh + JSON Schema
* Soit permanent sur une collection, soit à la demande lors des requêtes.
* La validation peut ou pas s'appliquer aux documents existants lors de sa mis en place: *validationLevel*
* L'échec d'une validation peut retourner une erreur ou un avertissement: *validationAction*
* On peut esquiver les validations (mais c'est mal): *bypassDocumentValidation*

```py
# à la création
db.createCollection( <collection>, { validator: { $jsonSchema: <schema> } } )
# ensuite
db.runCommand( { collMod: <collection>, validator:{ $jsonSchema: <schema> } } )
# lors des requêtes
db.collection.find( { $jsonSchema: <schema> } )
db.collection.find( { $nor: [ { $jsonSchema: <schema> } ] } )
# voir les logs
db.adminCommand( { getLog:'global'} ).log.forEach(x => {print(x)})
```

exemple à la création :

```js
db.createCollection("pokemon", {
    validator: {
       $jsonSchema: {
          bsonType: "object",
          required: [ "name"],
          properties: {
             name: {
                bsonType: "string",
                description: "something like pikachu"
             },
             type: {
                bsonType: "array",
                enum: ["feu","glace"]
                description: "type are not required"
             },             
          }
       }
    }
 })
```





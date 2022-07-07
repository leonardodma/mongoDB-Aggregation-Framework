# MongoDB Aggregation Framework

Documentação principal rápida: [docs](https://www.mongodb.com/docs/manual/meta/aggregation-quick-reference/)

## Principais Funções no Pipeline

### $match

Faz filtragens nos dados, usando operadores [Referência](https://www.mongodb.com/docs/manual/reference/operator/aggregation/match/)

- $in:
- $nin
- $all
- $eq
- $gte
- $gt
- $lte
- $lt

```json
[
  {
    "$match": {
      "$and": [
        { "imdb.rating": { "$gte": 7.0 } },
        { "genres": { "$nin": ["Crime", "Horror"] } },
        { "rated": { "$in": ["PG", "G"] } },
        { "languages": { "$all": ["English", "Japanese"] } }
      ]
    }
  }
]
```

---

### $project

Seleciona quais campos dos dados serão utilizados, e também consegue adionar e modificar campos. [Referência](https://www.mongodb.com/docs/manual/reference/operator/aggregation/project/)

```json
[
  {
    "$project": {
      "_id": 0,
      "title": 1,
      "words_cnt": { "$size": { "$split": ["$title", " "] } }
    }
  },
  {
    "$match": { "words_cnt": { "$eq": 1 } }
  }
]
```

Declarar variáveis:

```json
{
  "$project": {
    "finalTotal": {
      "$let": {
        "vars": {
          "total": { "$add": ["$price", "$tax"] },
          "discounted": {
            "$cond": { "if": "$applyDiscount", "then": 0.9, "else": 1 }
          }
        },
        "in": { "$multiply": ["$$total", "$$discounted"] }
      }
    }
  }
}
```

Percorrer usando map:

```json
{
  "$project": {
    "adjustedGrades": {
      "$map": {
        "input": "$quizzes",
        "as": "grade",
        "in": { "$add": ["$$grade", 2] }
      }
    }
  }
}
```

Percorrer usando reduce:

```json
{
  "$reduce": {
    "input": [1, 2, 3, 4],
    "initialValue": { "sum": 5, "product": 2 },
    "in": {
      "sum": { "$add": ["$$value.sum", "$$this"] },
      "product": { "$multiply": ["$$value.product", "$$this"] }
    }
  }
}
```

---

### $addFields

É como o `$project` para fazer tranformações de dados e adicionar novos campos, porém ele não remove campos da coleção original. [Referência](https://www.mongodb.com/docs/manual/reference/operator/aggregation/addFields/)

---

### $geoNear

Operador para realizar querys geográficas. Precisa ser o primeiro item dentro do pipeline. [Referência](https://www.mongodb.com/docs/manual/reference/operator/aggregation/geoNear/)

---

### Cursor-like Stages

Utilizados no final das Querys!

`$limit`: Limita a quantidade de resultados a serem observados:

```json
{ "$limit": 5 }
```

`$skip`: Passa alguns valores a serem observados:

```json
{ "$skip": 5 }
```

`$count`: Conta quantos documentos exitem na coleção, após as alterações feitas no pipeline:

```json
{ "$count": "field" }
```

`$sort`: Ordena os valores encotrados, sendo _-1_ para `DESC` e _1_ para `ASC`:

```json
{ "$sort": { "field": 1, "field_2": -1 } }
```

---

### $group

Usado para agrupar os dados por algum fator especificado no campo _\_id_. Se não existe a necessidade de agrupar por algum fator, esse o campo _\_id_ pode ser `null`. [Referência](https://www.mongodb.com/docs/manual/reference/operator/aggregation/group/)

```json
[
  {
    "$group": {
      "_id": {
        "numDirectors": {
          "$cond": [{ "$isArray": "$directors" }, { "$size": "$directors" }, 0]
        }
      },
      "numFilms": { "$sum": 1 }
    }
  }
]
```

### $unwind

Serve para desagrupar arrays que estão em um documento, para vários documentos, cada um com um item desse array. [Referência](https://www.mongodb.com/docs/manual/reference/operator/aggregation/unwind/)

```json
[
  { "$unwind": "$cast" },
  {
    "$group": {
      "_id": "$cast",
      "numFilms": { "$sum": 1 },
      "average": { "$avg": "$imdb.rating" }
    }
  }
]
```

### $lookup

Funciona como o left join, procurando valores em outras coleções com base em chaves estrangeiras. Se não houver match, o array ficará vazio. [Referência](https://www.mongodb.com/docs/manual/reference/operator/aggregation/lookup/)

Possui a seguinte sintaxe:
```txt
{
   $lookup:
     {
       from: <collection to join>,
       localField: <field from the input documents>,
       foreignField: <field from the documents of the "from" collection>,
       as: <output array field>
     }
}
```
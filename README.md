# Awesome-Text2GQL

[![Star](https://shields.io/github/stars/tugraph-family/Awesome-Text2GQL?logo=startrek&label=Star&color=yellow)](https://github.com/TuGraph-family/Awesome-Text2GQL/stargazers)
[![Fork](https://shields.io/github/forks/tugraph-family/Awesome-Text2GQL?logo=forgejo&label=Fork&color=orange)](https://github.com/TuGraph-family/Awesome-Text2GQL/forks)
[![Contributor](https://shields.io/github/contributors/tugraph-family/Awesome-Text2GQL?logo=actigraph&label=Contributor&color=abcdef)](https://github.com/TuGraph-family/Awesome-Text2GQL/contributors)
[![Commit](https://badgen.net/github/last-commit/tugraph-family/Awesome-Text2GQL/master?icon=git&label=Commit)](https://github.com/TuGraph-family/Awesome-Text2GQL/commits/master)
[![License](https://shields.io/github/license/tugraph-family/Awesome-Text2GQL?logo=apache&label=License&color=blue)](https://www.apache.org/licenses/LICENSE-2.0.html)
[![Release](https://shields.io/github/v/release/tugraph-family/Awesome-Text2GQL.svg?logo=stackblitz&label=Version&color=red)](https://github.com/TuGraph-family/Awesome-Text2GQL/releases)

## Introduction

Awesome-Text2GQL is an AI-assisted framework for Text2GQL dataset construction. The framework supports translation, generalization and generation of Text2GQL corpus and corresponding database instance. With the assistance of this framework, users are able to construct a high quality Text2GQL dataset on any graph query language(ISO-GQL, Cypher, Gremlin, SQL/PGQ, etc.) with a significantly lower cost than pure human labor annotation.

![image](./images/overview.jpg)

### Key Features

+ **Translation**

    + [x] **Question translation** from graph query languages to questions in different natural languages to accelerate question annotation.

    + [x] **Query translation** between different graph query languages to gather corpus from all existing graph query languages.

+ **Generalization**

    + [x] **Question generalization** for getting natural language questions in different language style but with same semantic meaning to increase the diversity of corpus.

    + [x] **Query generalization** for instantiating similar query patterns on different graph schemas to increase the diversity of corpus.

+ **Generation**(Future Plan)

    + [ ] **Data generation** for generating database instance from graph schema to increase the diversity of database domain.

    + [ ] **Question generation** for generating possibly asked questions from graph schema to assist question annotation.

    + [ ] **Query generation** for generating corresponding query from natural language question to assist question annotation.

## Demo: TuGraph-DB ChatBot

TuGraph-DB ChatBot is a demo demonstrates the effect of an agent trained by the corpus generated by Awesome-Text2GQL. It can intercract with the TuGraph-DB taking how you want to operate the db in Chinese as input, such as querying or creating data. And The ChatBot will also help you to excute the cypher too.

![demo](./images/demo.gif)

Refer to [demo](./chatbot_demo/README.md) for more information.

## Quick Start

### Environment Preparation

It is recommended to use poetry to manage your python environment while other tools may also work.

```
# install poetry and poetry shell
pip install poetry
pip install poetry-plugin-shell

# install environment
poetry install

# activate virtual environment
poetry shell
```

### LLM Setup

#### Remote LLM Setup

To run Awesome-Text2GQL funtions based on remote LLMs，apply API-KEY before you start.

1. Apply API-KEY

Awesome-Text2GQL's remote LLM client is based on the Qwen Inference Service served by Aliyun, you can refer to [Aliyun](https://help.aliyun.com/zh/dashscope/developer-reference/acquisition-and-configuration-of-api-key?spm=a2c4g.11186623.0.0.4e202a9dXlz5vH#1e6311202fthe) to apply the API-KEY.

2. Set API-KEY via environment variables (recommended)

```
# replace YOUR_DASHSCOPE_API_KEY with your API-KEY
echo "export DASHSCOPE_API_KEY='YOUR_DASHSCOPE_API_KEY'" >> ~/.bashrc
source ~/.bashrc
echo $DASHSCOPE_API_KEY
```

#### Local LLM Setup
Awesome-Text2GQL's local LLM client is based on transformers library, use model id from HuggingFace model hub if you can access HuggingFace or use the related local file path where the LLM model is. Add model_path when initializing llm client if you want to use local LLM instead of remote LLM.

### Run Example

#### Cypher2GQL

`python ./examples/cypher2gql.py`

This example shows how to use AwesomeText2GQL Framework to translate neo4j's Text2Cypher corpus into Text2GQL corpus with queries aligned to ISO-GQL grammar.

#### Generalize Cypher Corpus

`python ./examples/generalize_corpus_cypher.py`

This example shows how to use AwesomeText2GQL Framework to generalize from one cypher corpus pair to construct a Text2GQL corpus dataset.

#### Generalize GQL Corpus

`python ./examples/generalize_corpus_gql.py`

This example shows how to use AwesomeText2GQL Framework to generalize from one ISO-GQL corpus pair to construct a Text2GQL corpus dataset.

#### English to Chinese

`python ./examples/english_to_chinese.py`

This example shows how to use AwesomeText2GQL Framework to translate English question into Chinese question with the same semantic meaning.

#### AST Printing

`python ./examples/print_ast.py`

This example shows how to use AwesomeText2GQL Framework to print the ast of a query. Visualizing the AST is helpful for IR and other AST related development.

## Modules

Awesome-Text2GQL use Translator, Generalizer and Generator to assit the entire process of Text2GQL dataset construction. 

### Translator

Translator supports multilingual tranlsation for question translation and multi-graph-query-language translation for query translation. Users can use translator to translate existing corpus in different natural language and graph query language into target natural language and graph query language.

#### Question Translator

Question translator currently has the ability to translate a query into a natural language question(English) with a similar query template and corresponding question template. In the future, we will support multilingual translation of natural language question.

``` python
from app.core.llm.llm_client import LlmClient
from app.core.translator.question_translator import QuestionTranslator

llm_client = LlmClient(model="qwen-plus-0723")
query_template="MATCH (n:Person)-[:HAS_CHILD*1]->(n) WHERE n.name = 'Vanessa Redgrave' RETURN n"
question_template="Who are Roy Redgrave's second generations?"
query_list = [
    "MATCH (n1:person)-[e1:acted_in]->{1,1}(n2:movie) WHERE n1.id = 'Neo' RETURN n2.`duration` AS `DURATION`",
    "MATCH (n1:person)-[e1:directed]->{1,1}(n2:movie) WHERE n1.name = 'MacQUeen' RETURN n2.id AS ID",
    "MATCH (n1:person)-[e1:produce]->{1,1}(n2:movie) WHERE n1.name = 'Hans' RETURN n2.rated AS RATED"
    ]

# translate query into question
question_translator = QuestionTranslator(llm_client=llm_client, chunk_size=5)
question_list = question_translator.translate(
    query_template=query_template,
    question_template=question_template,
    query_list = query_list
)
```

#### Query Translator

![query_translator](./images/query_translator.png)

Query translator has the ability to translate queries in one query language into another, like cypher to gql. To achieve this, Awesome-Text2GQL designed and implemented a set of intermediate representation for commonly used graph query languages(ISO-GQL, Cypher, Gremlin, SQL/PGQ, etc.) and their dialects. With ast vistitor's implementations, different graph query language can be translated into the intermediate representation. With the query translator's implementations, intermediate representation can be translated into different graph query language.

``` python
from app.impl.iso_gql.translator.iso_gql_query_translator import IsoGqlQueryTranslator as GQLTranslator
from app.impl.tugraph_cypher.ast_visitor.tugraph_cypher_query_visitor import TugraphCypherAstVisitor

query_visitor = TugraphCypherAstVisitor()
gql_translator = GQLTranslator()
cypher = "MATCH (n:Person)-[:HAS_CHILD*1]->(n) WHERE n.name = 'Vanessa Redgrave' RETURN n"

# translate cypher to gql
success, query_pattern = query_visitor.get_query_pattern(cypher)
if success:
    gql = gql_translator.translate(query_pattern)
```

### Generalizer

Generalizer supports the corpus generalization based on the given query template and question template. Users can use generalizer to construct a large scale corpus dataset across multiple database instance from a limited number of existing corpus templates.

#### Question Generalizer

Question generalizer has the ability to generalize the given natural language question into similar questions with different language styles, and the symantic similarity is ensured with the given corresponding query. This generalization aims to increase the linguistic diversity of corpus to simulate the real world Text2GQL scenario.

``` python
from app.core.generalizer.question_generalizer import QuestionGeneralizer
from app.core.llm.llm_client import LlmClient

llm_client = LlmClient(model="qwen-plus-0723")
question_generalizer = QuestionGeneralizer(llm_client)
corpus_pair_list = [
    [
        "MATCH (n:Person)-[:HAS_CHILD*1]->(n) WHERE n.name = 'Vanessa Redgrave' RETURN n",
        "Who are Roy Redgrave's second generations?"
    ]
]

# generalize question
generalized_corpus_pair_list = []
for corpus_pair in corpus_pair_list:
    query = corpus_pair[0]
    question = corpus_pair[1]
    generalized_question_list = question_generalizer.generalize(
        query=query,
        question=question
    )
    for generalized_question in generalized_question_list:
        generalized_corpus_pair_list.append((query, generalized_question))
    generalized_corpus_pair_list.append((query, question))
```

#### Query Generalizer

![query_generalizer](./images/query_generalizer.png)

Query generalizer has the ability to generalize the given query into queries with similar query pattern on the given schema. With the intermediate representation for graph query languages, Awesome-Text2GQL can translate a query into intermediate query pattern, and the similar query pattern can be constructed with different variables on different schema. This generalization aims to migrate existing query patterns onto new database instance efficiently.

``` python
from app.core.generalizer.query_generalizer import QueryGeneralizer
from app.impl.tugraph_cypher.ast_visitor.tugraph_cypher_query_visitor import TugraphCypherAstVisitor

db_id = "movie"
instance_path = "../app/impl/tugraph_cypher/generalizer/base/db_instance/movie"
query_visitor = TugraphCypherAstVisitor()
query_generalizer = QueryGeneralizer(db_id, instance_path)
query_template="MATCH (n {name: 'Carrie-Anne Moss'}) RETURN n.born AS born"

# generalize cypher query
query_list = query_generalizer.generalize_from_cypher(query_template=query_template)
```

``` python
from app.core.generalizer.query_generalizer import QueryGeneralizer
from app.impl.iso_gql.translator.iso_gql_query_translator import IsoGqlQueryTranslator as GQLTranslator
from app.impl.tugraph_cypher.ast_visitor.tugraph_cypher_query_visitor import TugraphCypherAstVisitor

db_id = "movie"
instance_path = "../app/impl/tugraph_cypher/generalizer/base/db_instance/movie"
query_generalizer = QueryGeneralizer(db_id, instance_path)
query_visitor = TugraphCypherAstVisitor()
gql_translator = GQLTranslator()
query_template="MATCH (n:Person)-[:HAS_CHILD*1]->(n) WHERE n.name = 'Vanessa Redgrave' RETURN n"

# generalize gql query
query_list = []
success, query_pattern = query_visitor.get_query_pattern(query_template)
if success:
    query_pattern_list = query_generalizer.generalize(query_pattern=query_pattern)
    for query_pattern in query_pattern_list:
        query = gql_translator.translate(query_pattern)
        query_list.append(query)
```

### Generator(Future Plan)

Generator supports the corpus and database instance generation only based on a given schema with the assistance of LLM. Users can use generator to generate corpus and correponding database instance with only a schema when there is no avaliable corpus.

#### Data Generator

Data generator aims to generate actual database instance from a graph schema with the assistance of LLM. LLM can understand the structure of a graph schema and generate corresponding data to construct a real database instance for query execution. 

#### Question Generator

Question generator aims to generate natural language questions on given database instance with the assistance of LLM. LLM can understand the structure and the content of the database instance then ask natural language questions as a database user.

#### Query Generator

Query generator aims to generate the actual query of a corresponding natural language question with the current Text2GQL ability of LLM. The generated question may not be executable or grammarly correct, but can be use as a reference for further annotation.

## Development Guide

To make Awesome-Text2GQL supports the corpus construction of more types of graph query languages, we welcome contribution to the implementation of ast visitor, query translator, and schema parser of new graph query languages. If you find the compatibility of current intermediate representation is not enough for the new graph query language, we also welcome contribution to the intermediate representation.

### Introduction to Intermediate Representation

The clause class is the core of Awesome-Text2GQL's intermediate representation for graph query languages. Currently clause class has match clause, return clause, where clause and with clause as subclasses. The design of subclasses might be updated in the future for the compatibility of more graph query languages

+ Match Clause: the intermediate representation for pattern match

+ Return Clause: the intermediate representation for item return.

+ Where Clause: the intermediate representation for condition expression.

+ With Clause: the intermediate representation for variable control.

### Graph Query Language Implementation Guideline

#### Implement AST Visitor

The ast visitor class is a virtual class and should be implemented for different graph query language, like cypher ast visitor or gql ast visitor. Implementation on each graph query language should be able to parse the given query, visit the abstarct syntax tree, then return the graph pattern(a list of clauses) of the given query as the intermediate representation for further translation or generalization. See `app/impl/tugraph_cypher/ast_visitor/tugraph_cypher_ast_visitor.py` as an example.

#### Implement Query Translator

The query translator class is a virtual class and should be implemented for different graph query language, like cypher query translator or gql query translator. Implementation on each graph query language should implement the translate function to turn a list of clauses(the intermediate representation) into an actual query aligned to the grammar of corresponding language, and implement the grammar check function to check if a query is grammatically correct. See `app/impl/iso_gql/translator/iso_gql_query_translator.py` as an example.

#### Implement Schema Parser

The schema parser class is a virtual class and should be implemented for different DBMS, like neo4j schema parser or tugraph schema parser. Implementation on each DBMS should be able to parse the correspongding schema file, whether it's a set of queries or a json file, then return a in memory schema graph for query generalization. See `app/impl/tugraph_cypher/schema/schema_parser.py` as an example.

## Future Plan

Awesome-Text2GQL plans to support the Text2GQL corpus construction from only a schema with the generator module.

## Contribution

Do the following steps before submitting your code.

### Code Formatting

```
poetry run ruff format .
```

### Code Checking

```
poetry run ruff check ./app ./examples --fix
```

if all check passed, you can submit your code.

### Submitting Code

create a pull request, link it to a related issue, then wait for the project maintainer to review your changes and provide feedback. If your pull request is finally approved by our maintainer, we will merge it. Other details can reference to our [contributing document](https://github.com/TuGraph-family/community/blob/master/docs/CONTRIBUTING.md).

### Contributors Wall
<a href="https://github.com/TuGraph-family/Awesome-Text2GQL/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=TuGraph-family/Awesome-Text2GQL&max=200" />
</a>

## Attention

This project is still under development, suggestions, issues or pull requests are welcome.
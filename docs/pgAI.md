## Installation in PostgreSQL 17 on Debian 12 (LXC)

The intent is to be able to query the many CSV files i have on a local harddrive.

### Install EDB repos.
```bash
root@postgresql:~# apt install edb-pg17-aidb edb-pg17-pgfs edb-pg17-pgvector-0
Reading package lists... Done
Building dependency tree... Done
Reading state information... Done
The following NEW packages will be installed:
  edb-pg17-aidb edb-pg17-pgfs edb-pg17-pgvector-0
0 upgraded, 3 newly installed, 0 to remove and 0 not upgraded.
Need to get 6,634 kB of archives.
After this operation, 36.8 MB of additional disk space will be used.
Get:1 https://downloads.enterprisedb.com/pdZe6pcnWIgmuqdR7v1L38rG6Z6wJEsY/enterprise/deb/debian bookworm/main amd64 edb-pg17-aidb amd64 1.0.7-1.bookworm [6,179 kB]
Get:2 https://downloads.enterprisedb.com/pdZe6pcnWIgmuqdR7v1L38rG6Z6wJEsY/enterprise/deb/debian bookworm/main amd64 edb-pg17-pgfs amd64 1.0.4-1.bookworm [157 kB]
Get:3 https://downloads.enterprisedb.com/pdZe6pcnWIgmuqdR7v1L38rG6Z6wJEsY/enterprise/deb/debian bookworm/main amd64 edb-pg17-pgvector-0 amd64 0.8.0-3.bookworm [299 kB]
Fetched 6,634 kB in 2s (3,670 kB/s)      
Selecting previously unselected package edb-pg17-aidb.
(Reading database ... 22920 files and directories currently installed.)
Preparing to unpack .../edb-pg17-aidb_1.0.7-1.bookworm_amd64.deb ...
Unpacking edb-pg17-aidb (1.0.7-1.bookworm) ...
Selecting previously unselected package edb-pg17-pgfs.
Preparing to unpack .../edb-pg17-pgfs_1.0.4-1.bookworm_amd64.deb ...
Unpacking edb-pg17-pgfs (1.0.4-1.bookworm) ...
Selecting previously unselected package edb-pg17-pgvector-0.
Preparing to unpack .../edb-pg17-pgvector-0_0.8.0-3.bookworm_amd64.deb ...
Unpacking edb-pg17-pgvector-0 (0.8.0-3.bookworm) ...
Setting up edb-pg17-aidb (1.0.7-1.bookworm) ...
Setting up edb-pg17-pgvector-0 (0.8.0-3.bookworm) ...
Setting up edb-pg17-pgfs (1.0.4-1.bookworm) ...
Processing triggers for postgresql-common (267.pgdg120+1) ...
Building PostgreSQL dictionaries from installed myspell/hunspell packages...
Removing obsolete dictionary files:
root@postgresql:~#
```
### Connect to DB as user `postgres`
```bash
root@postgresql:~# psql -h localhost -U postgres
Password for user postgres: 
psql (17.2 (Debian 17.2-1.pgdg120+1))
SSL connection (protocol: TLSv1.3, cipher: TLS_AES_256_GCM_SHA384, compression: off, ALPN: postgresql)
Type "help" for help.
```
### Create extensions
```postgresql
postgres=# CREATE EXTENSION aidb CASCADE;
NOTICE:  installing required extension "vector"
CREATE EXTENSION
postgres=# CREATE EXTENSION pgfs;
CREATE EXTENSION
```
### Check extensions
```postgresql
postgres=# \dx
                                List of installed extensions
  Name   | Version |   Schema   |                        Description                         
---------+---------+------------+------------------------------------------------------------
 aidb    | 1.0.7   | aidb       | aidb: makes it easy to build AI applications with postgres
 pgfs    | 1.0.4   | pgfs       | pgfs: enables access to filesystem-like storage locations
 plpgsql | 1.0     | pg_catalog | PL/pgSQL procedural language
 vector  | 0.8.0   | public     | vector data type and ivfflat and hnsw access methods
(4 rows)

postgres=# 
```
### Which models can i use?
```postgresql
postgres=# SELECT * FROM aidb.list_registered_models();
 name  |  provider  |    options    
-------+------------+---------------
 bert  | bert_local | {"config={}"}
 clip  | clip_local | {"config={}"}
 t5    | t5_local   | {"config={}"}
 dummy | dummy      | {"config={}"}
(4 rows)

postgres=#
```

[BERT](https://en.wikipedia.org/wiki/BERT_(language_model)) (Bidirectional Encoder Representations from Transformers) is a transformer-based model that is used for natural language processing tasks such as text classification, text generation, and text completion.

[CLIP](https://openai.com/index/clip/) (Contrastive Language-Image Pre-training) is a model that learns visual concepts from natural language supervision.

[T5](https://en.wikipedia.org/wiki/T5_(language_model)) is a text-to-text transformer model that converts input text into output text.

DUMMY is a placeholder model that can be used for testing purposes.

Other models we can use are `openai_embeddings` and `openai_completions`.

### Creating a retriever for locally stored CSV files.
#### Create a storage location
```postgresql

```
#### 
#### Create a new volume
```postgresql

```

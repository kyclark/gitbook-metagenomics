# SQLite in Python

SQLite \(https://www.sqlite.org\) is a lightweight, SQL/relational database that is available by default with Python \(https://docs.python.org/3/library/sqlite3.html\).  By using `import sqlite3` you can interact with an SQLite database.  So, let's create one, returning to our earlier Centrifuge output.

```
$ cat tables.sql
create table tax (
    tax_id integer primary key,
    tax_name text not null,
    ncbi_id int not null,
    tax_rank text default '',
    genome_size int default 0,
    unique (ncbi_id)
);

create table sample (
    sample_id integer primary key,
    sample_name text not null,
    unique (sample_name)
);

create table sample_to_tax (
    sample_to_tax_id integer primary key,
    sample_id int not null,
    tax_id int not null,
    num_reads int default 0,
    abundance real default 0,
    num_unique_reads integer default 0,
    unique (sample_id, tax_id)
);
```

We are going to create a minimal database to track the abundance of species in various samples.  The biggest rule of relational databases is to not repeat data.  There should be one place to store each entity.  For us, we have a "sample" \(the Centrifuge ".tsv" file\), a "taxonomy" \(NCBI tax ID/name\), and the relationship of the sample to the taxonomy.  I have my own particular naming convention when it comes to relational tables/fields:

1. Name tables in the singular, e.g. "sample" not "samples"
2. Name the primary key \[tablename\] + underscore + "id", e.g., "sample\_id"
3. Name linking tables \[table1\] + undescore + "to" + underscore + \[table2\]
4. Always have a primary key that is an auto-incremented integer

You can instantiate the database by calling `make db` in the "csv" directory.




# SQLite in Python

SQLite \([https://www.sqlite.org\](https://www.sqlite.org\)\) is a lightweight, SQL/relational database that is available by default with Python \([https://docs.python.org/3/library/sqlite3.html\](https://docs.python.org/3/library/sqlite3.html\)\).  By using `import sqlite3` you can interact with an SQLite database.  So, let's create one, returning to our earlier Centrifuge output.

```
drop table if exists tax;
create table tax (
    tax_id integer primary key,
    tax_name text not null,
    ncbi_id int not null,
    tax_rank text default '',
    genome_size int default 0,
    unique (ncbi_id)
);

drop table if exists sample;
create table sample (
    sample_id integer primary key,
    sample_name text not null,
    unique (sample_name)
);

drop table if exists sample_to_tax;
create table sample_to_tax (
    sample_to_tax_id integer primary key,
    sample_id int not null,
    tax_id int not null,
    num_reads int default 0,
    abundance real default 0,
    num_unique_reads integer default 0,
    unique (sample_id, tax_id),
    foreign key (sample_id) references sample (sample_id),
    foreign key (tax_id) references tax (tax_id)
);
```

This database uses foreign keys \(https://sqlite.org/foreignkeys.html\) to maintain relationships between tables.  We will see in a moment how that prevents us from accidentally create "orphan" data.

We are going to create a minimal database to track the abundance of species in various samples.  The biggest rule of relational databases is to not repeat data.  There should be one place to store each entity.  For us, we have a "sample" \(the Centrifuge ".tsv" file\), a "taxonomy" \(NCBI tax ID/name\), and the relationship of the sample to the taxonomy.  I have my own particular naming convention when it comes to relational tables/fields:

1. Name tables in the singular, e.g. "sample" not "samples"
2. Name the primary key \[tablename\] + underscore + "id", e.g., "sample\_id"
3. Name linking tables \[table1\] + underscore + "to" + underscore + \[table2\]
4. Always have a primary key that is an auto-incremented integer

Here is a simple E/R \(entity-relationship\) graph of the schema \(created with SQL::Translator\):

![](/assets/tables.png)

You can instantiate the database by calling `make db` in the "csv" directory to _first remove the existing database_ and then recreate it by redirecting the "tables.sql" file into `sqlite3`:

```
$ make db
find . -name centrifuge.db -exec rm {} \;
sqlite3 centrifuge.db < tables.sql
```

You can then run `sqlite3 centrifuge.db` to use the CLI \(command-line interface\) to the database.  Use `.help` inside SQLite to see all the "dot" commands \(they begin with a `.`\):

```
$ sqlite3 centrifuge.db
SQLite version 3.13.0 2016-05-18 10:57:30
Enter ".help" for usage hints.
sqlite>
```

I often rely on the `.schema` command to look at the tables in an SQLite db.  If you run that, you should see essentially the same thing as was in the "tables.sql" file.  An alternate way to create the database is to use the `.read tables.sql` command from within SQLite to have it read and execute the SQL statements in that file.

We can manually insert a record into the `tax` table with a statement like this:

```
sqlite> insert into tax (tax_name, ncbi_id) values ('Homo sapiens', 3606);
```

What's interesting to note is how SQLite treats strings and numbers exactly like Python -- strings must be in quotes, numbers should be plain.  We can add a dummy "sample" and link them like so:

```
sqlite> insert into sample (sample_name) values ('foo');
sqlite> insert into sample_to_tax (sample_id, tax_id, num_reads, abundance)
   ...> values (1, 1, 100, .01);
```

To see what's in our database, use a `select` statement.  I will first use `.headers on` to see the column names 

```
sqlite> .headers on
sqlite> select * from tax;
tax_id|tax_name|ncbi_id|tax_rank|genome_size
1|Homo sapiens|3606||0
sqlite> select * from sample;
sample_id|sample_name
1|foo
```

That's still a bit hard to read, so we can set `.mode column` to see a bit better:

```
sqlite> select * from sample;
sample_id   sample_name
----------  -----------
1           foo
sqlite> select * from tax;
tax_id      tax_name      ncbi_id     tax_rank    genome_size
----------  ------------  ----------  ----------  -----------
1           Homo sapiens  3606                    0
sqlite> select * from sample_to_tax;
sample_to_tax_id  sample_id   tax_id      num_reads   abundance   num_unique_reads
----------------  ----------  ----------  ----------  ----------  ----------------
1                 1           1           100         0.01        0
```

Often what we want is to `join` the tables so we can see just the data we want, e.g.:

```
sqlite> select s.sample_name, t.tax_name, s2t.num_reads
   ...> from sample s, tax t, sample_to_tax s2t
   ...> where s.sample_id=s2t.sample_id
   ...> and s2t.tax_id=t.tax_id;
sample_name  tax_name      num_reads
-----------  ------------  ----------
foo          Homo sapiens  100
```

Now let's try to delete the `sample` record after we have turned on the enforcement of foreign keys:

```
sqlite> PRAGMA foreign_keys = ON;
sqlite> delete from sample where sample_id=1;
Error: FOREIGN KEY constraint failed
```

It would be bad to remove our sample and leave the sample/tax records in place.  This is what foreign keys do for us.  \(Other databases -- PostgreSQL, MySQL, Oracle, etc. -- do this without having to explicitly turn on this feature, but keep in mind that this is an extremely lightweight, fast, and easy database to create and administer.  When you need more speed/power/safety, then you will move to another database.\)

Obviously we're not going to manually enter our data by hand, so let's write a script to import some data.  This script is going to be somewhat long, so let's break it down.  Here's the start.  We need to take as arguments the Centrifuge "\*.tsv" \(tab-separated values\) file which is the summary table for all the species found in a given sample.  The script will take one or more of these positional arguments.  It will also take as a named argument the `--db` name of the SQLite database:

```
#!/usr/bin/env python3
"""Load Centrifuge into SQLite db"""

import argparse
import csv
import os
import re
import sqlite3
import sys

# --------------------------------------------------
def get_args():
    """get args"""
    parser = argparse.ArgumentParser(description='Load Centrifuge data')
    parser.add_argument('tsv_file', metavar='file',
                        help='Sample TSV file', nargs='+')
    parser.add_argument('-d', '--dbname', help='Centrifuge db name',
                        metavar='str', type=str, default='centrifuge.db')
    return parser.parse_args()
```






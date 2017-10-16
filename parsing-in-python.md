# CSV Files

Delimited text files are a standard way to distribute non-hierarchical data -- e.g., records that can be represented each on one line.  \(When you get into data that have relationships, e.g., parents/children, then structures like XML and JSON are more appropriate.\)  Let's first take a look at the `csv` module in Python \([https://docs.python.org/3/library/csv.html\](https://docs.python.org/3/library/csv.html\)\) to parse the output from Centrifuge \([http://www.ccb.jhu.edu/software/centrifuge/\](http://www.ccb.jhu.edu/software/centrifuge/\)\).

For this, we'll use some data from a study from Yellowstone National Park \([https://www.imicrobe.us/sample/view/1378\](https://www.imicrobe.us/sample/view/1378\)\).  For each input file, Centrifuge creates two output files: 1\) a file \("YELLOWSTONE\_SMPL\_20723.sum"\) showing the taxonomy ID for each read it was able to classify and 2\) a file \("YELLOWSTONE\_SMPL\_20723.tsv"\) of the complete taxonomy information for each taxonomy ID.  One record from the first looks like this:

```
      readID: Yellowstone_READ_00007510
       seqID: cid|321327
       taxID: 321327
       score: 640000
2ndBestScore: 0
   hitLength: 815
 queryLength: 839
  numMatches: 1
```

One from the second looks like this:

```
          name: synthetic construct
         taxID: 32630
       taxRank: species
    genomeSize: 26537524
      numReads: 19
numUniqueReads: 19
     abundance: 0.0
```

# Read Count by taxID

Let's write a program that shows a table of the number of records for each "taxID":

```
$ cat -n read_count_by_taxid.py
     1    #!/usr/bin/env python3
     2    """Counts by taxID"""
     3
     4    import csv
     5    import os
     6    import sys
     7    from collections import defaultdict
     8
     9    args = sys.argv[1:]
    10
    11    if len(args) != 1:
    12        print('Usage: {} SAMPLE.SUM'.format(os.path.basename(sys.argv[0])))
    13        sys.exit(1)
    14
    15    sum_file = args[0]
    16
    17    _, ext = os.path.splitext(sum_file)
    18    if not ext == '.sum':
    19        print('File extention "{}" is not ".sum"'.format(ext))
    20        sys.exit(1)
    21
    22    counts = defaultdict(int)
    23    with open(sum_file) as csvfile:
    24        reader = csv.DictReader(csvfile, delimiter='\t')
    25        for row in reader:
    26            taxID = row['taxID']
    27            counts[taxID] += 1
    28
    29    print('\t'.join(['count', 'taxID']))
    30    for taxID, count in counts.items():
    31        print('\t'.join([str(count), taxID]))
```

As always, it prints a "usage" statement when run with no arguments.  It also uses the `os.path.splitext` function to get the file extension and make sure that it is ".sum."  Finally, if the input looks OK, then it uses the `csv.DictReader` module to parse each record of the file into a dictionary:

```
$ ./read_count_by_taxid.py
Usage: read_count_by_taxid.py SAMPLE.SUM
$ ./read_count_by_taxid.py YELLOWSTONE_SMPL_20723.tsv
File extention ".tsv" is not ".sum"
$ ./read_count_by_taxid.py YELLOWSTONE_SMPL_20723.sum
count    taxID
80    321332
6432    321327
19    32630
```

That's a start, but most people would rather see the a species name rather than the NCBI taxonomy ID, so we'll need to go look up  the taxIDs in the ".tsv" file:

```
$ cat -n read_count_by_tax_name.py
     1    #!/usr/bin/env python3
     2    """Counts by tax name"""
     3
     4    import csv
     5    import os
     6    import sys
     7    from collections import defaultdict
     8
     9    args = sys.argv[1:]
    10
    11    if len(args) != 1:
    12        print('Usage: {} SAMPLE.SUM'.format(os.path.basename(sys.argv[0])))
    13        sys.exit(1)
    14
    15    sum_file = args[0]
    16
    17    basename, ext = os.path.splitext(sum_file)
    18    if not ext == '.sum':
    19        print('File extention "{}" is not ".sum"'.format(ext))
    20        sys.exit(1)
    21
    22    tsv_file = basename + '.tsv'
    23    if not os.path.isfile(tsv_file):
    24        print('Cannot find expected TSV "{}"'.format(tsv_file))
    25        sys.exit(1)
    26
    27    tax_name = {}
    28    with open(tsv_file) as csvfile:
    29        reader = csv.DictReader(csvfile, delimiter='\t')
    30        for row in reader:
    31            tax_name[row['taxID']] = row['name']
    32
    33    counts = defaultdict(int)
    34    with open(sum_file) as csvfile:
    35        reader = csv.DictReader(csvfile, delimiter='\t')
    36        for row in reader:
    37            taxID = row['taxID']
    38            counts[taxID] += 1
    39
    40    print('\t'.join(['count', 'taxID']))
    41    for taxID, count in counts.items():
    42        name = tax_name.get(taxID) or 'NA'
    43        print('\t'.join([str(count), name]))
$ ./read_count_by_tax_name.py YELLOWSTONE_SMPL_20723.sum
count    taxID
6432    Synechococcus sp. JA-3-3Ab
80    Synechococcus sp. JA-2-3B'a(2-13)
19    synthetic construct
```

# tabchk

Talk about tab-check program.

```
$ cat -n tabchk.py
     1	#!/usr/bin/env python3
     2
     3	import argparse
     4	import csv
     5	import os
     6	import sys
     7
     8	# --------------------------------------------------
     9	def get_args():
    10	    parser = argparse.ArgumentParser(description='Check a delimited text file')
    11	    parser.add_argument('file', metavar='str', help='File')
    12	    parser.add_argument('-s', '--sep', help='Field separator',
    13	                        metavar='str', type=str, default='\t')
    14	    parser.add_argument('-l', '--limit', help='How many records to show',
    15	                        metavar='int', type=int, default=1)
    16	    parser.add_argument('-d', '--dense', help='Not sparse (skip empty fields)',
    17	                        action='store_true')
    18	    return parser.parse_args()
    19
    20	# --------------------------------------------------
    21	def main():
    22	    args = get_args()
    23	    file = args.file
    24	    limit = args.limit
    25	    sep = args.sep
    26	    dense = args.dense
    27
    28	    if not os.path.isfile(file):
    29	        print('"{}" is not a file'.format(file))
    30	        sys.exit(1)
    31
    32	    with open(file) as csvfile:
    33	        reader = csv.DictReader(csvfile, delimiter=sep)
    34
    35	        for i, row in enumerate(reader):
    36	            vals = dict([x for x in row.items() if x[1] != '']) if dense else row
    37	            flds = vals.keys()
    38	            longest = max(map(len, flds))
    39	            fmt = '{:' + str(longest + 1) + '}: {}'
    40	            print('// ****** Record {} ****** //'.format(i+1))
    41	            for key, val in vals.items():
    42	                print(fmt.format(key, val))
    43
    44	            if i + 1 == limit:
    45	                break
    46
    47	# --------------------------------------------------
    48	if __name__ == '__main__':
    49	    main()
$ tabchk.py -d imicrobe-blast-annots.txt
// ****** Record 1 ****** //
sample_id             : 1
project_name          : Acid Mine Drainage Metagenome
ontologies            : ENVO:01000033 (oceanic pelagic zone biome),ENVO:00002149 (sea water),ENVO:00000015 (ocean)
project_id            : 1
source_mat_id         : 33
sample_type           : metagenome
sample_description    : ACID_MINE_02 - 5-Way (CG) Acid Mine Drainage Biofilm
sample_name           : ACID_MINE_02
collection_start_time : 2002-03-01 00:00:00.0
collection_stop_time  : 2002-03-01 00:00:00.0
site_name             : Richmond Mine
latitude              : 40.666668
longitude             : -122.51667
site_description      : Acid Mine Drainage
country               : UNITED STATES
region                : Iron Mountain, CA
library_acc           : JGI_AMD_5WAY_IRNMTN_LIB_20020301
sequencing_method     : dideoxysequencing (Sanger)
dna_type              : gDNA
num_of_reads          : 180713
habitat_name          : waste water
temperature           : 42
sample_acc            : JGI_AMD_5WAY_IRNMTN_SMPL_20020301
```






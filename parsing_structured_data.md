# Parsing Structured Data

We've looked at parsing structured data formats like delimited text files (commas, tabs) and FASTA format.  If there's one thing that we do not have a shortage of in bioinformatics, it's structured file formats.  Let's look a some more examples.

## JSON

JavaScript Object Notation is a very popular way to exchange data on the internet.  It has mostly supplanted XML as the de facto hierarchical text file (i.e., not just lines of records all having the same fields but things that can contain things like how genes can contain exons, introns, CDS, etc.).  

Let's say you have a PubMed ID (27208118), and you'd like to get the details of the article.  NCBI has web-accessible tools for this (http://www.ncbi.nlm.nih.gov/books/NBK25499/).  Here is how we can use ```wget``` to fetch a JSON file:

```
$ wget --quiet -O 27208118.json 'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/esummary.fcgi?db=pubmed&retmode=json&id=27208118'
$ head 27208118.json
{
    "header": {
        "type": "esummary",
        "version": "0.3"
    },
    "result": {
        "uids": [
            "27208118"
        ],
        "27208118": {
```

And here is a simple way to incorporate this into a Perl script and parse the JSON.  For this to work, we'll do ```panda install JSON::Tiny```.  Here's the script:

```
$ cat -n pubmed1.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use File::Temp;
     4 	use JSON::Tiny;
     5
     6 	# http://www.ncbi.nlm.nih.gov/books/NBK25499/
     7 	constant $PUBMED_URL = 'https://eutils.ncbi.nlm.nih.gov/entrez/eutils/'
     8 	                     ~ 'esummary.fcgi?db=pubmed&retmode=json&id=';
     9
    10 	sub MAIN (Int $pubmed-id=27208118) {
    11 	    my ($tmpfile, $tmpfh) = tempfile();
    12 	    $tmpfh.close;
    13 	    run(«wget --quiet -O $tmpfile "$PUBMED_URL$pubmed-id"»);
    14 	    my $json = $tmpfile.IO.slurp;
    15 	    my $data = from-json($json);
    16 	    $tmpfile.IO.unlink;
    17
    18 	    if $data{'result'}{$pubmed-id}.defined {
    19 	        my %pubmed = $data{'result'}{$pubmed-id};
    20 	        put "$pubmed-id = %pubmed{'title'} (%pubmed{'lastauthor'})";
    21 	    }
    22 	    else {
    23 	        put "Cannot find PubMed ID '$pubmed-id'";
    24 	        exit 1;
    25 	    }
    26 	}
$ ./pubmed1.pl6
27208118 = Potential Mechanisms for Microbial Energy Acquisition in Oxic Deep-Sea Sediments. (Heidelberg JF)
$ ./pubmed1.pl6 00000001
Cannot find PubMed ID '00000001'
```

On lines 3-4, I brought in a couple of modules I'll need to create a temporary file and parse JSON.  Line 7 have a ```constant``` declaration to indicate I don't want anything to change this string which is the URL of the "esummary" tool.  At line 11, I use ```tempfile``` to create a temporary file.  This can be harder to get right than you imagine.  Here's a reason:

```
Knock, knock.
Race condition.
Who's there?
```

So to avoid race conditions, please use the ```File::Temp``` module.  I don't actually need the file handle, so I ```close``` it so I can pass the temp filename to the ```wget``` command.  Then I can ```slurp``` in the file (line 14) and parse the JSON (line 15) before getting rid of the tempfile (line 16). 

If the call to "esummary" was successful, then the given PubMed ID would exist in the ```result``` section of the JSON.  From there I can easily extract the "title" and "lastauthor" (line 20).  If not, I need to let the user know and exit with a failure status (lines 23-24).

As it happens, we use web services like this quite a bit, so Perl has much better tools than just running ```wget```.  Here is an example using the "LWP::Simple" module (use ```panda install``` to install it):

```
$ cat -n pubmed2.pl6
     1 	#!/usr/bin/env perl6
     2
     3 	use LWP::Simple;
     4 	use JSON::Tiny;
     5
     6 	# http://www.ncbi.nlm.nih.gov/books/NBK25499/
     7 	constant $PUBMED_URL = 'http://eutils.ncbi.nlm.nih.gov/entrez/eutils/'
     8 	                     ~ 'esummary.fcgi?db=pubmed&retmode=json&id=';
     9
    10 	sub MAIN (Int $pubmed-id=27208118) {
    11 	    my $lwp  = LWP::Simple.new;
    12 	    my $json = $lwp.get("$PUBMED_URL$pubmed-id");
    13 	    my $data = from-json($json);
    14
    15 	    if $data{'result'}{$pubmed-id}.defined {
    16 	        my %pubmed = $data{'result'}{$pubmed-id};
    17 	        put "$pubmed-id = %pubmed{'title'} (%pubmed{'lastauthor'})";
    18 	    }
    19 	    else {
    20 	        put "Cannot find PubMed ID '$pubmed-id'";
    21 	        exit 1;
    22 	    }
    23 	}
[gila@~/work/metagenomics-book/perl6/structured-data]$ ./pubmed2.pl6
27208118 = Potential Mechanisms for Microbial Energy Acquisition in Oxic Deep-Sea Sediments. (Heidelberg JF)
```

Now we don't have to worry about those tempfiles.  LWP will impersonate a web browser and ```get``` the URL for us, returning the JSON (or XML or HTML or whatever) to us.  The rest looks the same.
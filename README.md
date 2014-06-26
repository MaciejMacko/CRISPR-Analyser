#CRISPR Analyser

##Summary
This module is a collection of C++ scripts that has the primary function of finding off targets for a CRISPR site as quick as possible, for any species. This is achieved by: 

1. Creating an index of every CRISPR site within the target genome 
2. Loading that index into memory (requires 2.4GB~ memory for Human)
3. Brute force checking the specified site against every possible CRISPR site  

For memory efficiency the index only contains a unique id and sequence, so is only really useful if you create a database to run along side the analyser. You can see ours here:

http://www.sanger.ac.uk/htgt/wge/

The code for the website is on github:

https://github.com/htgt/WGE

##Speed
The initial load time of the index will depend entirely on the speed of the disk you store it on. An index will be 2-3gb which is _all_ loaded into memory; for us this usually takes about 5 seconds per index. This is quite high if you are searching for a single CRISPR at a time, so generally I recommend using the server so you only have to load the index once.

Once the application has loaded all the CRISPRs into memory it usually takes 2-4 seconds to find all off targets with up to 4 mismatches for a gRNA. The program can be easily modified to check for more than 4 mismatches, but it was not feasible for us to store off targets with any more than 4 mismatches in our database.

##Build
The actual build step is very simple, and simply requires running make:

```
git clone https://github.com/htgt/CRISPR-Analyser.git
cd CRISPR-Analyser
make
```

But before the program can be used you must create an index:

###Find all CRISPRs within the genome
Unfortunately this is the hardest part at the moment because I haven't had time to re-write my original script I generated from eons ago. When I get time this script will be added as a command inside crispr_analyser

This step produces a .tsv of all CRISPRs in a given genome, which can be run directly into a psql copy statement to quickly add rows to a table.

For now you will have to get this script

[]()

###Create index
Once you have generated the tsv file(s) you can give them to the crispr_analyser index command with the -i flag:

```
./bin/crispr_analyser index -i /lustre/scratch109/sanger/ah19/chr1-10_crisprs_new.csv -i /lustre/scratch109/sanger/ah19/chr11_onwards_crisprs_new.csv -o /lustre/scratch109/sanger/ah19/crisprs_human.bin -s Human -a GRCh37 -e 1
```

The other options:

<pre>
 -i    The input files
 -o    The file to output the binary index to
 -s    The species name 
 -e    The species ID in your database
 -a    The assembly
</pre>

You can now skip to the usage section unless you want to use more than 1 species

#####Indexing a second species
If you are storing the CRISPRs in a database, all CRISPRs must have a unique ID so to add a second species you must add one more flag:

<pre>
 -f    The offset that your database IDs begin with
</pre>

For example, to add the second species you should start the CRISPR ids in your database at 300,000,001 and set -f to 300,000,000:

```
./crispr_analyser index -i /lustre/scratch109/sanger/ah19/mouse_first_10_fixed.tsv -i /lustre/scratch109/sanger/ah19/mouse_after_10_fixed.tsv -o /lustre/scratch109/sanger/ah19/GRCm38_index.bin -s Mouse -a GRCm38 -e 2 -f 300000000
```

Meaning 300,000,000 should be subtracted from whatever ID is given to get a species localised ID, as 300,000,001 here corresponds to Mouse CRISPR 1

##Usage

###Retrieve an ID given a gRNA

###Find off targets

###Run the server

##Server Usage
The server supports returns JSON or JSONP if a callback is given. 

The server has two modes of operation:

1. Finding off targets given an id (or ids)
2. Retrieving an id given a single 20bp gRNA sequence

###Off target searching
To find off targets there are two required parameters:

- ids - this can be a single id, or multiple can be specified in a comma separated string, like "253453634,23645743,473539233"
- species - this is a string with whatever species you have added to the server

You can view the json output by simply visiting the appropriate URL:

[http://localhost:8080/api/off_targets?ids=245377726&species=human](http://localhost:8080/api/off_targets?ids=245377726&species=human)

Or from a javascript console:

```javascript
$.getJSON("http://localhost:8080/api/off_targets?callback=?", {"ids":"245377736", "species": "Human"}, function (data) { console.log(data) });
```

Which will return the following wrapped in a callback:

```JSON
{
    "245377736": {
        "id": 245377736,
        "off_targets": [
            2500136,
            2872353,
            6251446,
            ...
            290706188,
            293805107,
            294577530
        ],
        "off_target_summary": "{0: 1, 1: 0, 2: 0, 3: 9, 4: 126}"
    }
}
```

#####Multiple IDs:
[http://localhost:8080/api/off_targets?ids=245377726,245377730&species=human]([http://localhost:8080/api/off_targets?ids=245377726,245377730&species=human)

###Sequence searching
Sequence searching works in a very similar way:

Viewing the JSON by going straight to the URL:

[http://localhost:8080/api/search?seq=TTAATTGTTTAGCAGTGTCA&species=mouse](http://localhost:8080/api/search?seq=TTAATTGTTTAGCAGTGTCA&species=mouse)

Or from a javascript console:

```javascript
$.getJSON("http://localhost:8080/api/search?callback=?", {"seq":"TTAATTGTTTAGCAGTGTCA", "species":"mouse"}, function (data) { console.log(data) });
```

Which returns:
```JSON
[342905617]
```

# Geographic Text Analysis
## Doing spatial humanities with texts.

### Do you ponder...

- what Natural Language Processing is and how it is relevant to History/the Humanities?
- faster ways to identify place names in text files and match these to lat/long coordinates?
- what the pitfalls of these automatic approaches can be (esp. for non-English languages)?

### The Basics

Natural Language - human generated language vs. formal computing languages (or just tabular data)
Natural Language Processing - research field encompassing linguistics, computer science, artificial intelligence that works to allow machines to process natural language
Rule-based vs. statistical NLP - writing rules vs. using machine learning algorithms to analyze a corpus
Corpus - set of texts (hand keyed, OCR'd, born digital)
Corpus linguistics -
Geographic information science -
Gazetteer -
Lexicon -



### GTA
#### What is it?
GTA helps us answer questions like:

- Where is [a corpus] talking about?
- What is [a corpus] saying about these places?
- What is [a corpus] saying about specific themes in proximity to these places?

GTA allows scholars to create re-usable geo-spatial data from **natural language texts**.

This is a crucial departure from many early spatial humanities projects that were based on geocoding **lists** of place names. Geocoding could be done by hand (manually finding a lat/long for a place name) or automatically (scanning pre-existing gazetteers for perfect or near perfect string matches to place names).

It has evolved from depending on string matches to using a combination of rule-based methods nd Machine Learning algorithms to **parse texts** and **georesolve locations**. While it is possible to perform these tasks separately, geoparsing software combines the NLP and GIS tasks.

Have a look at the geoparser `pipeline_scheme.png` file in this repo. Most GTA projects use a similar pipeline that combines tools originally developed for corpus linguistics and geographic information science.

1. Tokenization
2. Part-of-speech tagging
3. Lemmatization
4. Named Entity Recognition
5. Lookup of Entities in Digital Gazetteers
6. Disambiguation of Gazetteer record 'matches'
7. Georesolution to 1 coordinate set

In this workshop, we will be using the [Edinburgh Geoparser](https://www.ltg.ed.ac.uk/software/geoparser/), developed by the Language Technology Group at the University of Edinburgh. It is one of the first geoparsers to be geared towards humanities research applications. It was designed to work with relatively modern English running texts.

> Beatrice Alex, Kate Byrne, Claire Grover and Richard Tobin. 2015. [Adapting the Edinburgh Geoparser for Historical Georeferencing](https://www.euppublishing.com/doi/pdfplus/10.3366/ijhac.2015.0136). International Journal for Humanities and Arts Computing, 9(1), pp. 15-35, March 2015

> Claire Grover, Richard Tobin, Kate Byrne, Matthew Woollard, James Reid, Stuart Dunn, and Julian Ball. 2010b. [Use of the Edinburgh Geoparser for georeferencing digitised historical collections](http://homepages.inf.ed.ac.uk/grover/papers/PTRS-A-2010-Grover-3875-89.pdf). Philosophical Transactions of the Royal Society A, 368(1925):3875-3889.

The Edinburgh Geoparser, or EG for short, is free for research.

*Let's take a break for questions before we get started with the tutorial.*

## Edinburgh Geoparser Tutorial

### Setup
#### Mac users
- Download the [Edinburgh Geoparser](https://www.ltg.ed.ac.uk/software/geoparser/)
- decompress the file
- Save EG folder to your local drive (any location that suits you, close to your `home` folder)
- Open `scripts`>`setup` file in any text editor (don't open by double clicking on the file, use CTL+Open and then select your text editor)
- Replace  `Darwin?1[012345]*)` with `Darwin?1[0-9]*)`
- Open your terminal
- `cd` to your EG folder
- List EG folder contents `ls ./geoparser-v1.1`
- test that EG functions with this command using a sample input file provided with the program `cat ../in/172172.txt | ./run -t plain -g geonames -o ../out 172172`


#### Windows users
- Install [Docker](https://docs.docker.com/docker-for-windows/install/)
- Create Docker account
- Start Docker app
- Open up shell to run Docker commands
- We will be using a Docker container of the [Edinburgh Geoparser](https://hub.docker.com/r/kmcdono2/eg/)
- Download the `172172.txt` file from this Github repo and put it in a local folder
- `cd` to that folder in your shell
- Run the following command to test: `cat 172172.txt | docker run -i -v $(pwd):/out kmcdono2/eg:latest ./run -t plain -g geonames -o /out 172172`

(Alternate Windows solution: [MobaXterm](https://mobaxterm.mobatek.net/))


### Basic file structure

- README: run instructions
- in: files you want to parse and geo-resolve
- out: results
- lib: processing libraries
- resolve: programs for geo-resolution
- scripts: scripts to run EG

More on EG [file structure and pipeline](http://groups.inf.ed.ac.uk/geoparser/documentation/v1.1/html/overview.html).


### Considerations and Parameters

EG combines all of the parsing and georesolution steps into one pipeline and produces a set of output files that include some basic map visualizations.

EG works well with plain text **input**. It can also accept google books files and XML, under certain conditions.
`-t plain`
`-t gb`
`-t ltgxml`

EG accesses a few commonly used **gazetteers**. You can also specify a local gazetteer (if you have one that is specific to your time/place).

`-g geonames` Geonames
`-g os`       UK Ordnance Survey
`-g deep`     Historical placenames in England
`-g plplus`   Pleiades+, ancient Greek and Roman world

EG results include a range of files. Define the **output** directory and file name in the run command.

`-o /out 172172`

`172172.out.xml` - XML file containing text and all word-level metadata produced during the process
`172172.gaz.xml` - ranked list of geo-resolution candidates for each extracted place entity
`172172.display.html` - visualization of geoparsed text, map, and list of geo-coordinates for each place

There are other ways to limit your results:

`-top` - this parameter creates `172172.display-top.html` that only maps top-ranked locations

You can express a preference for results within a geographical area.

`-l lat long radius score` - bounding circle
Example: `-l `

`-lb W N E S score` - bounding box
(WNES are decimal degrees)
`cat ../in/172172.txt | ./run -t plain -g geonames
-lb -141.002701 83.110619 -52.620201 41.681019 2 -
o ../out 172172`

(Score of `2` means that a gazetteer result within the bounding box or circle has 2x weight as a result that is outside that area.)

Do you want to run **multiple files** through in one go? Download the `run-multiple-files.sh` script from this repo. Place it in the scripts directory. Run the following command to make it executable: `chmod u+x run-multiple-files.sh`

Then, still from the scripts directory, run:
`./run-multiple-files.sh -i ../in -o ../out`

In this case, `-i` specifies input directory and `-o` the output directory.

### Run EG

Let's run EG on Darwin's *The Origin of Species*
Download the `origins.txt` file from the repo and put it in your `in` directory.

#### Mac users
Be sure you've navigated to the scripts directory:
`cd scripts`
Then run the following (your output directory must already exist, e.g. `out`):
`cat ../in/origin.txt | ./run -t plain -g geonames -o ../out origin`

#### Windows users
Run:
`cat origin.txt | docker run -i -v $(pwd):/out kmcdono2/eg:latest ./run -t plain -g geonames -o /out origin`


### Evaluating your output

Let's take a look at one of the out.xml files to see what is happening to the text.

`<text>`
`<p>` Paragraphs
`<s>` Sentences
`<w>` Words

`<standoff>`
`<ents>` This element occurs twice because the NER process has two runs. 1) rule-based to identify and classify entities (`ner-rb`). EG entities can be classified as date, location, person, and organization. 2) verbs & verb phrases for detecting events. We are mostly interested in the first pass to identify and classify.

`<part>` Links an entity element back to its position in the text ("start word" [sw] and
"end word" [ew] both have word ids [e.g. w26]).


### Exporting your new metadata

It is possible to export specific metadata fields (place name in text, gazetteer record ID, country of location, lat/long, feature type, etc.).

Run this command to transform your XML results into a TSV (tab separated values) file:

`./bin/sys-i386-snow-leopard/lxprintf -e
"ent[@type='location']" "%s\t%s\t%s\t%s\t%s\n" "normalizespace(parts)"
"@gazref" "@in-country" "@lat" "@long" < ./
out/burtons.out.xml > ./out/burtons.out.tsv`

You can also download this script as a file and run it just like we did the multiple files script.

`chmod u+x extract-to-tsv.sh`

Then: `./extract-to-tsv.sh < ../out/burtons.out.xml > ../out/burtons.out.tsv`

You can edit the script (either in the command line or in the file) to include or exclude metadata fields. The script above, for ex., does not include @feat-type.

Check out the full [EG documentation](http://groups.inf.ed.ac.uk/geoparser/documentation/v1.1/html/).

*Questions?*

### New Inputs

Now lets try using some texts that interest you.
If you have plain text files in English, put them into the `in` directory. Use the same run scripts from above, but be sure to edit the in and out files or directories to match your new file names.

If you don't have any plain text, try downloading something from the [Internet Archive](https://archive.org/) or use one of the sample txts in the `in` directory.

*What kinds of problems do you encounter?   
Why do you think these problems occur?  
How would you need to adapt the geoparser to solve these problems?*

## Beyond EG

EG is an excellent tool, with developers who are interested in creating version for specific research needs.

However, there are lots of other fish in the GTA sea. If you want to try your hand at some other NLP packages that integrate Machine Learning into Named Entity Recognition, or have texts in non-English languages, for example, check out the following resources:

#### For the text parsing processes:
- [NLTK (python)](https://www.nltk.org/)
- NLP for non-English languages in [R (udpipe)](https://www.r-bloggers.com/natural-language-processing-for-non-english-languages-with-udpipe/)
- [SpaCy, also python](https://spacy.io/)


#### Combined Parsing + Georesolution
- [Mordecai](https://github.com/openeventdata/mordecai)
- [Perdido, for French](http://erig.univ-pau.fr/PERDIDO/)

*Interested in the challenge of evaluating different pipelines for your corpus?*
There is a brand new article by [Milan Gritta et al](https://arxiv.org/abs/1810.12368) that can guide you through this process.



### Workshop Evaluation

Before you leave, please take a moment to fill out this [quick evaluation form](https://stanforduniversity.qualtrics.com/jfe/form/SV_blp4YwVZW0D4dSt). It will help us adapt content for the future!

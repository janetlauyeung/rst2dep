# RST constituent and dependency conversion

Scripts to convert Rhetorical Structure Theory trees from .rs3 format to a
dependency representation and back. 

## rst2dep

This conversion follows Li et al.'s (2013) convention of taking the left-most child of multinuclear relations as the head child governed by the relation applied to the whole multinuc, and attaching each subsequent multinuclear child to the first child using the multinuclear relation; thus if a contrast multinuc with units [2-3] is an elaboration on unit [1], the child [2] will become an elaboration dependent of [1], and [3] will become a contrast dependent of child [2].

By convention, multinuclear relations are converted with relation names ending in `_m`, while satellite RST relations are converted with names ending in `_r`. The original nesting depth is ignored in the conversion, but attachment point height for each dependent is retained in the third column of the output file, allowing deterministic reconstruction of the constituent tree using dep2rst, assuming a projective, hierarchically ordered tree. Conversion of non-projective .rs3 constituent trees to dependencies is also supported, but cannot be reversed currently.

Optionally, users can also specify an additional folder containing subfolders `dep/` and `xml/` with .conllu parses and GUM style XML to add features to the output file.

```
usage: rst2dep.py [-h] [-r ROOT] infiles

positional arguments:
  infiles               file name or glob pattern, e.g. *.rs3

optional arguments:
  -h, --help            show this help message and exit
  -r ROOT, --root ROOT  optional: path to corpus root folder containing a
                        directory dep/ and a directory xml/ containing
                        additional corpus formats
```

## dep2rst

Discourse dependency to RST constituent conversion. The converter assumes target trees are projective and hierarchically ordered. The default strategy for determining constituent nesting order is to look for explicit attachment height encoding as created by rst2dep, otherwise competing nodes are attached as siblings. Alternatively, users can specify `-d rtl` or `-d ltr` to always attach right children below left children, or vice versa.

```
usage: dep2rst.py [-h] [-f {rsd,conllu}] [-d {ltr,rtl,dist}] [-r] file

Script to convert discourse dependencies to Rhetorical Structure Theory trees
in the .rs3 format. Example usage: python dep2rst.py INFILE

positional arguments:
  file                  discourse dependency file in .rsd or .conllu format

optional arguments:
  -h, --help            show this help message and exit
  -f {rsd,conllu}, --format {rsd,conllu}
                        input format
  -d {ltr,rtl,dist}, --depth {ltr,rtl,dist}
                        how to order depth
  -r, --rels            use DEFAULT_RELATIONS for the .rs3 header instead of
                        rels in input data
```

Supported input formats include .rsd and .conllu:

### Input format - .rsd

```
1	Greek court rules	0	_	_	_	2	attribution_r	_	_
2	worship of ancient Greek deities is legal	5	_	_	_	5	preparation_r	_	_
3	Monday , March 27 , 2006	4	_	_	_	5	circumstance_r	_	_
4	Greek court has ruled	0	_	_	_	5	attribution_r	_	_
5	that worshippers of the ancient Greek religion may now formally associate and worship at archeological sites .	0	_	_	_	0	ROOT	_	_
6	Prior to the ruling , the religion was banned from conducting public worship at archeological sites by the Greek Ministry of Culture .	1	_	_	_	5	background_r	_	_
7	Due to that , the religion was relatively secretive .	0	_	_	_	6	result_r	_	_
8	The Greek Orthodox Church , a Christian denomination , is extremely critical of worshippers of the ancient deities .	1	_	_	_	6	background_r	_	_
9	Today , about 100,000 Greeks worship the ancient gods , such as Zeus , Hera , Poseidon , Aphrodite , and Athena .	2	_	_	_	5	background_r	_	_
10	The Greek Orthodox Church estimates	0	_	_	_	11	attribution_r	_	_
11	that number is closer to 40,000 .	0	_	_	_	9	concession_r	_	_
12	Many neo - pagan religions , such as Wicca , use aspects of ancient Greek religions in their practice ;	3	_	_	_	5	background_r	_	_
13	Hellenic polytheism instead focuses exclusively on the ancient religions ,	0	_	_	_	12	contrast_m	_	_
14	as far as the fragmentary nature of the surviving source material allows .	0	_	_	_	13	concession_r	_	_
```
### Input format - .conllu
```
# text = Greek court rules worship of ancient Greek deities is legal
1	Greek	Greek	ADJ	JJ	Degree=Pos	2	amod	_	Discourse=attribution:1->2:0
2	court	court	NOUN	NN	Number=Sing	3	nsubj	_	_
3	rules	rule	VERB	VBZ	Mood=Ind|Number=Sing|Person=3|Tense=Pres|VerbForm=Fin	0	root	_	_
4	worship	worship	NOUN	NN	Number=Sing	10	nsubj	_	Discourse=preparation:2->5:5
5	of	of	ADP	IN	_	8	case	_	_
6	ancient	ancient	ADJ	JJ	Degree=Pos	8	amod	_	_
7	Greek	Greek	ADJ	JJ	Degree=Pos	8	amod	_	_
8	deities	deity	NOUN	NNS	Number=Plur	4	nmod	_	_
9	is	be	AUX	VBZ	Mood=Ind|Number=Sing|Person=3|Tense=Pres|VerbForm=Fin	10	cop	_	_
10	legal	legal	ADJ	JJ	Degree=Pos	3	ccomp	_	_

# text = Monday, March 27, 2006
1	Monday	Monday	PROPN	NNP	Number=Sing	0	root	_	Discourse=circumstance:3->5:4
2	,	,	PUNCT	,	_	4	punct	_	_
3	March	March	PROPN	NNP	Number=Sing	4	compound	_	_
...

```
# The CoBaLD Eng corpus

A corpus with full three language level markup in CONLL-U Plus standard. Enhanced-UD version: for the conversion of the English dataset we used E-UD formalism, as opposed to our [Russian dataset](https://github.com/CobaldAnnotation/CobaldRus) where the first version complies with the basic UD principles and uses sligtly different standard (CONLL-U with ten columns). 

<a href="https://creativecommons.org/licenses/by-nc/4.0/"><img src="https://img.shields.io/static/v1?label=license&message=CC-BY-NC-4.0&color=green"/></a>

### Citing

The paper is to be published soon

# Contents
1. [Dataset](#dataset)
2. [Corpus Markup](#corpus-markup)
3. [The conversion of the Compreno markup to the UD format](#the-conversion-of-the-compreno-markup-in-the-ud-format)<br />
3.1 [Tokenization](#tokenization )<br />
3.2 [The conversion of parts of speech and grammatical features](#the-conversion-of-parts-of-speech-and-grammatical-features)<br />
3.3 [The conversion of the syntactic heads](#the-conversion-of-the-syntactic-heads)<br />
3.4 [The conversion of the dependent constituents](#the-conversion-of-the-dependent-constituents)<br />
4. [Main problems and their solutions](#main-problems-and-their-solutions)

# Dataset
Our corpus consists of news from the [BBC News](http://mlg.ucd.ie/datasets/bbc.html) dataset (D. Greene and P. Cunningham. "Practical Solutions to the Problem of Diagonal Dominance in Kernel Document Clustering", Proc. ICML 2006.)

The dataset contains 3 markup levels:

- morphological,
- syntactic,
- semantic.

We have chosen the UD markup format as the one which seems the most popular now, however, it does not suggest semantic information. Full semantic markup can be found in the Compreno markup format, which includes word meanings (defined in terms of semantic classes &ndash; semantic fields, provided for each word meaning), and semantic relations between them (which as marked as deeps slots, or semantic roles). We use the [Enhanced UD formalism](https://universaldependencies.org/u/overview/enhanced-syntax.html).

To obtain the corpus in the UD format enriched by the semantic markup, we have organized the project pipeline as follows. 


# Corpus Markup

At the first stage, the corpus was annotated with the [ABBYY Compreno semantic markup](https://github.com/compreno-semantics): we obtained the markup automatically with the help of the ABBYY Compreno parser and checked it manually afterwards. Nevertheless, the number of categories in the full ABBYY Compreno markup is rather big: more than 200,000 semantic classes and more than 330 semantic roles, which seems too much both for the machine learning of the semantic parsers on the dataset of the given volume and for a number of application tasks the corpus can be used for.

To make the formalism simpler, we decided to use so-called generalized semantic classes, which denote not exactly the meaning of a word but are hyperonyms for larger word classes. For instance, all words with motion semantics belong to the hyperonym class MOTION, and all words denoting humans &ndash; in the semantic class HUMAN.

More than that, we reduced the number of the semantic roles. For example, full ABBYY Compreno markup suggests different roles for different characteristic dependencies (that is, weight, speed, size, and so on). In the generalized variant, all such characteristics correspond to one characteristical role. Or, the Compreno model has several slots for temporal relations: Time (‘yesterday’, ‘in two days’), Time\_Situation (‘when everybody comes’), Time\_Being (‘in times of Ivan the Terrible’). In the generalized presentation, all these roles correspond to only one Time slot.

As a result, the number of the hyperonym semantic classes used in the markup was reduced to about 650 classes, and the number of the semantic roles to 143 slots.


# The conversion of the Compreno markup to the UD format

Please refer to this paper if you mention the CoBaLD conversion (Compreno to UD) in your work
```
Ivoylova Alexandra, Dyachkova Darya, Petrova Maria, Michurina Mariia. 2023.
The problem of linguistic markup conversion: the transformation of the Compreno
markup into the UD format // Computational Linguistics and Intellectual Technologies
```


In the ABBYY Compreno markup represent constituency trees which can be extracted from the model parser through API. Each token is provided with both its semantic class and the semantic relation with its head.

The markup can also be provided with surface, or syntactic, roles, coreference and non-tree links, however, the purpose of the given dataset was only the semantic markup. The only surface slot mentioned in the markup is the $Dislocation slot (the $ sign denotes surface slots in the model) &ndash; it is the slot for the dislocated constituents (dislocated constituents are the ones that syntactically depend on one core, while semantically &ndash; on the other core). The only non-tree link we extracted in order to comply with the Enhanced UD principles was the `ref' tag which denotes the connection between a relative pronoun and its antecedent as in the following example:

![ref](https://github.com/CobaldAnnotation/CobaldEng/assets/71529585/2385a6a6-7a0b-4a81-ac75-ba5e1cb44577)

Unlike the UD format, the bracket format does not allow one to indicate morphological information in the markup, that is, parts of speech and grammatical features. Nevertheless, this information is available as well: the ABBYY Compreno parser builds the parsing trees for the sentences, where each node is provided with full grammatical and semantic information, namely, morphological and syntactic features (grammemes), syntactic and semantic relations, semantic classes, non-tree links, and so on:

![](img/Aspose.Words.100d08f2-5e68-4f73-8006-21098ad50dcb.005.png)

During the conversion of the ABBYY Compreno markup into the UD format, all the necessary information is taken from the parsing trees. After that, the markup looks as follows:

```
# text = I play in a heavy metal band. 
1	I	I	PRON	Pronoun	Case=Nom|Number=Sing|Person=1|PronType=Prs	2	nsubj	2:nsubj	_	Agent	BEING
2	play	play	VERB	Verb	Mood=Ind|Number=Sing|Person=1|Tense=Pres|VerbForm=Fin	0	root	0:root	_	Predicate	TO_PERFORM
3	in	in	ADP	Preposition	_	7	case	7:case	_	_	PREPOSITION
4	a	a	DET	Article	Definite=Ind|PronType=Art	7	det	7:det	_	_	ARTICLES
5	heavy	heavy	ADJ	Adjective	Degree=Pos	6	amod	6:amod	_	Ch_Parameter	WEIGHT_CHAR
6	metal	metal	NOUN	Noun	Number=Sing	7	compound	7:compound	_	Fabricative	SUBSTANCE
7	band	band	NOUN	Noun	Number=Sing	2	obl	2:obl:in	SpaceAfter=No	Locative	HUMAN
8	.	.	PUNCT	PUNCT	_	2	punct	_	_	_	_
```

The final UD-markup includes three levels: morphological, syntactic, and semantic. The UD presentation has its own morphology and syntax, therefore, the corresponding information in the ABBYY Compreno model is converted into the Enhanced UD format. As far as the semantic relations are concerned, the UD format does not have the semantic level, so the information about the SCs and the deep slots is added to the UD markup in the way it is presented in Compreno.

Now let us examine the conversion of morphology and syntax in more detail.

## Tokenization 
**Dividing**

Unlike the UD format, Compreno has so called non-morphological lexemes, such as ‘as much as’, ‘according to’, and so on. The thing is that these elements include the space in their body. To convert such elements into the UD format, one has to divide them into separate parts with the help of the special script. It entails the additional work on providing them with part of speech tags and the semantic and syntactic dependencies.

**Merging**

In most cases, the ABBYY Compreno model treats parts of composite words as separate tokens. It concerns words like ‘re-distribution’ (‘re’ + ‘distribution’), ‘anti-revolutionary’ (‘anti’ + ‘revolutionary’), names with numbers (‘TU-104’), or ordinal numbers written with combinations of a number and its grammatical inflection (‘10th day’). Such cases are joined in one UD token with the help of the script.

## The conversion of the syntactic heads

The syntactic heads are converted into the UD format from the bracket ABBYY Compreno format. The heads for the punctuation marks are assigned through a special algorithm. In most cases, the syntactic and the semantic heads coincide. The exception is ellipsis and dislocation.

Other significant structural differences between the two formats are the following:

1.  Copula ‘be’: in the constructions such as *‘the girl was beautiful’*, *‘Peter was a teacher’* and alike the UD considers ‘beautiful’, ‘teacher’ as heads, while in the Compreno ‘be’ is the head;
    
2.  Coordination: the UD makes the first member of the coordination the head for other coordination members, while in Compreno all homogeneous members depend on their real head;
    
3.  In cases like ‘including <smth>’ Compreno makes ‘including’ the head, while UD treats it as a preposition. We have to change the head through post-processing here.
    
In all such cases we converted Compreno structures to UD structures.

## The conversion of the dependent constituents
[Here](https://universaldependencies.org/u/dep/) you can see a table of categories for the dependency relations in the UD. In Compreno, the number of the categories is much bigger, and the categories are organized in a different way: each deep (semantic) slot can correspond to a number of surface (syntactic) slots. On the one hand, such a model allows one to define the semantic and syntactic relations more precisely and in more detail, on the other hand, it includes much more items than the UD model.

As we have decided to keep to the UD format here as the one which is better known for the users, the syntactic relations are presented in the UD format. Nevertheless, practice shows that it is not always easy to make the choice between the categories (for instance, the boundaries between the obj, iobj and obl seem sometimes rather vague). Therefore, the results of the conversion may contain some differences from the original format, but it seems that it should not influence the learning of the parsers significantly.

As the Enhanced UD format supposes ellipsis restoration, we have also added this feature to our annotation which looks like in the following example:

```
# text = "I'm not prejudiced, I'll sell to able-bodied people if I have to" 
1	"	"	PUNCT	PUNCT	_	5	punct	_	SpaceAfter=No	_	_
2-3	I'm	_	_	_	_	_	_	_	_	_	_
2	I	I	PRON	Pronoun	Case=Nom|Number=Sing|Person=1|PronType=Prs	5	nsubj	5:nsubj	SpaceAfter=No	Object	BEING
3	'm	be	VERB	Verb	Mood=Ind|Number=Sing|Person=1|Tense=Pres|VerbForm=Fin	5	cop	5:cop	_	Predicate	BE
4	not	not	PART	Particle	_	5	advmod	5:advmod	_	_	PARTICLES
5	prejudiced	prejudiced	ADJ	Adjective	Degree=Pos	0	root	0:root	SpaceAfter=No	State	CH_BY_WORLD_OUTLOOK_EDUCATION_AESTHETIC
6	,	,	PUNCT	PUNCT	_	9	punct	_	_	_	_
7-8	I'll	_	_	_	_	_	_	_	_	_	_
7	I	I	PRON	Pronoun	Case=Nom|Number=Sing|Person=1|PronType=Prs	9	nsubj	9:nsubj	SpaceAfter=No	Agent	BEING
8	'll	will	AUX	Verb	Mood=Ind|Number=Sing|Person=1|Tense=Pres|VerbForm=Fin	9	aux	9:aux	_	_	AUXILIARY_VERBS
9	sell	sell	VERB	Verb	Mood=Ind|Number=Plur|VerbForm=Fin	5	parataxis	0:root|5:parataxis	_	Predicate	TO_GIVE
10	to	to	ADP	Preposition	_	12	case	12:case	_	_	PREPOSITION
11	able-bodied	able-bodied	ADJ	Adjective	Degree=Pos	12	amod	12:amod	_	Characteristic	PHYSICAL_PSYCHIC_CONDITION
12	people	people	NOUN	Noun	Number=Plur	9	obl	9:obl:to	_	Possessor	HUMAN
13	if	if	SCONJ	Conjunction	_	15	mark	15:mark	_	_	CONJUNCTIONS
14	I	I	PRON	Pronoun	Case=Nom|Number=Sing|Person=1|PronType=Prs	15	nsubj	15:nsubj	_	Experiencer	BEING
15	have	have	VERB	Verb	Mood=Ind|Number=Sing|Person=1|Tense=Pres|VerbForm=Fin	9	advcl	9:advcl:if	_	Condition	MODALITY
16	to	to	ADP	Preposition	_	15	xcomp	16.1:mark	SpaceAfter=No	_	PARTICLES
16.1	#NULL	#NULL	VERB	Verb	Person=1|Tense=Pres|VerbForm=Inf	_	_	15:xcomp	ellipsis	Object_Situation	TO_GIVE
17	"	"	PUNCT	PUNCT	_	5	punct	_	_	_	_
```

# Main problems and their solutions

As we have indicated above, main problems of the conversion are bound with the following differences between the formats:

(1) The Compreno formalism suggests building constituency trees while UD concerns dependency trees, so the problems with dislocation in Compreno arise;

(2) Another difference is that the UD regards punctuation signs as heads like other tokens, therefore, we had to assign the heads for the punctuation marks automatically.
---
title: Word Index
layout: default
parent: Home
nav_order: 5
---

# Word Index

Optionally, the database can include a superset of calculated data essentially related to word forms and their base form (lemma). The word index is built on top of spans data (see [storage](storage) for more details).

Spans are used as the base for building a list of **words**, representing all the unique combinations of each token's language, value, part of speech, and lemma. Each word also has its pre-calculated total count of the corresponding tokens.

Provided that your indexer uses some kind of lemmatizer, words are the base for building a list of **lemmata**, representing all the word forms belonging to the same base form (lemma). Each lemma also has its pre-calculated total count of word forms.

Both words and lemmata have a pre-calculated detailed distribution across documents, as grouped by each of the document's attribute's unique name=value pair.

## Usage Story

Typically, word and lemmata are used to browse the index by focusing on single word forms or words.

For instance, a UI might provide a (paged) list of lemmata. The user might be able to expand any of these lemmata to see all the word forms referred to it. Alternatively, when there are no lemmata, the UI would directly provide a paged list of words.

In both cases, a user might have a quick glance at the distribution of each lemma or word with these steps:

(1) pick one or more document attributes to see the distribution of the selected lemma/word among all the values of each picked attribute. When picking numeric attributes, user can specify the desired number of "bins", so that all the numeric values are distributed in a set of contiguous ranges.

For instance, in a literary corpus one might pick attribute `genre` to see the distribution of a specific word among its values, like "epigrams", "rhetoric", "epic", etc.; or pick attribute `year` with 3 bins to see the distribution of all the works in different years. With 3 bins, the engine would find the minimum and maximum value for `year`, and then define 3 equal ranges to use as bins.

(2) for each picked attribute, display a pie chart with the frequency of the selected lemma/word for each value or (values bin) of the attribute.

## Creation Process

The word and lemma index is created as follows:

1. first, the index is cleared, because it needs to be globally computed on the whole dataset.

2. words are inserted grouping tokens by language, value, POS, and lemma. This means that we define as the same word form all the tokens having these properties equal. The word's count is the count of all the tokens belonging to it. Once words are inserted, their identifiers are updated in the corresponding spans.

3. lemmata are inserted grouping tokens by language and lemma, provided that there is one. The lemma has been assigned to tokens by a POS tagger. Thus each unique combination of language and lemma in a token is a lemma. The lemma's count is the sum of the count of all the words belonging to it. Once lemmata are inserted, their identifiers are updated in the corresponding words.

Their counts index is created as follows:

1. a list of all the combinations of name=value pairs in document attributes (both privileged and non privileged) is calculated from the database. Those attributes marked as numeric are grouped into bins corresponding to the ranges calculated from their minium and maximum values, split in a preset number of classes.

2. for each word, go through all the collected pairs and calculate the count of its spans in each of the document attribute's name=value pair.

3. the lemmata counts are just the sum of the words counts for each lemma.

Of course, once you have the index in the database, you can directly access data at will, or export them for third-party analysis. For instance, this simple query:

```sql
select w.pos, count(w.id) as c, sum(w.count) as f
from word w 
group by w.pos 
order by w.pos;
```

provides the distribution of words into POS categories, with their lexical frequency (`c`) and textual frequency (`f`), e.g.:

| pos   | c     | f      |
|-------|-------|--------|
| ADJ   | 9596  | 114159 |
| ADP   | 175   | 157239 |
| ADV   | 1127  | 64056  |
| AUX   | 276   | 45142  |
| CCONJ | 59    | 43776  |
| DET   | 240   | 100195 |
| INTJ  | 18    | 100    |
| NOUN  | 11472 | 324014 |
| PRON  | 217   | 42082  |
| SCONJ | 82    | 20710  |
| VERB  | 12034 | 117798 |

Or, this variation:

```sql
select length(w.value), count(w.id) as c, sum(w.count) as f
from word w 
group by length(w.value) 
order by length(w.value);
```

provides the list of words and their frequencies correlated to their length (it's an easy prediction that the shortest words, like articles, prepositions, etc. have the highest textual frequencies); etc.

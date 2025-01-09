---
title: Home
layout: home
nav_order: 1
---

# Pythia Documentation

![Pythia](img/pythia-400.png)

## Introduction

Pythia is a minimalist, yet highly-specialized text search engine designed with very specific requirements.

The Pythia project was primarily created to deal with the problem of providing a flexible search engine for literary texts, where **metadata** is huge, its sources are different, and its alignments with the underlying text dramatically varies at all analysis levels, even beyond the boundaries of language analysis.

At first, the engine was created on par with the [Chiron](https://myrmex.github.io/overview/chiron) rhythmic analysis framework, which is capable of collecting literally millions of detailed metadata, up to the sub-phonematic level, about poetry or prose texts. Later, when facing the issues posed by the [Atti Chiari](https://attichiari.unige.it) project, the first corpus of Italian legal acts, the engine has been adopted and further developed to face the unique challenges posed by these texts.

The special preprocessing flow created for this corpus, first and foremost required for the systematic pseudonymization of personal data, has been designed to face scholarly requirements both at linguistic and juridic levels. The pseudonymization process itself is driven by an external configuration, which fits its behavior to the requirements of each specific application. Additionally, further metadata derived from **many different sources** are to be merged in a uniform data model: human operators annotations, typographic markup from the source digital format, metadata about each act, detailed linguistic data from POS tagging, etc. In fact, about 85% of the resulting corpus consists in metadata rather than text.

Additionally, whatever the nature and extent of such metadata, they should fit into a **concordance-based** design, so that the linguistic context can be used to drive our search. This should also happen in such a way that does not limit contextual searches to simple distance counts, without taking into account aligment, relative position and location in the various structures encompassing the searched text, like sentences, verses, paragraphs, etc.

Metadata belonging to many different analysis levels, from various linguistic parts to sound or even typography, are modeled in different ways, and span on different regions of the same text, often freely overlapping. For instance, a poem contains different structures according to the chosen analysis level: it consists of verses from a metrical standpoint, and of sentences from a syntactical one. Both are linguistically defined structures, but they rest on different grounds, and in most cases, they are misaligned and variously overlapping. Should we then add a third structure, like their layout in a page, we would deal with yet another structure, which does not necessarily correspond to either of the two.

Yet, all the metadata ideally **converging** into our search engine have to find a way of coexisting even beyond the limitations posed by standards like TEI, here used to represent a subset of them while encoding the corpus. In fact, multiple, interwoven structures pose limitations to their representation in a TEI source, as far as it relies on a single tree-shaped structure; but we are free to imagine a way of letting them converge into our search engine index, by aligning TEI with other sources and merging all their data together.

Finally, on the more technical side another requirement is limiting the index to standard and popular technologies, so that it can be integrated and reused in third-party projects, even bypassing its engine to directly leverage it.

## Model

The main strategy to meet all these requirements has been designing at a higher abstraction level. Essentially, this implies a sort of de-materialization of text. Traditionally, often in search engines text is modeled as it appears in its serialized form, i.e. essentially as a sequence of characters. These sequences get variously extracted from the indexed text, optionally with the injection of additional metadata, including position, for those systems capable of dealing with it.

The index built from such modeling essentially is a list of these extracted sequences, so that when searching you just match the characters from the searched word to those of the indexed words, optionally filtering them for some metadata. This also allows to find a word at a specific distance from another, provided that our system can retrieve the position of each word in the index; even though, as remarked above, finding two words within the boundaries a specific linguistic structure, like a sentence or a verse, would often make much more sense than a mechanic count.

While of course the base principles remain the same, the Pythia search engine focuses on **objects** rather than just sequences of characters. The index is just a set of objects, each having an unlimited number of metadata. Additionally, objects not only correspond to words, but also to any other structure we may want to index in our documents, from words to sentences, verse, strophes, up to whole documents, or even to groups of documents.

Among these objects, some are _internal_ to the document, like words or sentences; while others are _external_, like document's metadata; that's why internal objects among other properties also include a position in the document itself, which allows for highly granular searches and KWIC-based results.

Thus, search no longer focuses on finding a subsequence of characters in a longer one, but rather on finding objects from their **properties**. Whatever their nature, all objects live in the same set; and they all expose their metadata on a uniform search surface. We find objects through this surface, by matching their metadata: and such objects can be words, but also sentences, verses, documents, or even immaterial things, when for instance we look for a word class (e.g. a past intransitive verb) rather than for a specific instance of it (like it. "andò").

So, the textual value of the searched object (when it has any) is no longer at the core: rather, it's just any of the metadata attached to the object to find. You can specify one or more metadata to find in any logical combination. It might well happen that you do not look for a specific word at all, e.g. when searching for words in a specific language and printed in italic, or for a Latin phrase in an Italian act.

Given this model, the search engine allows for a much richer set of queries, and can deal with any objects, from words to documents and beyond, each having an unlimited set of metadata, whatever their nature, meaning, and source. This way, all our metadata, drawn from all our sources, converge into a uniformly modeled structure, providing a consistent search surface, still open to indefinite expansion.

This approach paves the way to a wide range of search options, not only by virtue of the highly different nature of objects and metadata, but also by means of combinations in search criteria. For instance, in this model we can easily find a specific word (or word class, abbreviation, forename, bold, or whatever other metadata we may want to target) at the start or end of a sentence, by just finding these two objects together. In this case, we will look for a word with all the features we specify, and for a sentence; and we will combine these objects in the search with a positional operator, allowing us to find objects appearing as left- or right-aligned in the document. In the case of a sentence-initial word, we will choose left alignment; while for a sentence-final word, we will choose right alignment.

This is why besides the usual set of logical and grouping operators (equal, not equal, including, starting with, ending with, wildcards, regular expressions, fuzzy matching, and numeric comparisons) the search engine also provides one of **positional operators**, designed for collocations (near, not near, before, not before, after, not after, inside, not inside, overlaps, left-aligned, right-aligned). Each in-document object, either it's a word or something longer, just occupies a specific position or range of positions, like segments on a line; and the purpose of such operators is right to geometrically evaluate their relative alignment of two such segments.

## Indexing

So, the primary purpose of this engine is merging any metadata sources into a uniform searchable surface, representing search targets as objects sets. This implies that the indexing architecture should be highly **modular**, rather than based on a monolithic process.

The text to be indexed flows through the stages of a configurable **pipeline**, including all the modules you want to add logic to it. Such modules may come from stock software, or from third parties, so that the indexing process is highly customizable, and covers any aspect of processing the text, from retrieving to rendering it as in the above example.

In general, the pipeline allows for these classes of **components**, covering preprocessing, open indexing, and even a full environment for reading the indexed texts:

- **source collectors**, used to enumerate the documents from some source (the files in a folder, as well as the documents from a cloud repository, etc.).
- **literal filters**, applied to query text to ensure a uniform preprocessing corresponding to that applied to documents.
- **text filters**, applied to documents as a whole for specific preprocessing and adjustments.
- **attribute parsers**, used to extract metadata from documents, whatever its format (TEI, Excel, etc.).
- **document sort key builders**, used to build sort keys, which represent the default sorting criterion for documents in the UI, and may be built in different ways according to their metadata.
- **date value calculators**, used to calculate a computable date value used to chronologically filter or order documents.
- **tokenizers**, used to split the document's text into “words”, just like in any other search engine.
- **token filters**, used not only to filter the text value of each token by removing unwanted characters (e.g. punctuation, accents, casing differences, etc.), but also to supply additional metadata to it (e.g. add syllable or characters counts, POS tags, etc.).
- **structure parsers**, used to detect textual structures of any extent, like sentences, in a document. These may vary according to the document's format, so that for instance the algorithm used for TEI documents is different from that applied to plain text.
- **structure value filters**, applied to the value of any detected text structure, just like it happens for each single token.
- **text mappers**, used to automatically build a navigable map of the document according to its contents.
- **text renderers**, used to render the source document format into some presentational format, like HTML.

For each of these categories, you are free to pick the modules to use, or even add new ones with highly specific custom logic. This allows dealing with any text type and digital format, inject into it all the metadata we desire from whatever source or service, and keep the system open to customization and expansion, while also fostering software reuse via componentization. For instance, by properly building and configuring the pipeline for _Atti Chiari_ we get a system capable of indexing TEI documents while taking advantage of all the relevant information conveyed by tags; at the same time, leveraging a POS tagging service for them without being hampered by markup; and inject additional metadata from the source format itself (mostly DOCX), while drawing more document metadata from spreadsheets.


# Problem statement

The hypothesis underlying our pseudolabeling approach
is that the value a user derives from a result depends 
on both the relevance of the document to the query
and some properties of the document independent of any query.
This relatively weak statement is not, we think, controversial:
Google's PageRank algorithm is, for example, 
dedicated to finding how central pages are to the Internet
independent of any query.

We have annotations for how much a document matches a query,
which is what we'll call our "primary textual relevance".
We have some concrete indictators of popularity
(citation count, age, etc.)
for a document, which we'll call the "secondary factors".
We often speak of the problem of defining our ideal result order
as balancing primary and secondary factors.

The ideal balance of factors depends on the query.
If your query is for "EMNLP 2016"
then you probably want the hottest papers
that appeared in EMNLP 2016.
If your query is for "deep learning image segmentation of monkeys"
then you have a specific topic in mind,
and probably care most about finding papers that match your topic specifically.

# Formulate hotness

We, for lack of a better term, 
refer to the desirability of a document independent of a query
as the document's "hotness".
We do not really know what this value is for any document.
<!-- Click metrics, how we can't use them? -->
What we need to do is formulate some combination of our concrete metrics
that estimates this unknown desirability value.

We tried a few formulations of hotness,
mostly different ways of summing up citation statistics 
and balancing the age of a document.
We settled on 
the number of recent (in the last three years) citations 
over the age of the document.
<!-- The formula, typeset nicely -->
The intuition is that users are always interested in new papers,
but are willing to see older papers
if those older papers have been cited recently in proportion to their age.

# Training



# Balance concerns (書き直すべきタイトル)
# Model comparisons


This is the second post in our series on search relevance for Semantic Scholar.
The first talked about our creation of 
an Elasticsearch plugin to enable learning to rank.
This post talks about our efforts 
to balance primary and secondary factors when ranking results.

# Problem statement

The hypothesis underlying our efforts
is that the value a user derives from a result depends 
on both the topical relevance of the document to the query
and some popularity or interestingness 
of the document independent of any query.
This is the same hypothesis as is behind 
measures as venerable as Google's PageRank.
We would like to deliver as much value to users as posssible,
and so we need to incorporate this notion into our relevance metric.

We have annotations for how much a document matches a query,
which is what we'll call our "primary textual relevance".
We also have some concrete indicators of popularity
(citation count, age, etc.)
for a document, which we'll call the "secondary factors".
We often speak of the problem of defining 
our ideal result order and associated relevance metric
as balancing primary and secondary factors.

An interesting note is that
the ideal balance of factors depends on the query.
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
Unfortunately we don't have any single value 
which tells us how desirable a paper is to users.
<!-- Click metrics, how we can't use them? -->
What we need to do is formulate some function on the metrics we do have
which estimates this unknown desirability value.

We tried a few formulations of hotness,
mostly different ways of summing up citation statistics 
and balancing the age of a document.
We settled on
the number of recent (in the last three years) citations 
divided by the age of the document.
<!-- The formula, typeset nicely (in an image) -->
The intuition is that users are always interested in new papers,
but are willing to see older papers
if those older papers have been cited recently in proportion to their age.

# Training

In order to train our model to produce our ideal ranking
we need to define a little more carefully what that ideal ranking is.
First we imagine results sorted into bins based on primary textual relevance.
Then, within the bins, we imagine results sorted again based on hotness.
Maybe we want the edges of the bins to overlap a little, 
maybe they should be separate,
but that is the basic idea.

The way we represent this notion at training time is to map 
a tuple of (relevance label, hotness score) -> (new label) 
where the new label is much more fine grained
than the original relevance label. 
Below, `l` is the input label, 
`M` the new maximum label, 
`N` the old maximum label, 
and `h` the hotness score.

<img src="/Users/dericka/Documents/blog-post/pseudolabeling/label-mapping.svg" style="display: block; margin: auto; width: 20em;"/>

We also adjust the gain function:

With a formulation of hotness 
and a way to take into account hotness at traintime
we still need to decide how much exactly to weight hotness.
That is the `H` constant from the mapping function above.

<img src="/Users/dericka/Documents/blog-post/pseudolabeling/textual-relevance-vs-hotness.png"/>

Above we have a graph of the performance of a ranker 
in terms of primary relevance
against the emphasis placed on hotness at traintime.
This graph was our attempt to quantify what we expected intuitively:
emphasizing hotness would lead to a decrease in primary relevance.
Another important takeaway for us was
that primary relevance decreases more quickly the smaller the cutoff.
This trend means we really do have to balance responsibly.
For now we've chosen a hotness coefficient of 1
as that level seemed, under manual analysis, 
to provide the best mix of results.

The differing importance of hotness for different categories of queries 
is reflected in the final results because of the interaction 
between our annotation guidelines and the label mapping.
For query categories where hotness is most important
(author names and venues)
we annotate only two labels: for relevent or not.
The ranker should then, in theory,
learn to rank documents that match these queries strongly on hotness.

# Model comparisons

We hypothesized that the hotness adjusted objective function
would be more difficult for the model to optimize,
or at least that a more expressive model would be required to do a good job.
To test this hypothesis we ran a few experiments with training different models.

| Model (training metric)       | Hotness adjusted NDCG@10 | Unadjusted NDCG@10 |
|:-----------------------------:|:------------------------:|:------------------:|
| Linear (unadjusted)           | .60                      | .75                |
| Linear (hotness adjusted)     | .64                      | .71                |
| LambdaMART (unadjusted)       | .59                      | .76                |
| LambdaMART (hotness adjusted) | .73                      | .71                |


We see that with just textual relevance 
a linear model does as good of a job as LambdaMART.
However,
when we complicate the objective the increased expressivess of LambdaMART
(an ensemble of boosted regression trees)
becomes important to scoring well.

# Future work

There is a lot of future work for us in this area.
Maybe most pressing is the empirical justification and refinement of 
both our formulation of hotness
and our mapping from (relevance label, hotness) -> (new label).
We did some manual analysis to select values for now,
but since both are important for the user experience
we want to connect them to observable user metrics.

The point of this effort,
largely in line with the previous Elasticsearch plugin effort,
was also to produce a generic process
as much as any specific product.
We hope to use the "hotness" training process to train a ranker for a
recency focused search experience.
A "recency focused search experience" will hopefully 
stand in sharp contrast to a "sort by recency" experience.
We've probably all had the experience of selecting "sort by recency" 
on some website and getting terrible results 
that just happen to have been published yesterday.
We believe that what users want is 
results that are as recent as possible 
while still also being relevant.
This approach to balancing primary and secondary factors
will hopefully allow us to train a ranker to deliver that experience.

# Conclusion

???

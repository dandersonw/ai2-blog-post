intro - we at S2 use ES, etc.

# Problem Statement

We want to improve search relevance, i.e. we want to return more relevant
results to the user. How are we going to do that?

If we look at an official
ES
[guide](https://www.elastic.co/guide/en/elasticsearch/guide/current/controlling-relevance.html) to
improving search relevance we can see what is available out of the box. Most of
what is available centers around two things: matching multiple fields (title,
abstract), and matching in multiple ways (phrase, bigram). These are ways to get
different numerical features describing the document and its match with the
query. That's great - we want all the information we can get.

A problem arises when we have all this information though. How do we take n
different features and combine them into a final score? The essential
Elasticsearch technique is to write your query as a tree with similar features
grouped together. This allows you to attempt to tune your query by coming up
with better combinations of subtrees or transformations on subtree scores
(e.x. exponentiation). 

<!-- An example ES query -->

That is fine as far as it goes - but it does not go as far as we would like. We
cannot write the arbitrary models we'd like. We can write custom scoring
scripts, but they get only one `_score` value - they cannot see the un-combined
values. <!-- Call out to the values in the example that you cannot see -->

Lastly, it is not clear how to train a model in this context. Previously
training a model at S2 required a live Elasticsearch cluster - a large expense
we want to avoid. We should be able to have our features stored and usable
without an ES cluster. <!-- I probably want to be clearer in this paragraph if
there is space -->

# Solution - a plugin!

The high level goal of our plugin is to enable a normalized learning to rank
workflow with Elasticsearch at the core. In a sentence, we write a Lucene
`Query` that featurizes matching documents according to an arbitrary
specification and either passes those features as a vector to an arbitrary scoring
function or returns them for offline use.

## Feature templates

Features templates are specifications for the features to calculate and pass to
a model. These features to be calculated are theoretically arbitrary; for now we
implemented functionality to express features in terms of normal queries and
scripts.

<!-- An example feature template using a normal query-->

Normal queries do not provide access to all of the information we want though -
they usually provide a score which bakes in multiple calculations. We also
implemented a feature wrapper that reports more primitive values.

<!-- An example feature template using a finer query -->

<!-- The result from that (explicates what the more primitive values are) -->

## Arbitrary scoring functions

Given feature vectors we want produce single scores for documents. We wanted to
support two use cases: experimentation and production. For experimentation we
should be able to specify something dynamically and should not have to restart
the ES cluster. For production we need as much performance as possible.

For the experimentation use case we provide two pieces of functionality: the
ability to specify parameters for scores in the query, and the ability to load
RankLib models by filename. You could specify the parameters for your model in
the query, or call out to a file which can be deployed without restarting the
cluster.

For production we call Java classes by name. This is useful because it allows us
to compile our regression tree based models into very performant Java code.

<!-- Include a note about the compiled LambdaMART? -->
<!-- A note about reranking? -->

## Scraping and storing offline



# The new S3 ranker



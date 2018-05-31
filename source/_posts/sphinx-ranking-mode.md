---
title: Sphinx Ranking Mode(排序模式)
date: 2018-05-31 10:59:11
tags: [sphinx]
categories: 数据库
---

# Ranking overview(概览)

> Ranking (aka weighting) of the search results can be defined as a process of computing a so-called relevance (aka weight) for every given matched document with regards to a given query that matched it. So relevance is in the end just a number attached to every document that estimates how relevant the document is to the query. Search results can then be sorted based on this number and/or some additional parameters, so that the most sought after results would come up higher on the results page.

排序(又名加权)，是基于请求匹配到的结果，计算所谓的相关性(又名权重)的一个程序。 相关性是请求结束后被附加在文档结果中的一个估算出来的数值,表示匹配的文档于请求的关键词相关的程度，然后搜索的结果就能基于这个数值和其他的一些附加的参数进行排序，这样大多数相关的结果就能排在前面。
<!--more-->
> There is no single standard one-size-fits-all way to rank any document in any scenario. Moreover, there can not ever be such a way, because relevance is subjective. As in, what seems relevant to you might not seem relevant to me. Hence, in general case it's not just hard to compute, it's theoretically impossible.

对排序来说，在任何场景中都没有适应所有的情况的标准，甚至可以说不可能有这种标准，因为相关性是一种很主观的东西，比如，对你来说相关性很强，对我来说却没有。因而一般很难去计算，理论上是不可能的。

> So ranking in Sphinx is configurable. It has a of a so-called . A ranker can formally be defined as a function that takes document and query as its input and produces a relevance value as output. In layman's terms, a ranker controls exactly how (using which specific algorithm) will Sphinx assign weights to the document.

在sphinx中， 排序其实是可配置的， 他有一个叫ranker(这里我翻译成排序器)的概念， 根据定义的方法， 把匹配的文档和请求作为输入，输出来一个相关性的值。 简而言之， 一个ranker可以精确的给每个文档计算出相关性的值。

> Previously, this ranking function was rigidly bound to the matching
> 
> 1. So in the legacy matching modes (that is, SPH_MATCH_ALL, SPH_MATCH_ANY, SPH_MATCH_PHRASE, and SPH_MATCH_BOOLEAN) you can not
>
> choose the ranker. You can only do that in the SPH_MATCH_EXTENDED
>
> 1. (Which is the only mode in SphinxQL and the suggested mode in SphinxAPI anyway.) To choose a non-default ranker you can either use
>
> SetRankingMode() with SphinxAPI, or OPTION ranker clause in SELECT
statement when using SphinxQL.

以前，相排序方法被硬性的于匹配模式绑定在一起， 所以在一些老的匹配模式中(比如 SPH_MATCH_ALL, SPH_MATCH_ANY, SPH_MATCH_PHRASE, and SPH_MATCH_BOOLEAN)， 你不能选择ranker(排序器)。你只能在SPH_MATCH_EXTENDED(这也是在sphinxsql和sphinxApi中被建议使用的唯一的一种模式)模式下选择。 如何选择一个非默认的ranker(排序器)，在SphinxApi中使用SetRankingMode()方法，在SphinxQL中设置ranker选项

> As a sidenote, legacy matching modes are internally implemented via the unified syntax anyway. When you use one of those modes, Sphinx just internally adjusts the query and sets the associated ranker, then executes the query using the very same unified code path.

注意，老的匹配模式被内置了统一的语法，当你使用这些模式的时候，sphinx仅仅内部判断请求和设置相应的ranker,然后使用相同的代码路径去执行这些请求。

# Available built-in rankers(内置的ranker)
> Sphinx ships with a number of built-in rankers suited for different
>
> 1. A number of them uses two factors, phrase proximity (aka LCS) and BM25. Phrase proximity works on the keyword positions, while
>
> BM25 works on the keyword frequencies. Basically, the better the degree of the phrase match between the document body and the query, the higher is the phrase proximity (it maxes out when the document contains the entire query as a verbatim quote). And BM25 is higher when the document contains more rare words. We'll save the detailed discussion for later.

Sphinx 内置了一系列的ranker， 用于不同的目的。他们中都是基于两个因素， phrase proximity(又名LCS)和BM25， Phrase proximity用于表示关键字与关键字的位置有关， BM25于关键词的出现的频率有关。基本上， 请求与匹配的文档越接近， phrase proximity就越高(当文档完整的包含整个请求的关键字时最高)。当文档中包含的关键词越多，BM25就越高。我们稍候讨论这些细节

> Currently implemented rankers are:

当前内置的ranker有：

> 1. SPH_RANK_PROXIMITY_BM25, the default ranking mode that uses and combines both phrase proximity and BM25 ranking.

1. SPH_RANK_PROXIMITY_BM25, 默认的ranker,基于hrase proximity and BM25 ranking两个因素

> 2. SPH_RANK_BM25, statistical ranking mode which uses BM25 ranking only (similar to most other full-text engines). This mode is faster but may result in worse quality on queries which contain more than 1 keyword.

2. SPH_RANK_BM25， 当仅仅使用BM25这种排序因素的时候的模式(于大多数其他的全文引擎相似)，这种模式虽然快，但结果的质量不高，很多结果包含的关键词不止一个(即关键字越多，分值越高，但很多时候我们最想要的仅仅是一个完全命中的结果)

> 3. SPH_RANK_NONE, no ranking mode. This mode is obviously the fastest. A weight of 1 is assigned to all matches. This is sometimes called boolean searching that just matches the documents but does not rank them.

3. SPH_RANK_NONE 没有任何排序模式的模式，这种模式很明显最快， 所有匹配的文档的权重都是1， 有时候被成为布尔搜索，这种搜索仅仅搜索文档，但不会排序

> 4. SPH_RANK_WORDCOUNT, ranking by the keyword occurrences count. This computes the per-field keyword occurrence counts, then multiplies them by field weights, and sums the resulting values.

4. SPH_RANK_WORDCOUNT 根据关键字出现的次数排序，这种排序方式的计算是基于每个字段的关键字出现的次数，然后整合这些字段的权重得出的结果。

> 5. SPH_RANK_PROXIMITY, added in version 0.9.9-rc1, returns raw phrase proximity value as a result. This mode is internally used to emulate SPH_MATCH_ALL queries.

> 5. SPH_RANK_PROXIMITY， 这种排序返回的是每个文档于请求的相似程度，这种模式被内置用来在SPH_MATCH_ALL匹配模式的时候排序

> 6. SPH_RANK_MATCHANY, added in version 0.9.9-rc1, returns rank as it was computed in SPH_MATCH_ANY mode earlier, and is internally used to emulate SPH_MATCH_ANY queries.

6. SPH_RANK_MATCHANY， 早期的时候在SPH_MATCH_ANY匹配模式中使用， 返回相关值，在SPH_MATCH_ANY模式中内置的就是这种排序模式

> 7. SPH_RANK_FIELDMASK, added in version 0.9.9-rc2, returns a 32-bit mask with N-th bit corresponding to N-th fulltext field, numbering from 0. The bit will only be set when the respective field has any keyword occurrences satisfying the query.

7. SPH_RANK_FIELDMASK 返回一个32位的掩码， 每个位都对应一个相应的全文字段(不能应该要补零)， 从0开始， 只有当相应的字段有关键字出现的时候才会被置1

> 8. SPH_RANK_SPH04, added in version 1.10-beta, is generally based on the default SPH_RANK_PROXIMITY_BM25 ranker, but additionally boosts the matches when they occur in the very beginning or the very end of a text field. Thus, if a field equals the exact query, SPH04 should rank it higher than a field that contains the exact query but is not equal to it. (For instance, when the query is "Hyde Park", a document entitled "Hyde Park" should be ranked higher than a one entitled "Hyde Park, London" or "The Hyde Park Cafe".)

8. SPH_RANK_SPH04， 基于默认的SPH_RANK_PROXIMITY_BM25模式， 但假如匹配的文档的开头或者结尾出现了，那么这个文档的相关值就会提升，所以，如果某个文档的一个字段完全于请求的关键字一致， 那么这种模式下的排序的位置就应该比文档中包含请求关键字的文档高。(比如，如果请求的关键字是"Hyde Park", "Hyde Park"的文档就会比"Hyde Park, London"或者"The Hyde Park Cafe"的排序高)

> 9. SPH_RANK_EXPR, added in version 2.0.2-beta, lets you specify the ranking formula in run time. It exposes a number of internal text factors and lets you define how the final weight should be computed from those factors.

9. SPH_RANK_EXPR 这种模式让你能在运行的时候指定排序规则， 他暴露了一系列的内置的文本的因素， 让你能基于这些因素计算出最终的权重

> You should specify the SPH_RANK_ prefix and use capital letters only when using the SetRankingMode() call from the SphinxAPI. The API ports expose these as global constants. Using SphinxQL syntax, the prefix should be omitted and the ranker name is case insensitive.

你可以指定一个SPH_RANK_为前缀的排序模式，要全部大写。在SphinxAPI中使用SetRankingMode()方法，这个API中定义了这些模式的全局常量。 在SphinxQL中， 这个前缀要被映射，且ranker的名称是大小写敏感的(就是要指定ranker模式的参数选项)

# Example （例子）
~~~php
// SphinxAPI
$client->SetRankingMode ( SPH_RANK_SPH04 );

// SphinxQL
mysql_query ( "SELECT ... OPTION ranker=sph04" );
~~~
---
title: "Item to Item Collaborative Filtering"
date: 2021-11-16T17:15:59+01:00
draft: false
---

Recommendation algorithms are best known for their use on e-commerce Websites where they use input about a customer's interest to generate a list of recommended items.
Many applications use only the items that customers purchase and explicitly rate to represent their interests but they can also use other attributes, including items viewed, demographic data,
subject interests and favorite artists.

In 2003 Amazon published an industry report presenting their approach to that topic back then,  this post will be its summary.

# Recommendation algorithms

A typical set of challenges each recommendation algorithm is facing:
* A large retailer might have huge amounts of data, tens of millions of customers and millions of distinct catalog items.
* Many applications require the results set to be returned in realtime, in no more than half a second, while still producing high-quality recommendations.
* New customers typically have extremely limited information, based on only a few purchases on product ratings.
* Older customers can have an overflow of information, based on thousands of purchases and ratings.
* Customer data is volatile: Each interaction provides valuable customer data and the algorithm must respond immediately to new information.

Most recommendation algorithms start by finding a set of customers whose purchased and rated items overlap the user's purchased and rated items. The algorithm aggregates items from these
similar customers, eliminates items the user has already purchased or rated and recommends the remaining ones. Two popular versions of these algorithms are `collaborative filtering` and `cluster models`.
Other algorithms - including `search-based methods` and Amazon's `item-to-item collaborative filtering` focus on finding similar *items* not similar *customers*. For each of the user's purchased and rated items,
the algorithm attempts to find similar items then it aggregates and recommends them.

# Traditional Collaborative Filtering

A traditional collaborative filtering algorithm represents a customer as an N-dimensional vector of items, where N is the number of distinct catalog items.
The components of the vector are positive for purchased or positively rated items and negative for negatively rated items. To compensate for best-selling items,
the algorithm typically multiplies the vector components by the inverse frequency (the inverse of the number of customers who have purchased or rated the item), making less well-known items
much more relevant. For almost all customers, this vector is extremely sparse.

The algorithm generates recommendations based on a few customers who are most similar to the user. It can measure the similarity of two customers A and B in various ways, a common one is to measure
the cosine of the angle between the two vectors:

$$
\text{similarity}(\vec{A}, \vec{B}) = \cos(\vec{A}, \vec{B}) = \frac{\vec{A} \bullet \vec{B}}{||\vec{A}|| * ||\vec{B}||}
$$

The algorithm can select recommendations from the similar customers' items using various methods as well, a common technique is to rank each item according to how many similar customers purchased it.

Using collaborative filtering to generate recommendations is computationally expensive. It's $\mathcal{O}(MN)$ in the worst case where $M$ is the number of customers and $N$ is the number of product
catalog items since it examines $M$ customers and up to $N$ items for each customer. However, because the average customer vector is extremely sparse, the algorithm's performance tends to be closer to
$\mathcal{O}(M + N)$. Even so, for very large data sets - such as 10 million or more customers and 1 million or more catalog items - the algorithm encounters severe performance and scaling issues.

It's possible to partially addres these scaling issues by reducting the data size. We can reduce $M$ by randomly sampling customers or discarding customers with few purchases and reduce $N$ by discarding very popular
or unpopular items. It is also possible to reduce the number of items examind by a small, constant factor by partitioning the item space based on product category or subject classification. Dimensionality reduction techniques
such as clustering and principal component analysis can reduce $M$ or $N$ by a large factor.

Unfortunately, all these methods also reduce recommendation quality in several ways:
* If the algorithm examines only a small customer sample, the selected customers will be less similar to the user.
* Item-space partitioning restricts recommendations to a specific product or subject area.
* If the algorithm discards the most popular or unpopular items, they will never appear as recommendations and customers who have purchased only those items will not get recommendations.
* Dimensionality reduction techniques tend to have the same effect by eliminating low-frequenct items.
* And applying them to customer space effectively groups similar customers into clusters, such clustering can also degrade recommendation quality.

# Cluster models

To find customers who are similar to the user, cluster models divide the customer base into many segments and treat the task as a classification problem. The algorithm's goal is to assign the user to the segment
containing the most similar customers. It then uses the purchases and ratings of the customers in the segment to generate recommendations.

The segments typically are created using a clustering or other unsupervised learning algorithm although some applications use manually determined segments. Based on a defined similarity metric most similar customers together
form clusters or segments. As clustering over large data sets is impractical, most applications use various forms of greedy cluster generation. For very large data sets - especially those with high dimensionality - sampling
or dimensionality reduction is necessary.

Once the algorithm generates segments, it computers the user's similarity to vectors that summarize each segment, then chooses the segment with the strongest similarity and classifies user accordingly. Some algorithms
classify users into multiple segments and describe the strength of each relationship.

Cluster models have better online scalability and performance than collaborative filtering because they compare the user to a controlled number of segments rather than the entire customer base. The complex and expensive
clustering is run offline. However recommendation quality is low - quite often numerous customers in the segment are not the most similar customers and the recommendations they produce are less relevant. Using multiple
fine-grained segments could improbe the quality but then the user-segment classification becomes almost as expensive as collaborative filtering.

# Search-based methods

Search- or content-based methods treat the recommendations problem as a search for related items. Given the user's purchased and rated items, the algorithm constructs a search query to find other popular items by the same
author, artist or director or with similar keywords or subjects. If a customer buys the Godfather DVD Collection, for example, the system might recommend other crime drama titles, other titles starring Marlon Brando, or other
movies directed by Francis Ford Coppola.

If the user has few purchases or ratings, seach-based recommendation algorithms scale and perform well. For users with thousands of purchases, however, it's impractical to base a query on all the items. The algorithm must
use a subset or summary of the data, reducing quality. In all cases, recommendation quality is relatively poor. They are often either too general or too narrow. Recommendations should help a customer find and discover new,
relevant and interesting items. Popular items by the same author or in the same subject category fail to achieve this goal.

# Item-to-item collaborative filtering

Rather than matching the user to similar customers, item-to-item collaborative filtering matches each of the user's purchased and rated items to similar items then combines those similar items into a recommendation list.

To determine the most-similar match for a given item, the algorithm builds a similar-items table by finding items that customers tend to purchase together. We could build a product-to-product matrix by iterating through all
item pairs and computing a similarity metric for each pair. However many product pairs have no common customers so the approach is inefficient in terms of processing time and memory usage. The following iterative algorithm
provides a better approach by calculating the similarity between a single product and all related products:

*For each item in product catalog $I_1$*  
&nbsp;&nbsp;&nbsp;&nbsp;*For each customer $C$ who purchased $I_1$*  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*For each item $I_2$ purchased by customer $C$*  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Record that customer purchased $I_1$ and $I_2$*  
&nbsp;&nbsp;&nbsp;&nbsp;*For each customer item $I_2$*  
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;*Compute the similarity between $I_1$ and $I_2$*  

It's possible to compute the similarity between two items in various ways but a common method is to use the cosine measure that was described earlier in which each vector corresponds to an item rather than a customer and
the vector's $M$ dimensions correspond to customers who have purchased that item.

This offline computation of the similar-items table is extremely time intensive, with $\mathcal{O}(N^2M)$ as worst case. In practice, however, it’s closer to
$\mathcal{O}(NM)$, as most customers have very few purchases. Sampling customers who purchase best-selling titles reduces runtime even further, with little reduction in quality.

Given a similar-items table, the algorithm finds items similar to each of the user’s purchases and ratings, aggregates those items, and then recommends the most popular or correlated items.
This computation is very quick, depending only on the number of items the user purchased or rated.

# Conclusion

For large retailers like Amazon, recommendation algorithms provide an effective form of targeted marketing by creating a personalized shopping experience for each customer. At some point in time 
a good recommendation algorithm had to be scalable over very large customer bases and product catalogs, require only subsecond processing time to generate online recommendations,
be able to react immediately to changes in a user’s data and make compelling recommendations for all users regardless of the number of purchases and ratings.

The key to item-to-item collaborative filtering’s scalability and performance was that it created the expensive similar-items table offline. The algorithm’s
online component — looking up similar items for the user’s purchases and ratings — scaled independently of the catalog size or the total number of customers. It was dependent only on how
many titles the user has purchased or rated. Thus, the algorithm was fast even for extremely large data sets. Because the algorithm recommended highly correlated similar items, recommendation quality was great and
unlike traditional collaborative filtering, the algorithm also performed well with limited user data, producing reasonable recommendations based on as few as two or three items.
 

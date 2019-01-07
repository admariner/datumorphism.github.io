---
title: "Association Rules"
excerpt: "Frequent patterns using association rules"
date: 2019-01-06
toc: true
category:
- 'Pattern Mining'
tag:
- 'Basics'
- 'Pattern'
references:
- name: "Data Mining by Jiawei Han, Micheline Kamber, Jian Pei"
  link: ''
weight: 1
---

* ToC
{:toc}

Association rules is a method for pattern mining. In this article, we perform a association rule analysis of some demo data.

## The Problem Defined

Suppose we own a store called KIOSK. Here at KIOSK, we sell 4 different things.

1. Milk
2. Crossant
3. Coffee
4. Fries

We need to know what items are associated with each other when the customers are buying.

We have collected the following data. Be ware that this small amount of data might not be enough for a real world problem.

| INDEX | Items |
|:---:|:---:|
| 1 | croissant, milk |
| 2 | coffee, croissant |
| 3 | coffee, croissant |
| 4 | coffee, croissant, milk |
| 5 | coffee, milk |
| 6 | fries, milk |
| 7 | coffee, croissant, fries |
| 8 | croissant, fries  |
| 9 | croissant, milk |
| 10 | croissant, fries, milk |
| 11 | coffee, croissant, milk |

## The Rule, Support, and Confidence


Association rule should contain a rule, support, and confidence.

$$
\text{Milk} \Rightarrow \text{Croissant} [ \text{support} = 2/5, \text{confidence} = 2/3  ]
$$

In this rule, the first part means that if the customers are buying milk they buy croissant too. **Support** is the probability of the records with both milk and croissant (2) among all the records (5). **Confidence** is the conditional probability that the records has croissant given milk in the record.

Then we define two thresholds that tell us whether this rule if valid.

1. Support threshold
2. Confidence threshold

If the support and confidence are both larger than the thresholds, we say the rule is valid.

<div class="notes--info" markdown="1">
Sometimes it is easier to deal with **support counts** instead of support probability itself.
</div>

It is, however, very hard to calculate association rules with many items without a smart idea. Suppose we have 10 items, the total number of possible association rules is 10!=3628800. It is not practical to explore all the possible combinations.

## Keywords

1. itemset: a set that contains the items such as milk, coffee, etc.
2. k-itemset: an itemset with k items
3. frequent: an itemset is frequent if it's support count is larger than the threshold (minimum support count). In this article, we define the **minimum support count to be 2**.

## Algorithms

We need some algorithms to calculate the association rules. For example, we could use some algorithms to

1. eliminate some of the possible combinations when generating a pool of rules to be calculated,
2. or enable parallel computations,
3. or better search algorithms.

### Apriori Method

Apriori method is based on a simple rule: **Subsets of a frequent k-itemset should also be frequent.**

For example, if {coffee, croissant} is a frequent itemset, {coffee} and {croissant} must be frequent itemsets. This rule allows us to calculate less combination.

The following calculations assumed a **minimum support count of 2**.

#### Identifying Frequent Itemsets

To find all the frequent 1-itemsets, we simple explore all the possible 1-itemsets.

1. Construct itemsets and calculate the support count of all the 1-itemsets.
   
   | 1-itemset | support count |
   |:---:|:---:|
   | {coffee} | 6 |
   | {croissant} | 9 |
   | {fries} | 3 |
   | {milk} | 7 |

2. Drop all 1-itemsets that have support count less than 2. In our example, all the 1-itemsets are frequent.

   | 1-itemset | support count |
   |:---:|:---:|
   | {coffee} | 6 |
   | {croissant} | 9 |
   | {fries} | 3 |
   | {milk} | 7 |


To find all the frequent 2-itemsets, we construct the 2-itemsets using frequent 1-itemsets only.

1. Construct 2-itemsets based on the frequent 1-itemsets. We choose two 1-itemsets from the frequent 1-itemsets, and find the union.

   | 2-itemset | support count |
   |:---:|:---:|
   | {coffee, croissant} | 5 |
   | {coffee, fries} | 1 |
   | {coffee, milk} | 3 |
   | {croissant, fries} | 3 |
   | {croissant, milk} | 5 |
   | {fries, milk} | 2 |

2. Drop itemsets with support count less than 2. We get the frequent 2-itemsets.
 
   | 2-itemset | support count |
   |:---:|:---:|
   | {coffee, croissant} | 5 |
   | {coffee, milk} | 3 |
   | {croissant, fries} | 3 |
   | {croissant, milk} | 5 |
   | {fries, milk} | 2 |

To find all the frequent 3-itemsets, we construct 3-itemsets using frequent 2-itemsets.

1. Construct 3-itemsets by joining 2-itemsets that has 3-2=1 item in common. This is done in two steps. First, we find 2-itemset that has 3-2=1 item in common, e.g., {coffee, croissant}, {coffee, fries}, {coffee, milk}. Secondly, we choose two of them and union, e.g. {coffee, croissant} union {coffee, fries} = {coffee, croissant, fries}.

   | 3-itemset | support count |
   |:---:|:---:|
   | {coffee, croissant, fries} | 1 |
   | {coffee, croissant, milk} | 2 |
   | {coffee, fries, milk} | 0 |
   | {croissant, fries, milk} | 1 |

2. Drop itemsets with support count less than 2.

   | 3-itemset | support count |
   |:---:|:---:|
   | {coffee, croissant, milk} | 2 |

Our calculation stops here since any 4-itemset would require at least two frequent 3-itemsets which we have only one.

However, it is straightforward to construct the 4-itemsets if we had frequent 3-itemsets. We find 3-itemsets that have 4-2=2 items in common. Then we choose any two itemsets from them and union.

#### Generating Association Rules and Calculating the Confidence

Association rules are generated by splitting the itemsets into two subsets without intersections. Suppose we are constructing association rules from {coffee, croissant, milk}. We follow three steps.

1. Find the subsets. Here we have $C^3_1=3$ combinations.
   
   1. {coffee}, {croissant, milk}
   2. {coffee, croissant}, {milk}
   3. {coffee, milk}, {croissant}

2. The rules will be from one of the subsets to another, e.g., {coffee} to {croissant, milk}. We have 6 in total.
3. The confidence of the rule $\text{ {coffee} } \Rightarrow \text{ {croissant, milk} }$ is

   $$
   \text{Confidence} = \frac{ \text{Support Count of {croissant, milk} } }{ \text{Support Count of {coffee} } } = \frac{5}{6} = 83.3\%
   $$
   







局部敏感哈希

LSH是一种索引技术，可以在大型项目集合中高效地搜索最近的邻居，其中每个项目都由某个固定维度的向量表示。该算法是近似算法，但提供了概率保证，即使用正确的参数设置，结果与在整个集合中进行蛮力搜索几乎没有区别。但是，搜索时间肯定会有所不同：LSH很有用，因为查找的复杂度在集合的大小上变为次线性。

从原则上讲，该算法非常简单，但是当我开始使用它时，我只是为了了解它的工作原理而找不到任何简单的实现-所以我自己写了这个。它不是用于生产的，但是，根据您的要求，一旦了解了它的工作原理，就不难发现将其用于生产。

LSH的想法是提出一种哈希方案，该方案将紧邻的项目映射到同一容器，因此是其名称的“位置敏感”部分。起点是选择一系列简单的哈希函数。该家庭的每个成员都使用不同的随机选择的种子初始化。然后，通过将这些简单哈希函数的k个向量输出的值结合在一起，构造出项的局部敏感哈希。直观地看到，一旦k足够大，则两个项目映射到相同的k值序列就不会是巧合了。另一方面，两个项目可能是近邻，但在k个值中的一个或两个中可能仍然不同。为了提高召回率，我们构建了L个这种类似的哈希表，其中使用每个表的k个不同的哈希函数将项目映射到垃圾箱。 LSH索引是这组L表。在搜索时，我们将所有简单的哈希函数应用于查询项，并在每个表中找到其对应的bin。然后，我们只需要检查这些垃圾箱中物品的确切距离即可确定哪个是最近的邻居。

LSH A. Andoni和P. Indyk的原始创建者在本文中对这一理论进行了非常清晰的解释：高维中近似最近邻居的近最佳哈希算法

理论上的主要技巧是提出一组合适的简单散列，这些散列可为您用来确定相似项之间的特定距离量度提供所需的结果。我实现了最常用于L1，L2和余弦距离的系列。还有一个简单的系列可以处理jaccard距离，如果您确实是物品，这很有意义：在这种情况下，整个算法通常被称为“ minhash”。我发现在对数百万个高维项目的集合中进行实时查找时，minhashing非常有用。

如果您想使用更接近生产代码的版本，那么我建议您检查以下项目：

根据提供的度量找到max_results最接近q的索引点

LSH
===

Locality Sensitive Hashing

LSH is an indexing technique that makes it possible to search efficiently for nearest neighbours
amongst large collections of items, where each item is represented by a vector of some fixed dimension.
The algorithm is approximate but offers probabilistic guarantees i.e. with the right parameter
settings the results will rarely differ from doing a brute force search over your whole collection.
The search time will certainly be different though: LSH is useful because the complexity of lookups
becomes sublinear in the size of the collection.

In principle the algorithm is quite simple, but when I was getting to grips with it I couldn't find
any straightforward implementations just to see how it worked - so I wrote this one myself.  It's not
intended for use in production, but, depending on your requirements, you shouldn't find it too hard
to adapt it for production once you understand how it works.

The idea of LSH is to come up with a hashing scheme that maps closely neighbouring items to the same
bin, hence the "locality sensitive" part of its name.  The starting point is to pick a family of simple
hash functions.  Each member of this family is initialised with a different randomly chosen seed.  The
locality sensitive hash for an item is then constructed by joining together the values output by a
vector of k of these simple hash functions.  Intuitively you can see that once k is big enough, it's not
going to be a conincidence if two items map to the same sequence of k values.  On the other hand it's
possible that two items might be close neighbours but might still differ in one or two of the k values.
To increase recall we build L similar hash tables of this kind, where the items are mapped to bins using
k different hash functions for each table.  The LSH index is this set of L tables.  At search time we
apply all the simple hash functions to our query item and find its corresponding bin in each table.
Then we just have to check the exact distance of the items in those bins to decide which are the very
nearest neighbours.

The theory is explained quite clearly in this paper by the original creators of LSH A. Andoni and P. Indyk:
[Near-optimal hashing algorithms for approximate nearest neighbor in high dimensions](http://people.csail.mit.edu/indyk/p117-andoni.pdf)

The main theoretical trick is to come up with a suitable family of simple hashes that gives the desired
result for the particular distance measure you're using to decide how similar items are to one another.
I've implemented the families most commonly used for L1, L2 and cosine distance.  There's also a simple
family that works with jaccard distance, which makes sense if you're items are really sets: in that case
the overall algorithm is often known as "minhash".  I've found minhashing to be extremely useful when
doing realtime lookups in collections of several million high-dimensional items.

If you want to move on to versions that are a bit closer to production code then I recommend you check
out these projects:
* https://github.com/andrewclegg/sketchy/blob/master/sketchy.py  (sketches i.e. LSH for cosine distance)
* https://github.com/hamilton/LocalitySensitiveHashing (minhash i.e. LSH for jaccard distance)

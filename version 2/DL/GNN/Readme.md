### DeepFindr ->https://www.youtube.com/watch?v=0YLZXjMHA-8&list=PLV8yxwGOxvvoNkzPfCx2i8an--Tkt7O8Z&index=1

### Stanford CS224W: Machine Learning with Graphs -> https://www.youtube.com/watch?v=JAB_plj2rbA&list=PLoROMvodv4rPLKxIpqhjhPgdQy7imNkDn


### https://distill.pub/2021/gnn-intro/?utm_source=chatgpt.com
### Books
- Hands-On-Graph-Neural-Networks-Using-Python
- Deep Learning for the Life Sciences: Applying Deep Learning to Genomics, Microscopy, Drug Discovery, and More
- Graph Representation Learning


### 
- GraphDTA: Predicting drug–target binding affinity with graph neural networks
- Interpretable Drug-Target Prediction Using Graph Attention Networks
- DeepDTA: Deep Drug–Target Binding Affinity Prediction
- A Comprehensive Review of Machine Learning and Deep Learning Methods for Drug-Target Interaction Prediction


## Graph Attention Networks 
> N.B. (Collected from Hands-On-Graph-Neural-Networks-Using-Python)

Graph Attention Networks (GATs) are a theoretical improvement over GCNs. Instead of static normalization coefficients, they propose weighting factors calculated by a process called self-attention. The same process is at the core of one of the most successful deep learning architectures: the transformer, popularized by BERT and GPT-3. Introduced by Veličković et al. in 2017, GATs have become one of the most popular GNN architectures thanks to excellent out-of-the-box performance.

In this chapter, we will learn how the graph attention layer works in four steps. This is actually the perfect example for understanding how self-attention works in general. This theoretical background will allow us to implement a graph attention layer from scratch in NumPy. We will build the matrices by ourselves to understand how their values are calculated at each step.

In the last section, we’ll use a GAT on two node classification datasets: Cora, and a new one called CiteSeer. As anticipated in the last chapter, this will be a good opportunity to analyze our results a little further. Finally, we will compare the accuracy of this architecture with a GCN.

By the end of this chapter, you will be able to implement a graph attention layer from scratch and a GAT in PyTorch Geometric (PyG). You will learn about the differences between this architecture and a GCN. Furthermore, you will master an error analysis tool for graph data.

In this chapter, we’ll cover the following topics:

- Introducing the graph attention layer
- Implementing the graph attention layer in NumPy
- Implementing a GAT in PyTorch Geometric



#### Introducing the graph attention layer

The main idea behind GATs is that some nodes are more important than others. In fact, this was already the case with the graph convolutional layer: nodes with few neighbors were more important than others, thanks to the normalization coefficient 

$\frac{1}{\sqrt(deg(i)(deg(j)))}$


This approach is limiting because it only takes into account node degrees. On the other hand, the goal of the graph attention layer is to produce weighting factors that also consider the importance of node features.

Let’s call our weighting factors attention scores and note, $a_{ij}$ , the attention score between the nodes  and . We can define the graph attention operator as follows: $h_i=\sum_{j\epsilon N_i}\alpha_{ij}Wx_j$

An important characteristic of GATs is that the attention scores are calculated implicitly by comparing inputs to each other (hence the name self-attention). In this section, we will see how to calculate these attention scores in four steps and also how to make an improvement to the graph attention layer:
- Linear transformation
- Activation function
- Softmax normalization
- Multi-head attention
- Improved graph attention layer

##### Linear transformation

The attention score represents the importance between a central node i  and a neighbor j. As stated previously, it requires node features from both nodes. In the graph attention layer, it is represented by a concatenation between the hidden vectors $Wx_i$  and $Wx_j$,$[Wx_i||Wx_j]$ . Here, W is a classic shared weight matrix to compute hidden vectors. An additional linear transformation is applied to this result with a dedicated learnable weight matrix $W_{att}$. During training, this matrix learns weights to produce attention coefficients $a_{ij}$. This process is summarized by the following formula:$a_{ij}=W_{att}^T[Wx_i||Wx_j]$.This output is given to an activation function like in traditional neural networks.

##### Activation function
Nonlinearity is an essential component in neural networks to approximate nonlinear target functions. Such functions could not be captured by simply stacking linear layers, as their final outcome would still behave like a single linear layer.

In the official implementation (https://github.com/PetarV-/GAT/blob/master/utils/layers.py), the authors chose the Leaky Rectified Linear Unit (ReLU) activation function. ReLU does not work in less or equal 0 but leaky relu solved it.This is implemented by applying the Leaky ReLU function to the output of the previous step:$e_{ij}=LeakyReLU(a_{ij})$.However, we are now facing a new problem: the resulting values are not normalized!

##### Softmax normalization
We want to compare different attention scores, which means we need normalized values on the same scale. In machine learning, it is common to use the softmax function for this purpose. Let’s call $N_i$ the neighboring nodes of node i, including itself:
$a_{ij}=softmax_j(e_{ij})=\frac{exp(e_{ij})}{\sum_{k\epsilon N_i} exp(e_{i,k})}$.The result of this operation gives us our final attention scores $a_{ij}$. But there’s another problem: self-attention is not very stable.

##### Multi-head attention
This issue was already noticed by Vaswani et al. (2017) in the original transformer paper. Their proposed solution consists of calculating multiple embeddings with their own attention scores instead of a single one. This technique is called multi-head attention.The implementation is straightforward, as we just have to repeat the three previous steps multiple times. Each instance produces an embedding $h_{i}^{k}$, where k is the index of the attention head.There are two ways of combining these results.
* Averaging: With this, we sum the different embeddings and normalize the result by the number of attention heads :
$h_i=\frac{1}{n}\sum_{k=1}^{n}h_{i}^{k}=\frac{1}{n}\sum_{k=1}^n\sum_{j\epsilon N_i}\alpha^{k}_{ij}W^{k}x_{j}$

* Concatenation: Here, we concatenate the different embeddings, which will produce a larger matrix:
$h_i={||}_{k=1}^n h_i^{k}=||_{k=1}^{n}\sum_{j\epsilon N_i}a^k_{ij}W^k x_j$

* In practice, there is a simple rule to know which one to use: we choose the concatenation scheme when it’s a hidden layer and the average scheme when it’s the last layer of the network. The entire process can be summarized by the following diagram:

![Calculating attention scores with multi-head attention](./Images/Calculating%20attention%20scores%20with%20multi-head%20attention.png)

This is all there is to know about the theoretical aspect of the graph attention layer. However, since its inception in 2017, an improvement has been suggested.

##### Improved graph attention layer
Brody et al. (2021) argued that the graph attention layer only computes a static type of attention. This is an issue because there are simple graph problems we cannot express with a GAT. So they introduced an improved version, called GATv2, which computes a strictly more expressive dynamic attention.

Their solution consists of modifying the order of operations. The weight matrix W is applied after the concatenation and the attention weight matrix $W_{att}$ after the $LeakyReLU$ function. In summary, here is the original Graph Attentional Operator, also GAT:
$$ a_{ij}=\frac{exp(LeakyReLU(W_{att}^{t}W[x_i||x_j]))}{\sum_{k\epsilon N_i}exp(LeakyReLU(W_{att}^{t}W[x_i||x_k]))} $$
.And this is the modified operator, GATv2: $$ a_{ij}=\frac{exp(W_{att}^{t}LeakyReLU(W[x_i||x_j]))}{\sum_{k\epsilon N_i}exp(W_{att}^{t}LeakyReLU(W[x_i||x_k]))}$$
Which one should we use? According to Brody et al., GATv2 consistently outperforms the GAT and thus should be preferred. In addition to the theoretical proof, they also ran several experiments to show the performance of GATv2 compared to the original GAT. In the rest of this chapter, we will consider both options: the GAT in the second section and GATv2 in the third section.

#### Implementing the graph attention layer in NumPy
As previously stated, neural networks work in terms of matrix multiplications. Therefore, we need to translate our individual embeddings into operations for the entire graph. In this section, we will implement the original graph attention layer from scratch to properly understand the inner workings of self-attention. Naturally, this process can be repeated several times to create multi-head attention.

The first step consists of translating the original graph attention operator in terms of matrices. This is how we defined it in the last section:

$$ h_i=\sum{j\epsilon N_i}\alpha_{ij}Wx_j $$

By taking inspiration from the graph linear layer, we can write the following:
![](./Images/2.png)

Where $W_{\alpha}$ is a matrix that stores every $a_{ij}$.

In this example, we will use the following graph from the previous chapter:

```py
import networkx as nx
UG=nx.Graph()
UG.add_edges_from([(1,2),(1,3),(3,4),(1,4)])
nx.draw_networkx(UG,
                 pos=nx.spring_layout(UG,seed=0),
                 font_color="white",
                 font_size=14,
                 node_size=1000
                 )
```
![](./Images/3.png)
The graph must provide two important pieces of information: the adjacency matrix with self-loops  and the node features . Let’s see how to implement it in NumPy:


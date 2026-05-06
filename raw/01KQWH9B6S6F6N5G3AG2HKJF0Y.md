# Causal Masking in Attention

![Amit Shekhar](https://pbs.twimg.com/profile_images/1550394637075943424/PAdpHzNV_normal.jpg)

[**Amit Shekhar**](https://x.com/amitiitbhu) [@amitiitbhu](https://x.com/amitiitbhu)

![Causal Masking in Attention](https://pbs.twimg.com/media/HEp1aDAaoAAWzNd?format=jpg&name=small)

In this blog, we will learn about causal masking in attention.

We will start with the introduction of causal masking, understand the problem of seeing future tokens through an example, and then walk through its implementation to see how masked attention prevents the model from accessing future tokens.

Let's get started.

As we all know, the attention layer is the most important component of the Transformer architecture and the key building block behind LLMs (Large Language Models).

Inside the attention layer, we use causal masking. This ensures that a token can attend only to itself and past tokens, never to future tokens.

Causal masking is a mechanism that uses a mask matrix to block future positions in the attention score matrix, ensuring each token attends only to itself and past tokens, never to future tokens.

Let's try to understand this with an example.

Take the sentence: "I love teaching AI"

Token positions:

```markdown
[ I | love | teaching | AI ]
  1     2        3        4
```

Let's see what happens without causal masking.

### Without Causal Masking

- While predicting "love", the model could attend to "teaching" and "AI".
- While predicting "teaching", it could already see "AI".

This is not something we want.

We should not allow the model to see the future, because doing so would introduce information leakage.

If future tokens are visible, the model can learn shortcuts by using words that it is supposed to predict as inputs. This leads to unrealistically good training performance but breaks the fundamental assumption of language modeling: predicting the next token using only past context.

During inference, future tokens do not exist.

If the model relied on future information during training, it would fail when asked to generate text step by step.

Causal masking ensures that training conditions match inference conditions, forcing the model to learn genuine language patterns instead of memorizing answers.

So, if we allow the model to see the future, training becomes easier, but generation becomes incorrect.

Let's see what happens with causal masking.

### With Causal Masking

- While predicting "love", the model attends to "I".
- While predicting "teaching", it attends to "I" and "love".
- While predicting "AI", it attends to "I", "love", and "teaching".

Here, we do not allow the model to see the future, so it is forced to behave exactly as it would during inference.

Now, let's understand how this is implemented using a mask matrix.

## Implementation of Causal Masking

In the attention layer, we compute an attention score matrix.

An attention score is a number that indicates how relevant one token is to another token.

- Each token compares itself with all other tokens.
- The attention score measures how much focus to give to each token.
- A higher score means more attention.
- A lower score means less attention.

As a result, each token computes a score with every other token, forming a 2D matrix where rows represent query tokens and columns represent key tokens.

Since the sentence "I love teaching AI" contains 4 tokens, the attention scores form a 4 × 4 matrix.

Each row corresponds to a query token, and each column corresponds to a key token.

Each value in this matrix indicates how much one token should attend to another before normalization.

Example attention score matrix (before masking):

```markdown
            I     love   teaching    AI
          --------------------------------
I         | 2.0 | 1.0 |   3.0   | 0.5 |
love      | 1.5 | 2.5 |   0.5   | 1.0 |
teaching  | 0.2 | 1.0 |   2.0   | 1.5 |
AI        | 0.1 | 0.3 |   0.7   | 2.5 |
```

Ideally, masking is applied before softmax. Let's see what happens if we skip masking and directly apply softmax.

#### After Softmax: Attention Weights

```markdown
            I     love   teaching    AI
          --------------------------------
I         | 0.23 | 0.09 |  0.63 | 0.05 |
love      | 0.21 | 0.58 |  0.08 | 0.13 |
teaching  | 0.08 | 0.17 |  0.47 | 0.28 |
AI        | 0.07 | 0.08 |  0.12 | 0.73 |
```

At this stage, every token can attend to every other token, including future ones.

- While predicting "love", the model attends not only to past tokens but also to future tokens: it assigns an attention weight of 0.08 to "teaching" and 0.13 to "AI", which indicates information leakage.
- While predicting "teaching", the model already attends to the future token "AI" with an attention weight of 0.28, even though "AI" should not be visible at this stage.

Because of this issue, masking is required, and for that we use a causal mask matrix.

### The Causal Mask Matrix

We create a mask matrix of the same size. This matrix defines which positions are allowed (1) and which are blocked (0).

```markdown
            I   love  teaching   AI
          --------------------------
I         | 1 |  0  |    0    | 0 |
love      | 1 |  1  |    0    | 0 |
teaching  | 1 |  1  |    1    | 0 |
AI        | 1 |  1  |    1    | 1 |
```

This is our mask matrix, and we will apply it to the score matrix to block future tokens.

Keep this mask matrix in mind as you read further, as we will use it to mask the attention score matrix.

- 1 → attention allowed
- 0 → attention blocked (future)

After applying the mask, the blocked positions are set to −∞.

Causal masking blocks access to future tokens by assigning −∞ to those positions.

In some time, we will understand why −∞ is used.

Example attention score matrix (before masking):

```markdown
            I     love   teaching    AI
          --------------------------------
I         | 2.0 | 1.0 |   3.0   | 0.5 |
love      | 1.5 | 2.5 |   0.5   | 1.0 |
teaching  | 0.2 | 1.0 |   2.0   | 1.5 |
AI        | 0.1 | 0.3 |   0.7   | 2.5 |
```

Masked attention score matrix (after masking using our mask matrix):

```markdown
            I     love   teaching    AI
          --------------------------------
I         | 2.0 | -∞  |   -∞    | -∞  |
love      | 1.5 | 2.5 |   -∞    | -∞  |
teaching  | 0.2 | 1.0 |   2.0   | -∞  |
AI        | 0.1 | 0.3 |   0.7   | 2.5 |
```

When the mask is applied, positions marked 0 in the mask matrix are blocked by setting their attention scores to −∞, while positions marked 1 retain their original values in the attention score matrix.

Why −∞? As we have to apply softmax, and because of how softmax works mathematically, −∞ becomes exactly zero after softmax. If we used zero instead, softmax would still assign a non-zero probability, allowing the model to partially attend to future tokens and defeating the purpose of causal masking.

#### After Softmax: Attention Weights

```markdown
            I     love   teaching    AI
          --------------------------------
I         | 1.00 | 0.00 |  0.00 | 0.00 |
love      | 0.27 | 0.73 |  0.00 | 0.00 |
teaching  | 0.11 | 0.24 |  0.65 | 0.00 |
AI        | 0.07 | 0.08 |  0.12 | 0.73 |
```

Now, the future tokens receive zero attention:

- While predicting "love", the model attends to "I" with an attention weight of 0.27 and to itself ("love") with 0.73. It does not attend to the future tokens "teaching" and "AI", as both receive 0.00 attention.
- While predicting "teaching", the model attends to "I" (0.11) and "love" (0.24), and to itself ("teaching") with 0.65. It does not attend to the future token "AI", which has an attention weight of 0.00.
- While predicting "AI", the model attends to "I" (0.07), "love" (0.08), and "teaching" (0.12), and to itself ("AI") with 0.73.

This is how causal masking prevents the model from seeing future tokens.

That's it for now.

Thanks

Amit Shekhar Founder @ [Outcome School](https://outcomeschool.com/)

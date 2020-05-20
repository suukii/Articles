# 什么是上下文无关语法(CFG, Context Free Grammar)


从名字可以看出，既然上下文无关语法(Context Free Grammar, 以下简称 CFG)被称为“语法”，说明它就是一套规则，是用来规定语句是如何生成的。

就像英语语法一样，比如英语语法规定了一个英语句子的构成方式可以是“名词+动词+名词”这种形式，那么根据英语语法：

- `I drink milk` 是一个符合语法的句子
- `Tom drinks cat` 看起来也是一个符合语法的句子

但其实后者并不是正确的英语句子，为什么呢？因为英语语法并不是 CFG，英语是依赖上下文的，一个句子是否正确不仅要看它的语法构成是否正确，还要从上下文来看它是否能被正确理解，`Tom drinks cat` 很明显不是一个可以被正常理解的句子，所以，虽然它符合“动词+动词+名词”结构，但它并不是一个正确的英语句子。

而 CFG 不同于英语语法，它完全不考虑上下文，只要一个句子符合语法规则，那它就是一个有效的句子。假如在一个平行世界中，英语语法是一种 CFG 的话，那 `Tom drinks cat` 也是一个正确的句子了。

虽然 CFG 不能用来描述自然语言(因为自然语言一般都需要考虑上下文)，但我们还是可以用 CFG 来描述自然语言的语法和句子结构的，或许，我们可以把英语语法想象成 `CFG + 上下文`，在不考虑上下文的情况下(也就是不管语句是否意思通顺，只要语句结构符合语法规定它就是正确的)，我们完全可以使用 CFG 来描述英语语法。


## CFG 的组成

首先来熟悉一下几个相关词汇：

- Terminals(终结符): 是构成最终语句的成分，比如在英语中，句子由单词组成，所以终结符就是一个个单词。
- Non Terminals(非终结符)：也被称为变量，可以认为非终结符是终结符或其他非终结符的占位符。
- Start Symbol(开始变量)：也是一个非终结符，由于是语法规则的开始，所以也被称为开始变量。
- Production Rules(规则/产生式)：语法规则，用来描述一个句子是怎样产生的。


## 来一个英语栗子

### 定义语法规则

让我们试着用 CFG 来定义一个英语句子的语法：

`Sentence --> NounPhrase VerbPhrase`

上式表示：我们定义了一个规则来规定一个 Sentence 应该长什么样，根据这个规则，要生成 `Sentence`，我们需要把 `NounPhrase` 和 `VerbPhrase` 合在一起。

解析：`Sentence --> NounPhrase VerbPhrase` 是一个产生式，其中 `Sentence`, `NounPhrase`, `VerbPhrase` 是变量，它们都是 CFG 中的非终结符，其中 `Sentence` 是开始变量。

上面的产生式描述了一个句子的结构，但根据这个语法还不能生成句子，因为我们只知道 Sentence 由 NounPhrase 和 VerbPhrase 组成，那 NounPhrase 和 VerbPhrase 又是什么呢？我们需要继续定义它们。

```
NounPhrase --> Adj Noun
NounPhrase --> Adj NounPhrase
```

上面两个产生式定义了生成 NounPhrase 的两种方法，Adj + Noun，或者 Adj + NounPhrase，这两条规则还可以合写成一条：

`NounPhrase --> Adj Noun | Adj NounPhrase`。

而 VerbPhrase 的生成规则可以定义为：

`VerbPhrase --> Verb NounPhrase`

注意：到这一步我们得到的仍然都是非终结符，上述产生式中的 Adj, Noun, Verb 也都是变量，我们还需要继续定义它们。

```
Adj  --> pink | Pretty
Noun --> girls | skirts
Verb --> like
```

终于，出现了组成句子的真正成分：单词。因为单词可以用来组成句子，所以不需要继续定义，语法定义的工作到这里就结束了。在 CFG 中，这些不需要再展开定义的成分被称为终结符(Terminals)，终结符就是构成最终句子的成分。

现在我们完整的语法规则是这样的：

```
Sentence --> NounPhrase VerbPhrase
NounPhrase --> Adj Noun | Adj NounPhrase
VerbPhrase --> Verb NounPhrase
Adj  --> pink | Pretty
Noun --> girls | skirts
Verb --> like
```

### 根据语法规则创造句子

现在让我们根据这些语法来生成一个句子：

1. 要生成 Sentence，需要 NounPhrase + VerbPhrase

2. 要生成 NounPhrase，需要 Adj + Noun 或者 Adj + NounPhrase，这里我们选择 Adj + Noun
3. 要生成 Adj，我们可以使用 pink 或者 Pretty 这个单词，这里我们选择 Pretty
4. 要生成 Noun，我们可以使用 girls 或者 skirt，这里我们选择 girls

5. 要生成 VerbPhrase，需要 Verb + NounPhrase
6. 要生成 Verb，我们可以使用 like 这个单词
7. 要生成 NounPhrase，需要 Adj + Noun 或者 Adj + NounPhrase，这里我们选择 Adj + Noun
8. 要生成 Adj，我们可以使用 pink 或者 Pretty 这个单词，这里我们选择 pink
9. 要生成 Noun，我们可以使用 girls 或者 skirt，这里我们选择 skirts

P.S. 上述步骤中所有选择都是随机的。

这些步骤用句子描述太繁琐了，我们还是来看一张图吧，基本上，这就是一个递归操作，用树结构来表示，就清晰多了。

![https://github.com/suukii/Articles/blob/master/assets/CFG_1.png](https://github.com/suukii/Articles/blob/master/assets/CFG_1.png)

把树的所有叶子节点，也就是所有终结符连起来，我们就生成了一个句子 `Pretty girls like pink skirts` (BTW, it seems to rhyme!)。这个树一般叫做解析树(Parse Tree)，我们还可以创建另一棵树来生成另一个句子，步骤是一样的，只是其中的选择不同，不同的选择会生成不同的结果，比如 `Pretty skirts like pink girls`。


## JavaScript 栗子

英语语法并不是真正的 CFG，因为英语必须依赖上下文，CFG 一般用来描述编程语言，比如 JS，实际上，ECMAScript 规范就是使用 CFG 来定义 JavaScript 这门语言的。

我们来看一个简单的 JS 语法生成式，下面是 ECMAScript 规范对 `NumericLiteral` 的定义，如果我们要生成一个 `NumericLiteral`，我们就得遵循这个语法规则。

```
NumericLiteral::
DecimalLiteral
DecimalBigIntegerLiteral
NonDecimalIntegerLiteral
NonDecimalIntegerLiteral BigIntLiteralSuffix
```

转换成上文用到的语法的话，规则写成这样：

`NumericLiteral --> DecimalLiteral | DecimalBigIntegerLiteral | NonDecimalIntegerLiteral | NonDecimalIntegerLiteral BigIntLiteralSuffix`

ECMAScript 规范用到的语法稍有不同，不过本质上是一样的。

首先规范定义了 `NumericLiteral` 可以有 4 种生成方式。我们可以选择一种，这里就选第一种吧，然后我们发现了 `DecimalLiteral` 也是一个非终结符，我们得接着找对它的定义。

```
DecimalLiteral::
DecimalIntegerLiteral . DecimalDigits(opt) ExponentPart(opt)
. DecimalDigits ExponentPart(opt)
DecimalIntegerLiteral ExponentPart(opt)
```

规范中定义了 `NumericLiteral` 可以有 3 种生成方式，我们选择一种 `DecimalIntegerLiteral . DecimalDigits(opt) ExponentPart(opt)`，因为后面的部分都是可选的(opt)，我们就简单点，只考虑第一部分 `DecimalIntegerLiteral` 吧，接着去找 `DecimalIntegerLiteral` 的定义：

```
DecimalIntegerLiteral::
0
NonZeroDigit DecimalDigits(opt)
```

规范中定义了 `DecimalIntegerLiteral` 可以有 2 种生成方式，如果我们选第一种 `0`，因为 `0` 是一个终结符，我们的生成过程就到此结束了，我们生成了一个 `NumericLiteral`，就是 `0`，用树结构来表示生成过程的话就是下图。

![https://github.com/suukii/Articles/blob/master/assets/CFG_2.png](https://github.com/suukii/Articles/blob/master/assets/CFG_2.png)

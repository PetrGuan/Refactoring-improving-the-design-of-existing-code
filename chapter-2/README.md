# Chapter 2 重构的法则


上一章节的例子应该大体上让你明白了什么是重构。现在是时候让我们来回顾一下关于重构的那些法则了。

## 定义什么是重构

就像其他软件开发的术语一样，“重构”这个词也经常会被不准确地使用。“重构”可以被用作名词和动词，定义如下：

> **Refactoring (noun)**: 是对软件内部结构的一种改变，这种改变可以使代码更容易被理解和更改，并且不能改变软件对外的行为。

> **Refactoring (verb)**:对软件进行一系列的重构（名词），并且不改变软件对外的行为。

在过去的几十年间，许多业界人员都会错误地使用“重构”这个词，他们认为任何对代码的清理都能被称为是“重构”。事实上重构是由一系列小的重构组成的，重要的是每次你的重构都足够的小，因为足够小，所以你的软件还是能够正常运行并且很容易进行，这些小的步骤最后会成为一个大的重构，

> 如果有人和你说他把代码重构坏了，那么你应该很确定他不是在重构。

我使用 “restructuring” 这个词来指代对 code base 的任何 reorganizing 或者 cleaning up ，并且把重构看作是 “restructuring” 的一种。第一次见我重构的人可能会认为这样效率很低，我进行很多很小的步骤来重构，而不是一次性修改很大一部分。但最终正是这些小步骤让我能够更快地进行下去，因为我根本不用花费很多时间去 debugging。

在我的定义里，我提到了“软件对外的行为”，这里的意思简单来说就是在重构完之后程序应该和重构完之前有一样的功能。并不是说程序内部运行和重构之前一模一样，比如调用栈，性能等等。这里的功能应该是指用户所真正关心的功能。

重构和性能优化非常相似，因为它们都不应该改变程序的功能。区别在于彼此的目的：重构总是为了“使代码更容易被理解和更改”，这可能使程序变慢但也可能变快。对性能优化而言，我只考虑怎么使程序变快，有时不可避免的会需要难以理解的代码。

## 两顶帽子

Kent Beck 对此有一个 two hats 的比喻。当你使用重构来开发软件的时候，你把时间分配在两个不同的方面：增加新功能和重构。当我增加新功能的时候，我不应该改变原有的代码；我只是在增加新的功能。我通过增加新的测试来衡量我的工作进程。当我重构的时候，我不应该增加新的功能；我只是在 restructure 我的代码。我不用增加任何的测试（除非我发现之前有遗漏）。

当我自己开发软件的时候，我发现我经常换着戴这两顶“帽子”。我刚开始是想增加新的功能，但有时候会发现如果我重新组织代码，会更容易达到我增加新功能的目的。所以我换帽子到“重构”了。一旦代码有了更好的组织结构，我再换帽子到“增加功能”，并且增加功能。一旦新功能增加好了，我可能又意识到我写的代码实在太难理解了，所以我又换帽子并且重构并且重构。整个过程可能只有十分钟，但是我总是有意识的知道我在干什么。

## 为什么要重构？

我不会说“重构可以解决软件的所有问题”，它不是 “silver bullet”（银弹）。但它是一种珍贵的控制代码的工具，并且可以或者说应该被广泛使用。

## 重构提高软件的设计

如果没有重构，软件的结构会有衰变的倾向。随着开发者为了各种短期目的改变代码（常常没有很好的理解架构），代码会渐渐地失去原有的结构，并且这会有一种累积的效果，代码越是难读懂，开发者越是会乱改，于是代码的结构衰变的越快。定期的重构有助于维护代码的结构。

设计不良的代码通常会需要更多的代码来做同一件事，经常是因为在很多地方散落了重复的代码。因此改善设计的一个重要方面就是要避免重复代码。并不是因为减少了代码数量系统会变得那么的快而重要，减少重复代码会对你以后改变代码起巨大的作用。代码越多，越是难正确地去修改，因为你要理解更多的代码。我在一个地方修改了代码，但是系统并没有按我理解的去运行，这是因为在另一个地方也有同样的代码但我却没有修改。通过避免重复代码，我能够保证某部分代码 `says everything once and only once`，这正是良好设计的核心。

## 重构使软件更容易被理解

编程从许多意义上来说，其实是一种和计算机进行的沟通。我们写代码来告诉计算机该做什么，然后计算机准确地去做，不会多做也不会少。我们及时地减少我想让计算机做什么和我告诉计算机做什么之间的差距，编程其实就是关于如何准确地把我想做的事传达给计算机。但很有可能我们的代码会被其他用户使用，或许几个月后，另一个开发者会尝试着去阅读我的代码，并做一些他需要的变化。我们经常忽视了别人可能会阅读并修改我们的代码，而这恰恰是最重要的。就算计算机要多进行几个时钟循环来编译那又怎么样呢？但如果你的代码需要另一个程序员花几周甚至几个月才能正确理解，那就真的很重要了。

问题在于当我们自己编程的时候，我们只想要实现好我们的需求，而根本不会去想未来的那个开发者。重构帮助我们使得代码可读性更强。在重构之前，我们的代码可以工作但没有很好的结构。花那么一点时间来重构可以使我们的代码更好的阐明我们真正的意图。

我并不是无私的为其他开发者来重构，其实很多情况下那个其他开发者正是我们自己。所以这就使得重构更加重要了。我个人是非常懒非常健忘的，我健忘的一个形式就是我从来不记得我之前写过的代码。事实上我刻意地不去记以前写过的代码，因为我怕我的小脑袋会爆炸。我把我要记的所有东西都放在了重构好的代码里，那样我可以少担心点我的脑细胞。

## 重构利于我发现 Bug

有助于理解代码也就同时意味着有助于发现 bug。我必须承认我不是一个特别善于发现 bug 的程序员。有的天才可以通过读一大段复杂的代码并发现其中的 bug，而我却办不到。然而我有自己的办法，我发现如果我重构代码，我会试着去很深入地了解我的代码，并且把这种理解又体现在重构的代码里。通过使程序的结构变得更加清晰，甚至是我都能发现之前根本发现不了的 bug！

这让我想起了 Kent Beck 经常提起的关于自己的一句话：“我不是个伟大的程序员，我只是个还不错的程序员多加了点伟大的习惯而已。”重构帮助我更有效地书写健壮的代码。

## 重构帮助我更快地编程

归根结底，所有之前提到的导致的最终结果：重构帮助我更快地编程。

这听起来似乎是违反直觉的。当我讨论重构的时候，人们会自然而然的明白这会提高软件质量，比如更好的内部设计，可读性，减少 bug。但当你花时间重构，你不是减慢了开发效率吗？

根据我和一些开发者接触后所得到的经验是，他们通常一开始会进行地很快，但慢慢的增加新功能会变得越来越慢。每个新的功能都需要更多的时间来理解当前的代码，理解怎么把新代码加入到老代码里。这还不算完，就算你成功加入了这个新功能，通常你都会花很多的时间来修复可能会出现的各种 bug。慢慢的，你的代码会看起来像被各种补丁所组成，要准确地理解简直像在进行考古工作。于是有的开发者就会觉得这样的开发状态还不如自己从头开始实现。

下面的图是一个很好的例子：

![image](https://github.com/byelaney/Refactoring-improving-the-design-of-existing-code/blob/master/chapter-2/img/p0049_01.jpg)

当然也有的开发者有不一样的故事。他们很快就能添加新功能因为之前的代码写的非常好：

![image](https://github.com/byelaney/Refactoring-improving-the-design-of-existing-code/blob/master/chapter-2/img/p0049_02.jpg)

这两者的区别就在于软件内部的质量。质量高的代码允许我很容易地就知道该怎么加代码，以及加在哪里来实现我要的新功能。好的模块化让我只要理解全部代码的一部分就能添加新功能。如果代码非常清晰，那么同时我引入 bug 的几率也会大大降低，就算我引入了 bug，debug 也会变得没那么困难。慢慢的，你的 code base 就会变成类似一个平台，允许各种开发者在上面添加功能。

我把这种现象称为 Design Stamina Hypothesis。通过我们对软件内部设计的努力，我们增加了软件的耐受力，这份耐受力能让我们的软件走的更快走的更远。尽管我没有严格的证明，但它和我，以及我所认识的成百上千的杰出工程师的经验是吻合的。

20 年前，约定俗成的做法是在写任何一行代码之前，就对软件进行良好的设计。但现在重构改变了一切，我们明白了我们可以一边开发一边重构，由于一开始的设计几乎不可能是非常完美的，这也越发体现了重构的重要性。

## 我们什么时候该重构？

我几乎每个小时都在重构，我总结了以下几点：

> **The Rule of Three**
>
> 以下是 Don Roberts 给我的建议: 你第一次要写一段代码的时候，你只管写就是了。你第二次要写一段代码（和第一次的代码做的事非常类似）的时候，你稍微瞟一眼之前的实现，但也不用想太多，只管写就行。你第三次碰到类似的需求时，你才进行重构。
>
> 如果你喜欢棒球：Three strikes, then you refactor。

### 预备的重构--使加入新功能变得简单

重构的最佳时机是在你需要给 code base 添加新功能之前。我是这样做的，我先查看已经存在的代码，并且尝试着如果我改变一点原来的结构，那么添加新功能会不会就变得很简单？有可能我要的功能有一个函数已经实现了，只是和我的要求有些细微的区别。如果我不去重构，我可能只是单纯的拷贝这个函数并且适当修改。

“就像我想往东前进 100 英里，但是往东需要穿过茂密的丛林。我可以先往北前进 20 英里开上高速公路，然后再以三倍于直接往东的速度往东前进。”（磨刀不误砍柴工）

同样的故事也发生在修 bug 的时候。当我发现了一个问题的 root cause，我同时发现如果我能重构一下（把 copy-paste 的代码整合，或者分离相关逻辑等等），我能真正把这个 bug 修复，整理了代码结构之后也有助于避免其他开发者陷入一样的弯路。

### 重构的理解：使代码易于理解

在我修改代码之前，我当然需要理解这段代码是干什么的。这些代码可能是我或者是别人写的，每次我试着去理解代码的时候，我都会问问自己需不需要重构一下，让它更加便于理解。我可能会查看有没有写的不够优美的条件语句，或者是对糟糕命名的函数进行重命名。

这个时候我的脑海中已经有了一定的理解了，但我没有记下所有细节的本事。Ward Cunningham 曾经说过，通过重构我把脑海里的理解放入代码本身里。之后我会跑一下测试看看我的理解对不对，如果我成功了那么这些所谓的“理解”将会保存更久并且直接被我的同事所看到。

这不仅仅是为了以后，它也帮助了我现在的工作。早期，我比较浅地理解在一下细节上，比如通过重命名一些变量我知道了它们究竟是做什么的；或者我把很冗长的函数变成了几个小的函数，代码变清晰了，之前看不到的设计也会逐渐显现。如果我不去重构它，这些我可能永远都不会领悟到，因为不是人人都是可以把代码可视化在心中的天才。Ralph Johnson 把早期的重构称为把窗户上的脏东西给擦干净，以便你能看清。当我学习代码的时候，重构帮助我的理解更上一层楼，那些对此嗤之以鼻的人可能永远都不会明白这背后的重要性。

### 重构，每次修改一点点

有时候有一种情况是，你意识到当前的代码质量一塌糊涂，逻辑复杂，函数冗余，到处是复制粘贴，你简直想把整个代码全部重写。但故事不是这样的，你肯定不会想把你的注意力从主要任务上分离开太多，但你又不想对这段代码坐视不管，这时你可以对这些代码作个记号，当你主要的任务完成后再回来修改它。

有时候，你可能得花很久很久才能修复它，但你的时间真的很宝贵。那么就每次修改一点点吧，只要保证每次都往好的方向改一点点，时间长了这段代码就自然被重构好了。

### 计划主义和机会主义的重构

以上提的都是机会主义的重构，事先并没有明确计划的，只是在加新功能或者修 bug 的时候顺便去重构，这是我的工作习惯。不管我是在加功能还是修 bug，重构都会有利于我正在进行的工作以及未来要做的工作。重构不是一项分离的工作，就像你不会把写 if 语句单独腾出来，以后再做。我一般不会事先计划好重构，大多数重构只是我在做别的事的时候顺手做的罢了。

> You have to refactor when you run into ugly code—but excellent code needs plenty of refactoring too.

单纯把重构看作修 bug 或者清理丑陋的代码是人们常犯的一个错误。丑陋的代码当然是需要重构的，但是好的代码也需要大量的重构。当我写代码的时候，我经常会决定 tradeoff，我该参数化多少函数，或者说函数应该划分在哪里？我之前做的 tradeoff 可能并不会适合我现在在做的新功能，好处在于重构过的干净的代码总是很容易让你去修改，来适应新的需求。

> “for each desired change, make the change easy (warning: this may be hard), then make the easy change”
> - Kent Beck

长时间以来，人们认为编写软件是一个渐增的过程：加新的功能，意味着我们要增加新的代码。但是好的开发人员明白，最好的做法是先把代码变得容易加新功能，然后再去加新功能。所以软件是永远不该被认为处于 “done” 这个状态的。一旦有了新的需求，软件就会改变来适应它，有时候对已有的代码作出改变比加入的新代码还要多。

当然有计划的重构并不总是不好的。如果一个团队总是忽视重构，那么未来可能要花大量的时间来对代码进行重构，才能加新功能。现在用一周时间重构甚至可能为你以后省下几个月的时间成本。大多数重构应该是机会主义的，比较小的。

我听过的一种建议是把重构和加新功能分隔开，通过不同的 commits。一个好处是它们可以被独立地 review 和 approve，然而我本人是不太认同的。太多太多情况下，重构总是和加新功能紧密联系在一起的，把它们分隔开可能反而浪费时间。分隔开了以后可能你甚至会失去重构下的 context，并且难以 review 重构的代码。每个团队要去寻找适合他们自己的方式。

### 长期的重构

大多数重构在几小时甚至几分钟内就能完成。但有时候有很大一部分代码需要重构，可能需要一个团队几周来完成。比如他们要替换现有的一个库；或者把一些代码拉出来分享给其他团队；或者是修复 build up 时候的一些错综复杂的依赖关系。

即使是以上的情况，我也不建议专门用一个团队去重构。一般来说比较好的策略是互相达成协议在工作的过程中慢慢的去重构。每当有人要改需要重构的附近的代码时，他可以稍微去重构一下那段代码。这样做的好处是每次重构完代码还是处于可以工作的状态（因为重构足够小）。如果要替换现有的库，可以先给现有的库引入一层抽象的接口，一旦负责调用库的代码使用了这些抽象，替换库就会变得很简单了。

### 在 Code Review 时重构

一些组织会经常做 code review，这样做时比较好的。Code review 有利于在整个团队中的知识共享。通过 review，经验丰富的工程师可以把这些经验传给新手，他们帮助新人更好地理解整个软件系统。Code review 对于代码的整洁性也是很重要的，你的代码可能你自己觉得很清晰，但是其他人可能不这么认为。这是不可避免的-让别人很好地了解自己是很难得。Code review 也会为提出有效的建议创造条件，比如我一周只能想出来这些办法，但是其他人可能有更好的办法，所以 review 是非常重要的。

有时候重构能帮助你 review 别人的代码。重构之前，我可能只是阅读别人的代码，有一定的理解并且提出一些建议。现在在建议之前我会先考虑是否重构之后能更容易实现，如果答案是肯定的那么我们就该重构。尝试了很多次之后我发现这样我能加深对代码的理解，提出更好的建议。

如果把重构嵌入 code review 取决于 review 的形式。通常的 pull request 模型，reviewer 单独去 review 代码可能效果没那么好。最好是代码的原作者一起参与 review，我个人的建议是和原作者 one-on-one 坐在一起，讨论代码逻辑并重构，类似于结对编程。

### 你该怎么和你的 manager 交代呢？

一个最常见的问题：“你该怎么和你的 manager 说你要花时间在重构上？” 有的 manager 或者客户可能觉得重构是一个不好的词，意味着你要纠正代码里存在的错误，或者做一些没有产出的工作。当你的团队专门花时间重构并且做的不是那么好的时候这种风气会更加蔓延。

对于有着技术基础以及设计理念的 manager 而言，重构的价值应该不难被理解。这样的 manager 应该密切关注团队是否花了足够的精力在重构上，因为通常人们都不注意重构，而不是重构的过了头。

当然，大多数 manager 和客户并不能理解重构的价值，那么这时候我告诉你一个办法：“别告诉他们你在重构！”

颠覆者？我不这么认为。软件工程师是专家，我们的工作是尽可能快地去生产可靠的软件，而重构正是一个不可或缺的因素，所以专业的事情让专业的人来判断。

### 什么时候不要重构？

是不是听起来我通篇都在说重构的好处，但确实有时候重构是不推荐的。

如果我看到了一段一团糟的代码，但是我不用修改它们，那么这时候我就不会去重构。那些丑陋的代码就让它丑陋去吧，把它们当成 API 就好了，只有我真正要修改它们的时候重构才有意义。

另一个场景是直接重写比重构更简单，这里的判断非常 tricky。没法轻易地判断是不是重构要花很多时间，直到你花了一些时间去理解这些代码。这里的判断需要非常多的经验和灵感，所以我不能轻易地给出建议。


## 重构会带来的问题

每当有人鼓吹一项技术，工具，或者一种架构的时候，我总是会去寻找其中的问题。生活中很少有完全是蓝天白云的美丽世界，你必须理解其中的 tradeoff 从而有效地去使用它们。我确实认为重构是一种非常有价值的技术，但它也存在黑暗面，下来我们来讨论一下重构的问题和如何应对。

### 减慢新功能的加入

尽管我之前的章节说了重构会加速，但有的时候重构也确实减慢了新功能的加入，尤其是对做很多重构的人来说。

> The whole purpose of refactoring is to make us program faster, producing more value with less effort.

我们经常会面对这样的 tradeoff。你会陷入一种境地，一大段代码需要被重构，但你要加的功能非常简单非常小，所以你更偏向直接添加并且不去管那一大段代码。这需要你作为职业开发者的判断，能帮助你的可能只有经验和天赋了。


I’m very conscious that preparatory refactoring often makes a change easier, so I certainly will do it if I see that it makes my new feature easier to implement. I’m also more inclined to refactor if this is a problem I’ve seen before—sometimes it takes me a couple of times seeing some particular ugliness before I decide to refactor it away. Conversely, I’m more likely to not refactor if it’s part of the code I rarely touch and the cost of the inconvenience isn’t something I feel very often. Sometimes, I delay a refactoring because I’m not sure what improvement to do, although at other times I’ll try something as an experiment to see if it makes things better.

Still, the evidence I hear from my colleagues in the industry is that too little refactoring is far more prevalent than too much. In other words, most people should try to refactor more often. You may have trouble telling the difference in productivity between a healthy and a sickly code base because you haven’t had enough experience of a healthy code base—of the power that comes from easily combining existing parts into new configurations to quickly enable complicated new features.

Although it’s often managers that are criticized for the counter-productive habit of squelching refactoring in the name of speed, I’ve often seen developers do it to themselves. Sometimes, they think they shouldn’t be refactoring even though their leadership is actually in favor. If you’re a tech lead in a team, it’s important to show team members that you value improving the health of a code base. That judgment I mentioned earlier on whether to refactor or not is something that takes years of experience to build up. Those with less experience in refactoring need lots of mentoring to accelerate them through the process.

But I think the most dangerous way that people get trapped is when they try to justify refactoring in terms of “clean code,” “good engineering practice,” or similar moral reasons. The point of refactoring isn’t to show how sparkly a code base is—it is purely economic. We refactor because it makes us faster—faster to add features, faster to fix bugs. It’s important to keep that in front of your mind and in front of communication with others. The economic benefits of refactoring should always be the driving factor, and the more that is understood by developers, managers, and customers, the more of the “good design” curve we’ll see.

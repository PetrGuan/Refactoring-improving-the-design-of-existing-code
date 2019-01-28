# Chapter-1

假设现在我们有两个 json 文件，一个是 **plays.json**

```json
{
  "hamlet": {"name": "Hamlet", "type": "tragedy"},
  "as-like": {"name": "As You Like It", "type": "comedy"},
  "othello": {"name": "Othello", "type": "tragedy"}
}

```

另一个是 **invoices.json**,

```json
[
  {
    "customer": "BigCo",
    "performances": [
      {
        "playID": "hamlet",
        "audience": 55
      },
      {
        "playID": "as-like",
        "audience": 35
      },
      {
        "playID": "othello",
        "audience": 40
      }
    ]
  }
]
```

下来我们看现有的代码，用来打印出账单：

```javascript
function statement (invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `Statement for ${invoice.customer}\n`;
  const format = new Intl.NumberFormat("en-US",
                        { style: "currency", currency: "USD",
                          minimumFractionDigits: 2 }).format;
  for (let perf of invoice.performances) {
    const play = plays[perf.playID];
    let thisAmount = 0;

    switch (play.type) {
    case "tragedy":
      thisAmount = 40000;
      if (perf.audience > 30) {
        thisAmount += 1000 * (perf.audience - 30);
      }
      break;
    case "comedy":
      thisAmount = 30000;
      if (perf.audience > 20) {
        thisAmount += 10000 + 500 * (perf.audience - 20);
      }
      thisAmount += 300 * perf.audience;
      break;
    default:
        throw new Error(`unknown type: ${play.type}`);
    }

    // add volume credits
    volumeCredits += Math.max(perf.audience - 30, 0);
    // add extra credit for every ten comedy attendees
    if ("comedy" === play.type) volumeCredits += Math.floor(perf.audience / 5);

    //print line for this order
    result += `  ${play.name}: ${format(thisAmount/100)} (${perf.audience} seats)\n`;
    totalAmount += thisAmount;
  }
  result += `Amount owed is ${format(totalAmount/100)}\n`;
  result += `You earned ${volumeCredits} credits\n`;
  return result;
}
```

以上代码会输出：

```
Statement for BigCo
  Hamlet: $650.00 (55 seats)
  As You Like It: $580.00 (35 seats)
  Othello: $500.00 (40 seats)
Amount owed is $1,730.00
You earned 47 credits
```

## COMMENTS ON THE STARTING PROGRAM

如果单就这一段代码而言，由于它比较短所以可能没有那么大必要需要重构，但如果它是大项目中的一个部分，这样的代码就可能没有那么容易理解了。我们称之为 “ugly code”，为什么呢？由于代码不容易理解，在后来的开发者需要修改这段代码或者添加新功能的时候，他会不知道该修改哪里，即使改了，也会不清楚这样的修改是否会影响其他的部分，于是 bug 很容易就会被引入。

> When you have to add a feature to a program but the code is not structured in a convenient way, first refactor the program to make it easy to add the feature, then add the feature.

首先要避免的就是简单的 “copy-paste”，如果是不需要改变的程序那么这样做无可厚非，但如果是经常要改动更新的程序，这样做无疑是在挖坑。试想如果你复制了一段代码三次，之后的改动需要改动这段代码中的一个地方，那么你需要改三次地方才能保证一致性，即使有 IDE 的帮助你也会有可能漏掉修改一处，从而引入难以排查的 bug。

## THE FIRST STEP IN REFACTORING

第一步的重要性可能没有那么明显，但它很重要，那就是准备你的 test，用来保证你的重构没有破坏原有程序的正确性。由于程序输出的是 string，那么就应该构造一系列测试的输入和对应的 string 输出。

> Before you start refactoring, make sure you have a solid suite of tests. These tests must be self-checking.

## DECOMPOSING THE *STATEMENT* FUNCTION

首先来看这一段 switch 语句：

```javascript
switch (play.type) {
    case "tragedy":
      thisAmount = 40000;
      if (perf.audience > 30) {
        thisAmount += 1000 * (perf.audience - 30);
      }
      break;
    case "comedy":
      thisAmount = 30000;
      if (perf.audience > 20) {
        thisAmount += 10000 + 500 * (perf.audience - 20);
      }
      thisAmount += 300 * perf.audience;
      break;
    default:
        throw new Error(`unknown type: ${play.type}`);
    }
```

很明显，这段代码在计算每个演出的费用，好的代码应该是不言自明的，我不用仔细看就知道它在做什么，于是很自然的我们应该把这一段代码放在一个函数里，并以它的作用来命名，比如 "amountFor(aPerformance)"。这被称为 "**Extract Function (106)**"。

首先，我需要先查看这段代码里的变量，确定如果函数化后哪些会脱离作用域，在这个例子里是："**perf, play, thisAmount**"  这三个变量，前两个只是使用了它们的值而没有改变，所以我可以直接当作函数变量传进去，“**thisAmount**"” 变量改变了，由于只有这一个变量改变了自身，所以我们可以当作函数返回值来返回。具体如下：

```javascript
function amountFor(perf, play) {
  let thisAmount = 0;
  switch (play.type) {
  case "tragedy":
    thisAmount = 40000;
    if (perf.audience > 30) {
      thisAmount += 1000 * (perf.audience - 30);
    }
    break;
  case "comedy":
    thisAmount = 30000;
    if (perf.audience > 20) {
      thisAmount += 10000 + 500 * (perf.audience - 20);
    }
    thisAmount += 300 * perf.audience;
    break;
  default:
      throw new Error(`unknown type: ${play.type}`);
  }
  return thisAmount;
}
```

这下原有的函数可以重构为：

```javascript
function statement (invoice, plays) {
  ...
  for (let perf of invoice.performances) {
    ...
    let thisAmount = amountFor(perf, play);
    ...
  }
  ...
```

记住这点，每次在你重构以后，立即进行编译和测试，因为这会大大降低你 debug 的难度，这就是重构的核心思想：small changes and testing after each change。

> Refactoring changes the programs in small steps, so if you make a mistake, it is easy to find where the bug is.

让我们回过头来再看一下提取出来的代码，首先我们可以查看是否有变量需要重命名：

```javascript
function amountFor(perf, play) {
  let result = 0;
  switch (play.type) {
  case "tragedy":
    result = 40000;
    if (perf.audience > 30) {
      result += 1000 * (perf.audience - 30);
    }
    break;
  case "comedy":
    result = 30000;
    if (perf.audience > 20) {
      result += 10000 + 500 * (perf.audience - 20);
    }
    result += 300 * perf.audience;
    break;
  default:
      throw new Error(`unknown type: ${play.type}`);
  }
  return result;
}
```

这里我们把 **thisAmount** 改为了 **result**，接着我们把 **perf** 改为 **aPerformance**：

```javascript
function amountFor(aPerformance, play) {
  let result = 0;
  switch (play.type) {
  case "tragedy":
    result = 40000;
    if (aPerformance.audience > 30) {
      result += 1000 * (aPerformance.audience - 30);
    }
    break;
  case "comedy":
    result = 30000;
    if (aPerformance.audience > 20) {
      result += 10000 + 500 * (aPerformance.audience - 20);
    }
    result += 300 * aPerformance.audience;
    break;
  default:
      throw new Error(`unknown type: ${play.type}`);
  }
  return result;
}
```

这里由于 javascript 是动态类型的语言，把变量的类型也嵌入命名中是一个比较好的做法。

> Any fool can write code that a computer can understand. Good programmers write code that humans can understand.

## Removing the play Variable

仔细观察可以发现，play 变量是由 performance 变量计算而来，所以没有必要把 play 当作变量来传递，我们可以直接在 amountFor 函数里把它计算出来。当分解比较长的函数的时候，像 play 这样的变量应该尽量删除，这是因为临时变量会产生很多临时的作用域名称，这往往会令人困扰，这样的重构技巧被称作为 "Replace Temp with Query"。我们把这个过程作为一个函数：

```javascript
function playFor(aPerformance) {
  return plays[aPerformance.playID];
}
```

整个函数现在变为：

```javascript
function statement (invoice, plays) {
  ...
  for (let perf of invoice.performances) {
    const play = playFor(perf);
  ...
  return result;
```

接着我们使用 “**Inline Variable**”：

```javascript
function statement (invoice, plays) {
  ...
  for (let perf of invoice.performances) {
    // 删除下面这句
    // const play = playFor(perf);
    let thisAmount = amountFor(perf, playFor(perf));

    ...
    if ("comedy" === playFor(perf).type) volumeCredits += Math.floor(perf.audience / 5);

    //print line for this order
    result += `  ${playFor(perf).name}: ${format(thisAmount/100)} (${perf.audience} seats)\n`;
    totalAmount += thisAmount;
  }
  ...
  return result;
```

有了这个 “**inline**”，我们可以继续下一个重构操作，“**Change Function Declaration**”，来移除 “**play**” 变量：

```javascript
function amountFor(aPerformance) {
  let result = 0;
  switch (playFor(aPerformance).type) {
  case "tragedy":
    result = 40000;
    if (aPerformance.audience > 30) {
      result += 1000 * (aPerformance.audience - 30);
    }
    break;
  case "comedy":
    result = 30000;
    if (aPerformance.audience > 20) {
      result += 10000 + 500 * (aPerformance.audience - 20);
    }
    result += 300 * aPerformance.audience;
    break;
  default:
      throw new Error(`unknown type: ${playFor(aPerformance).type}`);
  }
  return result;
}

function statement (invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `Statement for ${invoice.customer}\n`;
  const format = new Intl.NumberFormat("en-US",
                        { style: "currency", currency: "USD",
                          minimumFractionDigits: 2 }).format;
  for (let perf of invoice.performances) {
    let thisAmount = amountFor(perf, playFor(perf));

    // add volume credits
    volumeCredits += Math.max(perf.audience - 30, 0);
    // add extra credit for every ten comedy attendees
    if ("comedy" === playFor(perf).type) volumeCredits += Math.floor(perf.audience / 5);

    //print line for this order
    result += `  ${playFor(perf).name}: ${format(thisAmount/100)} (${perf.audience} seats)\n`;
    totalAmount += thisAmount;
  }
  result += `Amount owed is ${format(totalAmount/100)}\n`;
  result += `You earned ${volumeCredits} credits\n`;
  return result;
```

有些程序员会很惊讶，因为之前的代码 "**playFor()**" 在每个循环里只执行一次，而现在要执行三次。在这里我们明白这不会显著影响程序的执行效率，但即使如此我们也要在心中有意识程序的效率。

下来我们看 "**playFor()**" 被调用的地方，很明显我们可以移除 `let thisAmount = amountFor(perf, playFor(perf));`

```javascript
function statement (invoice, plays) {
  ...
  for (let perf of invoice.performances) {
    // 删除下一句
    // let thisAmount = amountFor(perf, playFor(perf));
    ...
    //print line for this order
    result += `  ${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience} seats)\n`;
    totalAmount += amountFor(perf);
  }
  ...
  return result;
```

## Extracting Volume Credits

我们可以把 **volumeCredits** 提取出来，作为一个函数：

```javascript
function volumeCreditsFor(perf) {
  let result = 0;
  result += Math.max(aPerformance.audience - 30, 0);
  if ("comedy" === playFor(aPerformance).type) result += Math.floor(aPerformance.audience / 5);
  return result;
}
```

这时我们的 **statement** 函数可以重构为：

```javascript
function statement (invoice, plays) {
  ...
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
    ...
  }
  ...
```

## Removing the format Variable

临时的变量可能会成为问题，因为它们只在各自的作用域内有效，所以往往会使得作用域变得很长很复杂。所以我们的下一步可以把 **format** 变量用函数来替代。

```javascript
function format(aNumber) {
  return new Intl.NumberFormat("en-US",
                      { style: "currency", currency: "USD",
                        minimumFractionDigits: 2 }).format(aNumber);}
```

我们的函数可以重构为：

```javascript
function statement (invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `Statement for ${invoice.customer}\n`;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);

    //print line for this order
    result += `  ${playFor(perf).name}: ${format(amountFor(perf)/100)} (${perf.audience} seats)\n`;
    totalAmount += amountFor(perf);

  }
  result += `Amount owed is ${format(totalAmount/100)}\n`;
  result += `You earned ${volumeCredits} credits\n`;
  return result;
```

这里要注意的是函数的名字，你可以发现单纯看 **format** 函数名字，你无法了解它在 **format** 什么，所以更好的做法是使用 “**Change Function Declaration**”。

```javascript
function usd(aNumber) {
  return new Intl.NumberFormat("en-US",
                      { style: "currency", currency: "USD",
                        minimumFractionDigits: 2 }).format(aNumber/100);
}

function statement (invoice, plays) {
  let totalAmount = 0;
  let volumeCredits = 0;
  let result = `Statement for ${invoice.customer}\n`;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);

    //print line for this order
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${perf.audience} seats)\n`;
    totalAmount += amountFor(perf);

  }
  result += `Amount owed is ${usd(totalAmount)}\n`;
  result += `You earned ${volumeCredits} credits\n`;
  return result;
```

## Removing Total Volume Credits

我们下一个重构的对象是 **volumeCredits**。我们的一个动作就是，使用 “**Split Loop**” 来分隔开 **volumeCredits**,以及 “**Slide Statements**”。

```javascript
function statement (invoice, plays) {
  let totalAmount = 0;

  let result = `Statement for ${invoice.customer}\n`;

  for (let perf of invoice.performances) {

    //print line for this order
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${perf.audience} seats)\n`;
    totalAmount += amountFor(perf);

  }
  let volumeCredits = 0;
  for (let perf of invoice.performances) {
    volumeCredits += volumeCreditsFor(perf);
  }

  result += `Amount owed is ${usd(totalAmount)}\n`;
  result += `You earned ${volumeCredits} credits\n`;
  return result;
```

之后再把对 **volumeCredits** 的计算提取到一个函数里：

```javascript
function totalVolumeCredits() {
  let result = 0;
  for (let perf of invoice.performances) {
    result += volumeCreditsFor(perf);
  }
  return result;
}
```

这时候函数的调用就可以重构为：

```javascript
function statement (invoice, plays) {
  let totalAmount = 0;
  let result = `Statement for ${invoice.customer}\n`;
  for (let perf of invoice.performances) {

    //print line for this order
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${perf.audience} seats)\n`;
    totalAmount += amountFor(perf);
  }

  result += `Amount owed is ${usd(totalAmount)}\n`;
  result += `You earned ${totalVolumeCredits()} credits\n`;
  return result;
```

这里我们的一个理念是这样的，虽然有时候重构会造成程序的效率问题，但是大多数情况下几乎是可以忽略不计的，如果真的严重影响了效率，那么我们通常的做法是先重构，再去对重构完的代码提升效率，因为重构完的代码清晰可读，往往会更容易让你提升效率。以下是用到的重构技巧：

> * **Split Loop** (226) to isolate the accumulation

> * **Slide Statements** (221) to bring the initializing code next to the accumulation

> * **Extract Function** (106) to create a function for calculating the total

> * **Inline Variable** (123) to remove the variable completely

同样的技巧，把 “**totalAmount**” 提取成函数：

```javascript
function totalAmount() {
  let result = 0;
  for (let perf of invoice.performances) {
    result += amountFor(perf);
  }
  return result;
}

function statement (invoice, plays) {
  let result = `Statement for ${invoice.customer}\n`;
  for (let perf of invoice.performances) {
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))}    (${perf.audience} seats)\n`;
  }
  result += `Amount owed is ${usd(totalAmount())}\n`;
  result += `You earned ${totalVolumeCredits()} credits\n`;
  return result;
```

## STATUS: LOTS OF NESTED FUNCTIONS

现在我们来回顾一下我们重构的结果：

```javascript
  function statement (invoice, plays) {
    let result = `Statement for ${invoice.customer}\n`;
    for (let perf of invoice.performances) {
      result += `  ${playFor(perf).name}: ${usd(amountFor(perf))}    (${perf.audience} seats)\n`;
    }
    result += `Amount owed is ${usd(totalAmount())}\n`;
    result += `You earned ${totalVolumeCredits()} credits\n`;
    return result;

  function totalAmount() {
    let result = 0;
    for (let perf of invoice.performances) {
      result += amountFor(perf);
    }
    return result;
  }

  function totalVolumeCredits() {
    let result = 0;
    for (let perf of invoice.performances) {
      result += volumeCreditsFor(perf);
    }
    return result;
  }

  function usd(aNumber) {
    return new Intl.NumberFormat("en-US",
                        { style: "currency", currency: "USD",
                          minimumFractionDigits: 2 }).format(aNumber/100);
  }

  function volumeCreditsFor(aPerformance) {
    let result = 0;
    result += Math.max(aPerformance.audience - 30, 0);
    if ("comedy" === playFor(aPerformance).type) result += Math.floor(aPerformance.audience / 5);
    return result;
  }

  function playFor(aPerformance) {
    return plays[aPerformance.playID];
  }

  function amountFor(aPerformance) {
    let result = 0;
    switch (playFor(aPerformance).type) {
    case "tragedy":
      result = 40000;
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case "comedy":
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
        throw new Error(`unknown type: ${playFor(aPerformance).type}`);
    }
    return result;
  }
}
```

现在代码整体的结构比之前好了很多。起初的 top-level “**statement**” 函数现在只有 **7** 行，并且每行都在进行 print 相关操作，所有具体的计算逻辑已经被移到了其他的函数中，这样使得你只要专注于一个个琐碎的函数，一切都变得更简单以及容易理解。


## SPLITTING THE PHASES OF CALCULATION AND FORMATTING

目前为止，我们的重构侧重于给函数更为合理的结构，以便于理解程序逻辑。一般而言早期的重构都侧重于此，把复杂的代码块分解，并且更好地重命名。之后我们可以专注于功能性方面的变化，为 “**statement**” 提供 **HTML** 的版本。在进行了之前的重构之后，事情会变得更简单，由于计算相关的代码都分散开来了，我们只需要实现顶端 **7** 行代码的 **HTML** 版本即可。

我们使用的重构技巧叫做 “**Split Phase (154)**”，目的是把逻辑分割成两个部分：第一部分负责计算 statement 需要的数据，第二部分负责把文本渲染成 HTML。

首先使用 “**Extract Function (106)**”：

```javascript
function statement (invoice, plays) {
  return renderPlainText(invoice, plays);
}


function renderPlainText(invoice, plays) {
  let result = `Statement for ${invoice.customer}\n`;
  for (let perf of invoice.performances) {
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${perf.audience} seats)\n`;
  }
  result += `Amount owed is ${usd(totalAmount())}\n`;
  result += `You earned ${totalVolumeCredits()} credits\n`;
  return result;

  function totalAmount() {…}
  function totalVolumeCredits() {…}
  function usd(aNumber) {…}
  function volumeCreditsFor(aPerformance) {…}
  function playFor(aPerformance) {…}
  function amountFor(aPerformance) {…}
}
```

和以往一样进行 compile-test-commit，然后创建一个对象作为中间数据结构，把这个对象作为变量传给 “**renderPlainText**”：

```javascript
function statement (invoice, plays) {
  const statementData = {};
  return renderPlainText(statementData, invoice, plays);
}

function renderPlainText(data, invoice, plays) {
  let result = `Statement for ${invoice.customer}\n`;
  for (let perf of invoice.performances) {
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${perf.audience} seats)\n`;
  }
  result += `Amount owed is ${usd(totalAmount())}\n`;
  result += `You earned ${totalVolumeCredits()} credits\n`;
  return result;

  function totalAmount() {…}
  function totalVolumeCredits() {…}
  function usd(aNumber) {…}
  function volumeCreditsFor(aPerformance) {…}
  function playFor(aPerformance) {…}
  function amountFor(aPerformance) {…}
}
```

之后我们检查 “**renderPlainText**” 用到的其他变量，最后想要达到的目的是 renderPlainText 只负责对传入的数据进行渲染。第一步是把 customer 加入中间变量：

```javascript
function statement (invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  return renderPlainText(statementData, invoice, plays);
}

function renderPlainText(data, invoice, plays) {
  let result = `Statement for ${data.customer}\n`;
  for (let perf of invoice.performances) {
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${perf.audience} seats)\n`;
  }
  result += `Amount owed is ${usd(totalAmount())}\n`;
  result += `You earned ${totalVolumeCredits()} credits\n`;
  return result;
}
```

同样的，把 performances 也加入到中间变量，之后我就可以把 invoice 变量给删除了。

```javascript
function statement (invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances;
  return renderPlainText(statementData, plays);
}

function renderPlainText(data, plays) {
  let result = `Statement for ${data.customer}\n`;
  for (let perf of data.performances) {
    result += `  ${playFor(perf).name}: ${usd(amountFor(perf))} (${perf.audience} seats)\n`;
  }
  result += `Amount owed is ${usd(totalAmount())}\n`;
  result += `You earned ${totalVolumeCredits()} credits\n`;
  return result;
}
```

下来我想要让 play name 来自于中间数据，这么做我需要 play 的数据来 enrich the performance record。

```javascript
function statement (invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  return renderPlainText(statementData, plays);
}
  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    return result;
  }

```

现在我们仅仅是复制了一个 performance object，但马上我们会添加数据，我偏爱把传入函数的变量当作不可变的。

下来我们对 playFor 和 statement 使用 “**Move Function (196)**”：

```javascript
function enrichPerformance(aPerformance) {
  const result = Object.assign({}, aPerformance);
  result.play = playFor(aPerformance);
  return result;
}

function playFor(aPerformance) {
  return plays[aPerformance.playID];
}
```

之后我把所有引用到 playFor 的地方用 data 替代。

function renderPlainText：

```javascript
let result = `Statement for ${data.customer}\n`;
 for (let perf of data.performances) {
   result += `  ${perf.play.name}: ${usd(amountFor(perf))} (${perf.audience} seats)\n`;
 }
 result += `Amount owed is ${usd(totalAmount())}\n`;
 result += `You earned ${totalVolumeCredits()} credits\n`;
 return result;

function volumeCreditsFor(aPerformance) {
  let result = 0;
  result += Math.max(aPerformance.audience - 30, 0);
  if ("comedy" === aPerformance.play.type) result += Math.floor(aPerformance.audience / 5);
  return result;
}

function amountFor(aPerformance) {
  let result = 0;
  switch (aPerformance.play.type) {
  case "tragedy":
    result = 40000;
    if (aPerformance.audience > 30) {
      result += 1000 * (aPerformance.audience - 30);
    }
    break;
  case "comedy":
    result = 30000;
    if (aPerformance.audience > 20) {
      result += 10000 + 500 * (aPerformance.audience - 20);
    }
    result += 300 * aPerformance.audience;
    break;
  default:
      throw new Error(`unknown type: ${aPerformance.play.type}`);
  }
  return result;
}
```

之后我用同样方式移动 amountFor。

function statement：

```javascript
function enrichPerformance(aPerformance) {
  const result = Object.assign({}, aPerformance);
  result.play = playFor(result);
  result.amount = amountFor(result);
  return result;
}

function amountFor(aPerformance) {…}
```

function renderPlainText：

```javascript
 let result = `Statement for ${data.customer}\n`;
 for (let perf of data.performances) {
   result += `  ${perf.play.name}: ${usd(perf.amount)} (${perf.audience} seats)\n`;
 }
 result += `Amount owed is ${usd(totalAmount())}\n`;
 result += `You earned ${totalVolumeCredits()} credits\n`;
 return result;
function totalAmount() {
  let result = 0;
  for (let perf of data.performances) {
    result += perf.amount;
  }
  return result;
}
```

之后是 volume credits 的计算。

function statement…

```javascript
function enrichPerformance(aPerformance) {
  const result = Object.assign({}, aPerformance);
  result.play = playFor(result);
  result.amount = amountFor(result);
  result.volumeCredits = volumeCreditsFor(result);
  return result;
}

  function volumeCreditsFor(aPerformance) {…}
```

function renderPlainText…

```javascript
function totalVolumeCredits() {
  let result = 0;
  for (let perf of data.performances) {
    result += perf.volumeCredits;
  }
  return result;
}
```

最后是两个 totals 的计算。

function statement…

```javascript
const statementData = {};
statementData.customer = invoice.customer;
statementData.performances = invoice.performances.map(enrichPerformance);
statementData.totalAmount = totalAmount(statementData);
statementData.totalVolumeCredits = totalVolumeCredits(statementData);
return renderPlainText(statementData, plays);


   function totalAmount(data) {…}
   function totalVolumeCredits(data) {…}
```

function renderPlainText…

```javascript
let result = `Statement for ${data.customer}\n`;
for (let perf of data.performances) {
  result += `  ${perf.play.name}: ${usd(perf.amount)} (${perf.audience} seats)\n`;
}
result += `Amount owed is ${usd(data.totalAmount)}\n`;
result += `You earned ${data.totalVolumeCredits} credits\n`;
return result;
```

尽管我可以在这几个 totals 函数里使用 statementData 变量，但我还是偏向传入更具体的变量。

之后我可以使用 “**Replace Loop with Pipeline (230)**”

function renderPlainText…

```javascript
function totalAmount(data) {
  return data.performances
    .reduce((total, p) => total + p.amount, 0);
}
function totalVolumeCredits(data) {
  return data.performances
    .reduce((total, p) => total + p.volumeCredits, 0);
}
```

top level…

```javascript
function statement (invoice, plays) {
  return renderPlainText(createStatementData(invoice, plays));
}

function createStatementData(invoice, plays) {
  const statementData = {};
  statementData.customer = invoice.customer;
  statementData.performances = invoice.performances.map(enrichPerformance);
  statementData.totalAmount = totalAmount(statementData);
  statementData.totalVolumeCredits = totalVolumeCredits(statementData);
  return statementData;
```

现在我可以把这些逻辑移到它独立的文件里了。

statement.js…

    import createStatementData from './createStatementData.js';


createStatementData.js…

```javascript
export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;


    function enrichPerformance(aPerformance) {…}
    function playFor(aPerformance) {…}
    function amountFor(aPerformance) {…}
    function volumeCreditsFor(aPerformance) {…}
    function totalAmount(data) {…}
    function totalVolumeCredits(data) {…}
```

现在很容易实现 HTML 版本。

statement.js…

```javascript
function htmlStatement (invoice, plays) {
  return renderHtml(createStatementData(invoice, plays));
}
function renderHtml (data) {
  let result = `<h1>Statement for ${data.customer}</h1>\n`;
  result += "<table>\n";
  result += "<tr><th>play</th><th>seats</th><th>cost</th></tr>";
  for (let perf of data.performances) {
    result += `  <tr><td>${perf.play.name}</td><td>${perf.audience}</td>`;
    result += `<td>${usd(perf.amount)}</td></tr>\n`;
  }
  result += "</table>\n";
  result += `<p>Amount owed is <em>${usd(data.totalAmount)}</em></p>\n`;
  result += `<p>You earned <em>${data.totalVolumeCredits}</em> credits</p>\n`;
  return result;
}

  function usd(aNumber) {…}
```

## STATUS: SEPARATED INTO TWO FILES (AND PHASES)

回顾一下我们做了什么，现在我有两个文件，

statement.js

```javascript
import createStatementData from './createStatementData.js';
function statement (invoice, plays) {
  return renderPlainText(createStatementData(invoice, plays));
}
function renderPlainText(data, plays) {
  let result = `Statement for ${data.customer}\n`;
  for (let perf of data.performances) {
    result += `  ${perf.play.name}: ${usd(perf.amount)} (${perf.audience} seats)\n`;
  }
  result += `Amount owed is ${usd(data.totalAmount)}\n`;
  result += `You earned ${data.totalVolumeCredits} credits\n`;
  return result;
}
function htmlStatement (invoice, plays) {
  return renderHtml(createStatementData(invoice, plays));
}
function renderHtml (data) {
  let result = `<h1>Statement for ${data.customer}</h1>\n`;
  result += "<table>\n";
  result += "<tr><th>play</th><th>seats</th><th>cost</th></tr>";
  for (let perf of data.performances) {
    result += `  <tr><td>${perf.play.name}</td><td>${perf.audience}</td>`;
    result += `<td>${usd(perf.amount)}</td></tr>\n`;
  }
  result += "</table>\n";
  result += `<p>Amount owed is <em>${usd(data.totalAmount)}</em></p>\n`;
  result += `<p>You earned <em>${data.totalVolumeCredits}</em> credits</p>\n`;
  return result;
}
function usd(aNumber) {
  return new Intl.NumberFormat("en-US",
                               { style: "currency", currency: "USD",
                                 minimumFractionDigits: 2 }).format(aNumber/100);
}
```

createStatementData.js

```javascript
export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    result.amount = amountFor(result);
    result.volumeCredits = volumeCreditsFor(result);
    return result;
  }
  function playFor(aPerformance) {
    return plays[aPerformance.playID]
  }
  function amountFor(aPerformance) {
    let result = 0;
    switch (aPerformance.play.type) {
    case "tragedy":
      result = 40000;
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case "comedy":
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
        throw new Error(`unknown type: ${aPerformance.play.type}`);
    }
    return result;
  }
  function volumeCreditsFor(aPerformance) {
    let result = 0;
    result += Math.max(aPerformance.audience - 30, 0);
    if ("comedy" === aPerformance.play.type) result += Math.floor(aPerformance.audience / 5);
    return result;
  }
  function totalAmount(data) {
    return data.performances
      .reduce((total, p) => total + p.amount, 0);
  }
  function totalVolumeCredits(data) {
    return data.performances
      .reduce((total, p) => total + p.volumeCredits, 0);
  }
```

重构之后我们的代码行数增加了，从 44 行 到 70 行(不包括 htmlStatement)。多出来的代码把逻辑分割成了可以辨识的很多小部分，并且把计算和布局(layout)分离了。这样的模块化可以让我们很容易理解每个小部分在做什么，简明扼要是才智智慧的灵魂，但是对不断迭代的软件而言，能够阐明自己(clarity)才是灵魂。

> When programming, follow the camping rule: Always leave the code base healthier than when you found it.


My rule is a variation on the camping rule: Always leave the code base healthier than when you found it. It will never be perfect, but it should be better.


## REORGANIZING THE CALCULATIONS BY TYPE

无须多言，这里我们主要用到的技巧叫 “**Replace Conditional with Polymorphism (272)**”

createStatementData.js…

```javascript
export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;

  function enrichPerformance(aPerformance) {
    const result = Object.assign({}, aPerformance);
    result.play = playFor(result);
    result.amount = amountFor(result);
    result.volumeCredits = volumeCreditsFor(result);
    return result;
  }
  function playFor(aPerformance) {
    return plays[aPerformance.playID]
  }
  function amountFor(aPerformance) {
    let result = 0;
    switch (aPerformance.play.type) {
    case "tragedy":
      result = 40000;
      if (aPerformance.audience > 30) {
        result += 1000 * (aPerformance.audience - 30);
      }
      break;
    case "comedy":
      result = 30000;
      if (aPerformance.audience > 20) {
        result += 10000 + 500 * (aPerformance.audience - 20);
      }
      result += 300 * aPerformance.audience;
      break;
    default:
        throw new Error(`unknown type: ${aPerformance.play.type}`);
    }
    return result;
  }
  function volumeCreditsFor(aPerformance) {
    let result = 0;
    result += Math.max(aPerformance.audience - 30, 0);
    if ("comedy" === aPerformance.play.type) result += Math.floor(aPerformance.audience / 5);
    return result;
  }
  function totalAmount(data) {
    return data.performances
      .reduce((total, p) => total + p.amount, 0);
  }
  function totalVolumeCredits(data) {
    return data.performances
      .reduce((total, p) => total + p.volumeCredits, 0);
  }
```

## Creating a Performance Calculator

我们需要一个 host 类来调用计算，这个类就叫做 performance calculator 吧。

```javascript
class PerformanceCalculator {
  constructor(aPerformance) {
    this.performance = aPerformance;
  }
}

function enrichPerformance(aPerformance) {
  const calculator = new PerformanceCalculator(aPerformance);
  const result = Object.assign({}, aPerformance);
  result.play = playFor(result);
  result.amount = amountFor(result);
  result.volumeCredits = volumeCreditsFor(result);
  return result;
}
```

先从 play 开始，

```javascript
class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }
}

function enrichPerformance(aPerformance) {
  const calculator = new PerformanceCalculator(aPerformance, playFor(aPerformance));
  const result = Object.assign({}, aPerformance);
  result.play = calculator.play;
  result.amount = amountFor(result);
  result.volumeCredits = volumeCreditsFor(result);
  return result;
}
```

## Moving Functions into the Calculator

第一步非常简单就是把逻辑完整地复制到 calculator 类里面：

```javascript
class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }

  get amount() {
  let result = 0;
  switch (this.play.type) {
    case "tragedy":
      result = 40000;
      if (this.performance.audience > 30) {
        result += 1000 * (this.performance.audience - 30);
      }
      break;
    case "comedy":
      result = 30000;
      if (this.performance.audience > 20) {
        result += 10000 + 500 * (this.performance.audience - 20);
      }
      result += 300 * this.performance.audience;
      break;
    default:
      throw new Error(`unknown type: ${this.play.type}`);
  }
  return result;
}
}
```

接下来把实际调用的函数变为一个代理函数，让我们的类来实际调用：

```javascript
function amountFor(aPerformance) {
  return new PerformanceCalculator(aPerformance, playFor(aPerformance)).amount;
}
```

现在我们的 enrichPerformance 函数是这样的：

```javascript
function enrichPerformance(aPerformance) {
  const calculator = new PerformanceCalculator(aPerformance, playFor(aPerformance));
  const result = Object.assign({}, aPerformance);
  result.play = calculator.play;
  result.amount = calculator.amount;
  result.volumeCredits = volumeCreditsFor(result);
  return result;
}
```

同样对 volumeCredits 进行重构：

```javascript
class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }

  get amount() {
    ...
  }

  get volumeCredits() {
  let result = 0;
  result += Math.max(this.performance.audience - 30, 0);
  if ("comedy" === this.play.type) result += Math.floor(this.performance.audience / 5);
  return result;
  }
}

function enrichPerformance(aPerformance) {
  const calculator = new PerformanceCalculator(aPerformance, playFor(aPerformance));
  const result = Object.assign({}, aPerformance);
  result.play = calculator.play;
  result.amount = calculator.amount;
  result.volumeCredits = calculator.volumeCredits;
  return result;
}
```

## Making the Performance Calculator Polymorphic

现在我们基本的计算逻辑都已经放在类里面了，接下来我们可以使用 "**Replace Type Code with Subclasses (362)**"。又由于 JS 的构造函数不能返回子类实例，我们使用 "**Replace Constructor with Factory Function (334)**":

```javascript
class TragedyCalculator extends PerformanceCalculator {
  // Do own logic...
}
class ComedyCalculator extends PerformanceCalculator {
  // Do own logic...
}

function createPerformanceCalculator(aPerformance, aPlay) {
    switch(aPlay.type) {
    case "tragedy": return new TragedyCalculator(aPerformance, aPlay);
    case "comedy" : return new ComedyCalculator(aPerformance, aPlay);
    default:
        throw new Error(`unknown type: ${aPlay.type}`);
    }
}

function enrichPerformance(aPerformance) {
  const calculator = createPerformanceCalculator(aPerformance, playFor(aPerformance));
  const result = Object.assign({}, aPerformance);
  result.play = calculator.play;
  result.amount = calculator.amount;
  result.volumeCredits = calculator.volumeCredits;
  return result;
}
```

接下来我们就可以把具体计算逻辑里面的条件语句给抽离到子类的计算逻辑之中：

```javascript
class TragedyCalculator extends PerformanceCalculator {
  get amount() {
  let result = 40000;
  if (this.performance.audience > 30) {
    result += 1000 * (this.performance.audience - 30);
  }
  return result;
  }
}

class ComedyCalculator extends PerformanceCalculator {
  get amount() {
  let result = 30000;
  if (this.performance.audience > 20) {
    result += 10000 + 500 * (this.performance.audience - 20);
  }
  result += 300 * this.performance.audience;
  return result;
  }
}
```

现在我可以做点事在父类 amount 里面了，那就是它应该永远不被调用！

```javascript
class PerformanceCalculator {
  get amount() {
    throw new Error('subclass responsibility');
  }
}
```

更好的理解多态的思想，把普遍的逻辑放在父类里面，变化(variations)放在子类里并且覆写父类的方法，这样的实践是比较好的。

```javascript
class PerformanceCalculator {
  get volumeCredits() {
  return Math.max(this.performance.audience - 30, 0);
  }
}

class ComedyCalculator extends PerformanceCalculator {
  get volumeCredits() {
  return super.volumeCredits + Math.floor(this.performance.audience / 5);
  }
}
```

## STATUS: CREATING THE DATA WITH THE POLYMORPHIC CALCULATOR

再次回顾我们的多态：

```javascript
export default function createStatementData(invoice, plays) {
  const result = {};
  result.customer = invoice.customer;
  result.performances = invoice.performances.map(enrichPerformance);
  result.totalAmount = totalAmount(result);
  result.totalVolumeCredits = totalVolumeCredits(result);
  return result;

  function enrichPerformance(aPerformance) {
    const calculator = createPerformanceCalculator(aPerformance, playFor(aPerformance));
    const result = Object.assign({}, aPerformance);
    result.play = calculator.play;
    result.amount = calculator.amount;
    result.volumeCredits = calculator.volumeCredits;
    return result;
  }
  function playFor(aPerformance) {
    return plays[aPerformance.playID]
  }
  function totalAmount(data) {
    return data.performances
      .reduce((total, p) => total + p.amount, 0);
  }
  function totalVolumeCredits(data) {
    return data.performances
      .reduce((total, p) => total + p.volumeCredits, 0);
  }
}

function createPerformanceCalculator(aPerformance, aPlay) {
    switch(aPlay.type) {
    case "tragedy": return new TragedyCalculator(aPerformance, aPlay);
    case "comedy" : return new ComedyCalculator(aPerformance, aPlay);
    default:
        throw new Error(`unknown type: ${aPlay.type}`);
    }
}
class PerformanceCalculator {
  constructor(aPerformance, aPlay) {
    this.performance = aPerformance;
    this.play = aPlay;
  }
  get amount() {
    throw new Error('subclass responsibility');}
  get volumeCredits() {
    return Math.max(this.performance.audience - 30, 0);
  }
}
class TragedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 40000;
    if (this.performance.audience > 30) {
      result += 1000 * (this.performance.audience - 30);
    }
    return result;
  }
}
class ComedyCalculator extends PerformanceCalculator {
  get amount() {
    let result = 30000;
    if (this.performance.audience > 20) {
      result += 10000 + 500 * (this.performance.audience - 20);
    }
    result += 300 * this.performance.audience;
    return result;
  }
  get volumeCredits() {
    return super.volumeCredits + Math.floor(this.performance.audience / 5);
  }
}
```

我们又增加了代码行数，但是我们得到的是更好的结构，这也是一种 trade-off。

## FINAL THOUGHTS

以上的例子很简单，但希望通过这个例子你能对重构有了一种大概的了解。

以上的重构可以分为三个阶段：

> * 把原来的函数分解成一系列子函数
> * 把计算的逻辑和打印的逻辑抽离
> * 最后使用多态来实现具体的计算逻辑

早期的重构都是由更好的来理解代码所驱动的，比较常见的步骤：

> * Read the code
> * Gain some insight
> * Use refactoring to move that insight from your head back into the code

整洁优雅的代码是容易理解的，容易被改变的。当你需要作出修改的时候，好的代码会让你轻易的就找到去哪里改并且不会引入错误。

> The true test of good code is how easy it is to change it.

最后，在每次微小的改动之后都进行测试，保证你在朝着正确的方向上前进！

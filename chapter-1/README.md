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

> * **Image Split Loop** (226) to isolate the accumulation

> * **Image Slide Statements** (221) to bring the initializing code next to the accumulation

> * **Image Extract Function** (106) to create a function for calculating the total

> * **Image Inline Variable** (123) to remove the variable completely

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

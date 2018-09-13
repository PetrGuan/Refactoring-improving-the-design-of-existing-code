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

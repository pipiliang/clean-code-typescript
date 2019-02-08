# TypeScript代码整洁之道
本项目是对[clean-code-typescript](https://github.com/labs42io/clean-code-typescript)项目的翻译以及精简，让国内程序员可更快速阅读和学习。

## 目录
  1. [简介](#简介)
  2. [变量](#变量)
  3. [函数](#函数)
  4. [对象与数据结构](#对象和数据结构)
  5. [类](#类)
  6. [SOLID原则](#SOLID原则)
  7. [测试](#测试)
  8. [并发](#并发)
  9. [错误处理](#错误处理)
  10. [格式化](#格式化)
  11. [注释](#注释)

## 简介

![Humorous image of software quality estimation as a count of how many expletives you shout when reading code](https://www.osnews.com/images/comics/wtfm.jpg)

这不是TypeScript编码规范，是将Robert C. Martin的[*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)应用到TypeScript上，指导如何使用TypeScript编写[易读、可重用和可重构](https://github.com/ryanmcdermott/3rs-of-software-architecture)的软件。

并不是每一个原则都必须严格遵守，得到普遍认同的原则就更少了。这些虽然只是指导，没有其他内容，但却是*Clean Code*作者多年经验的总结。

软件工程技术已经有50多年的历史了，我们仍然要学习很多的东西。当软件架构和架构本身一样古老的时候，也许我们会有更严格的规则来遵守。现在，让这些指导原则作为评估您和您的团队的JavaScript代码质量的试金石。

还有件事：理解这些原则不会立即让您成为更好的程序员，也不意味着工作多年不会犯错。每一段代码都是从不完美开始的，通过走查，我们把不完美的地方剔除，就像湿粘土最终成形一样，不要因为需要不断改进而自责!


**[⬆ 回到顶部](#目录)**

## 变量

### 变量名要有意义

做有意义的区分，让读者更容易理解变量的不同。

**反例:**

```ts

function between<T>(a1: T, a2: T, a3: T) {

  return a2 <= a1 && a1 <= a3;

}

```

**正例:**

```ts

function between<T>(value: T, left: T, right: T) {

  return left <= value && value <= right;

}

```

**[⬆ 回到顶部](#目录)**

### 变量名可拼读出来

如果你不能读出它，你在讨论它时听起来就会像个白痴。

**反例:**

```ts

class DtaRcrd102 {

  private genymdhms: Date;

  private modymdhms: Date;

  private pszqint = '102';

}

```

**正例:**

```ts

class Customer {

  private generationTimestamp: Date;

  private modificationTimestamp: Date;

  private recordId = '102';

}

```

**[⬆ 回到顶部](#目录)**

### 对功能类似的变量名采用统一的命名风格

**反例:**

```ts

function getUserInfo(): User;

function getUserDetails(): User;

function getUserData(): User;

```

**正例:**

```ts

function getUser(): User;

```

**[⬆ 回到顶部](#目录)**

### 变量名可检索

我们读代码比写的要多。写代码要是可读和可查找的，这很重要。 不取一些对理解程序有用的变量名，那就会坑阅读代码的人。要让命名可查找，像[TSLint](https://palantir.github.io/tslint/rules/no-magic-numbers/) 这样的工具可以帮助识别未命名的常量。

**反例:**

```ts

// What the heck is 86400000 for?

setTimeout(restart, 86400000);

```

**正例:**

```ts

// Declare them as capitalized named constants.

const MILLISECONDS_IN_A_DAY = 24 * 60 * 60 * 1000;

setTimeout(restart, MILLISECONDS_IN_A_DAY);

```

**[⬆ 回到顶部](#目录)**

### 使用自解释的变量名

**反例:**

```ts

declare const users:Map<string, User>;

for (const keyValue of users) {

  // iterate through users map

}

```

**正例:**

```ts

declare const users:Map<string, User>;

for (const [id, user] of users) {

  // iterate through users map

}

```

**[⬆ 回到顶部](#目录)**

### 避免思维映射，避免缩写

不要让人去猜测或想象变量的含义，*明确是王道.*

**反例:**

```ts

const u = getUser();

const s = getSubscription();

const t = charge(u, s);

```

**正例:**

```ts

const user = getUser();

const subscription = getSubscription();

const transaction = charge(user, subscription);

```

**[⬆ 回到顶部](#目录)**

### 避免重复描述

如果您的类名或对象名已经表达了某中信息，在内部变量名中不要再重复。

**反例:**

```ts

type Car = {

  carMake: string;

  carModel: string;

  carColor: string;

}

function print(car: Car): void {

  console.log(`${this.carMake} ${this.carModel} (${this.carColor})`);

}

```

**正例:**

```ts

type Car = {

  make: string;

  model: string;

  color: string;

}

function print(car: Car): void {

  console.log(`${this.make} ${this.model} (${this.color})`);

}

```

**[⬆ 回到顶部](#目录)**

### 使用默认参数，而不是短路或条件

默认参数通常比短路更干净。

**反例:**

```ts

function loadPages(count: number) {

  const loadCount = count !== undefined ? count : 10;

  // ...

}

```

**正例:**

```ts

function loadPages(count: number = 10) {

  // ...

}

```

**[⬆ 回到顶部](#目录)**



## 函数

### 参数越少越好 (理想情况不超过2个)

参数个数限制很重要，这样函数测试会更容易，超过三个参数会导致出现需要测试各种不同参数的组合场景。

One or two arguments is the ideal case, and three should be avoided if possible. Anything more than that should be consolidated.Usually, if you have more than two arguments then your function is trying to do too much.In cases where it's not, most of the time a higher-level object will suffice as an argument.  

理想情况，只有一两个参数，应该尽可能避免三个参数。通常，如果您有两个以上的参数，那么您的函数就做得太多了。若不是这样，多数情况抽象成一个高层对象作为参数更好。

Consider using object literals if you are finding yourself needing a lot of arguments.  To make it obvious what properties the function expects, you can use the [destructuring](https://basarat.gitbooks.io/typescript/docs/destructuring.html) syntax.This has a few advantages:

如果需要很多参数，请您考虑使用对象。为了使函数期望的属性更清晰，可以使用[解构](https://basarat.gitbooks.io/typescript/docs/destructuring.html)，这有几个优点：

1. When someone looks at the function signature, it's immediately clear what properties are being used.

1. 当有人查看函数签名时，会立即清楚使用了哪些属性。

2. Destructuring also clones the specified primitive values of the argument object passed into the function. This can help prevent side effects. Note: objects and arrays that are destructured from the argument object are NOT cloned

2. 解构克隆传递给函数的参数对象的指定原始值，这有助于预防副作用。(注意：不会克隆从参数对象中解构的对象和数组)

3. TypeScript warns you about unused properties, which would be impossible without destructuring.

3. TypeScript会对未使用的属性显示警告，如果没有解构这是不可能的。



**反例:**

```ts

function createMenu(title: string, body: string, buttonText: string, cancellable: boolean) {

  // ...

}

createMenu('Foo', 'Bar', 'Baz', true);

```

**正例:**

```ts

function createMenu(options: {title: string, body: string, buttonText: string, cancellable: boolean}) {

  // ...

}

createMenu({

  title: 'Foo',

  body: 'Bar',

  buttonText: 'Baz',

  cancellable: true

});

```
通过 TypeScript's [类型别名](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases)，您可以进一步提高可读性。

```ts

type MenuOptions = {title: string, body: string, buttonText: string, cancellable: boolean};

function createMenu(options: MenuOptions) {

  // ...

}

createMenu({

  title: 'Foo',

  body: 'Bar',

  buttonText: 'Baz',

  cancellable: true

});

```

**[⬆ 回到顶部](#目录)**

### 只做一件事

This is by far the most important rule in software engineering. When functions do more than one thing, they are harder to compose, test, and reason about. When you can isolate a function to just one action, they can be refactored easily and your code will read much cleaner. If you take nothing else away from this guide other than this, you'll be ahead of many developers.

这是目前为止软件工程中最重要的规则。当函数做不止一件事时，它就更难组合、测试以及推理。当您将函数分割只实现一个操作时，它就更易于重构、代码就更清晰。如果您只从本指南中了解到这一点，那么您就优于多数程序员了。

**反例:**

```ts

function emailClients(clients: Client) {

  clients.forEach((client) => {

    const clientRecord = database.lookup(client);

    if (clientRecord.isActive()) {

      email(client);

    }

  });

}

```

**正例:**

```ts

function emailClients(clients: Client) {

  clients.filter(isActiveClient).forEach(email);

}

function isActiveClient(client: Client) {

  const clientRecord = database.lookup(client);

  return clientRecord.isActive();

}

```

**[⬆ 回到顶部](#目录)**

### 名副其实

从函数名就可看出函数的功能。

**反例:**

```ts

function addToDate(date: Date, month: number): Date {

  // ...

}

const date = new Date();

// It's hard to tell from the function name what is added

addToDate(date, 1);

```

**正例:**

```ts

function addMonthToDate(date: Date, month: number): Date {

  // ...

}

const date = new Date();

addMonthToDate(date, 1);

```

**[⬆ 回到顶部](#目录)**

### 每个函数只包含同一个层级的抽象

When you have more than one level of abstraction your function is usually doing too much. Splitting up functions leads to reusability and easier testing.

当您有多个抽象级别时，您的函数通常会做得太多。分解函数可以实现可重用性和更容易的测试。

**反例:**

```ts

function parseCode(code:string) {

  const REGEXES = [ /* ... */ ];

  const statements = code.split(' ');

  const tokens = [];

  REGEXES.forEach((regex) => {

    statements.forEach((statement) => {

      // ...

    });

  });

  const ast = [];

  tokens.forEach((token) => {

    // lex...

  });

  ast.forEach((node) => {

    // parse...

  });

}

```

**正例:**

```ts

const REGEXES = [ /* ... */ ];

function parseCode(code:string) {

  const tokens = tokenize(code);

  const syntaxTree = parse(tokens);

  syntaxTree.forEach((node) => {

    // parse...

  });

}

function tokenize(code: string):Token[] {

  const statements = code.split(' ');

  const tokens:Token[] = [];

  REGEXES.forEach((regex) => {

    statements.forEach((statement) => {

      tokens.push( /* ... */ );

    });

  });

  return tokens;

}

function parse(tokens: Token[]): SyntaxTree {

  const syntaxTree:SyntaxTree[] = [];

  tokens.forEach((token) => {

    syntaxTree.push( /* ... */ );

  });

  return syntaxTree;

}

```

**[⬆ 回到顶部](#目录)**

### 删除重复代码

Do your absolute best to avoid duplicate code.Duplicate code is bad because it means that there's more than one place to alter something if you need to change some logic.  

尽力避免重复代码。 重复代码很糟糕，因为它意味着如果您需要更改某些逻辑，则可以有多个位置来更改某些内容。

Imagine if you run a restaurant and you keep track of your inventory: all your tomatoes, onions, garlic, spices, etc.

If you have multiple lists that you keep this on, then all have to be updated when you serve a dish with tomatoes in them.

If you only have one list, there's only one place to update!  

想象一下，如果你经营一家餐厅，你要记录你的库存:你所有的西红柿、洋葱、大蒜、香料等等。如果您只有一个列表，那么只有一个地方可以更新!

Oftentimes you have duplicate code because you have two or more slightly different things, that share a lot in common, but their differences force you to have two or more separate functions that do much of the same things. Removing duplicate code means creating an abstraction that can handle this set of different things with just one function/module/class.  

通常你有重复的代码，因为你有两个或两个以上稍微不同的东西，它们有很多共同点，但是它们的不同迫使你有两个或多个独立的函数来做很多相同的事情。删除重复代码意味着创建一个抽象，该抽象仅用一个函数/模块/类就可以处理这组不同的东西。

Getting the abstraction right is critical, that's why you should follow the [SOLID](#solid) principles. Bad abstractions can be worse than duplicate code, so be careful! Having said this, if you can make a good abstraction, do it! Don't repeat yourself, otherwise you'll find yourself updating multiple places anytime you want to change one thing.

获得正确的抽象是至关重要的，这就是为什么您应该遵循坚实的原则。糟糕的抽象可能比重复的代码更糟糕，所以要小心!话虽如此，如果你能做一个好的抽象，那就去做吧!不要重复你自己，否则你会发现你随时都在更新多个地方，只要你想要改变一件事。

**反例:**

```ts

function showDeveloperList(developers: Developer[]) {

  developers.forEach((developer) => {

    const expectedSalary = developer.calculateExpectedSalary();

    const experience = developer.getExperience();

    const githubLink = developer.getGithubLink();

    const data = {

      expectedSalary,

      experience,

      githubLink

    };

    render(data);

  });

}

function showManagerList(managers: Manager[]) {

  managers.forEach((manager) => {

    const expectedSalary = manager.calculateExpectedSalary();

    const experience = manager.getExperience();

    const portfolio = manager.getMBAProjects();

    const data = {

      expectedSalary,

      experience,

      portfolio

    };

    render(data);

  });

}

```

**正例:**

```ts

class Developer {

  // ...

  getExtraDetails() {

    return {

      githubLink: this.githubLink,

    }

  }

}

class Manager {

  // ...

  getExtraDetails() {

    return {

      portfolio: this.portfolio,

    }

  }

}

function showEmployeeList(employee: Developer | Manager) {

  employee.forEach((employee) => {

    const expectedSalary = developer.calculateExpectedSalary();

    const experience = developer.getExperience();

    const extra = employee.getExtraDetails();

    const data = {

      expectedSalary,

      experience,

      extra,

    };

    render(data);

  });

}

```

You should be critical about code duplication. Sometimes there is a tradeoff between duplicated code and increased complexity by introducing unnecessary abstraction. When two implementations from two different modules look similar but live in different domains, duplication might be acceptable and preferred over extracting the common code. The extracted common code in this case introduces an indirect dependency between the two modules.

您应该对代码复制持批评态度。有时，在重复代码和引入不必要的抽象而增加的复杂性之间存在权衡。当来自两个不同模块的两个实现看起来相似，但位于不同的域中时，复制可能是可以接受的，并且优于提取公共代码。在本例中提取的公共代码引入了两个模块之间的间接依赖关系。


**[⬆ 回到顶部](#目录)**

### 使用`Object.assign`或解构来设置默认对象

**反例:**

```ts

type MenuConfig = {title?: string, body?: string, buttonText?: string, cancellable?: boolean};

function createMenu(config: MenuConfig) {

  config.title = config.title || 'Foo';

  config.body = config.body || 'Bar';

  config.buttonText = config.buttonText || 'Baz';

  config.cancellable = config.cancellable !== undefined ? config.cancellable : true;

}

const menuConfig = {

  title: null,

  body: 'Bar',

  buttonText: null,

  cancellable: true

};

createMenu(menuConfig);

```

**正例:**

```ts

type MenuConfig = {title?: string, body?: string, buttonText?: string, cancellable?: boolean};

function createMenu(config: MenuConfig) {

  const menuConfig = Object.assign({

    title: 'Foo',

    body: 'Bar',

    buttonText: 'Baz',

    cancellable: true

  }, config);

}

createMenu({ body: 'Bar' });

```

或者，您可以使用默认值的解构:

```ts

type MenuConfig = {title?: string, body?: string, buttonText?: string, cancellable?: boolean};

function createMenu({title = 'Foo', body = 'Bar', buttonText = 'Baz', cancellable = true}: MenuConfig) {

  // ...

}

createMenu({ body: 'Bar' });

```

为了避免任何副作用和意外行为，通过显式传递未定义或`null`值，您可以告诉TypeScript编译器不允许它。参见TypeScript中的`--strictnullcheck`选项。

**[⬆ 回到顶部](#目录)**

### 不要使用Flag参数

标志告诉用户这个函数的功能不止一件。函数应该做一件事。如果函数遵循基于布尔值的不同代码路径，则将其拆分。

**反例:**

```ts

function createFile(name:string, temp:boolean) {

  if (temp) {

    fs.create(`./temp/${name}`);

  } else {

    fs.create(name);

  }

}

```

**正例:**

```ts

function createFile(name:string) {

  fs.create(name);

}

function createTempFile(name:string) {

  fs.create(`./temp/${name}`);

}

```

**[⬆ 回到顶部](#目录)**

### 避免副作用 (part1)

A function produces a side effect if it does anything other than take a value in and return another value or values.A side effect could be writing to a file, modifying some global variable, or accidentally wiring all your money to a stranger.  

函数如果不接受一个值并返回另一个或多个值，则会产生副作用。副作用可能是写入文件，修改某个全局变量，或者意外地将您所有的钱连接到一个陌生人。

Now, you do need to have side effects in a program on occasion. Like the previous example, you might need to write to a file.What you want to do is to centralize where you are doing this. Don't have several functions and classes that write to a particular file.Have one service that does it. One and only one.  

现在，你需要在一个程序中偶尔有副作用。与前面的示例一样，您可能需要写入文件。你要做的是集中你正在做的事情。不要有几个函数和类写一个特定的文件。有一个服务可以做到这一点。只有一个。

The main point is to avoid common pitfalls like sharing state between objects without any structure, using mutable data types that can be written to by anything, and not centralizing where your side effects occur. If you can do this, you will be happier than the vast majority of other programmers.

重点是要避免常见的陷阱，比如在没有任何结构的对象之间共享状态、使用任何东西都可以写入的可变数据类型，以及不集中副作用发生的位置。如果你能做到这一点，你会比绝大多数其他程序员更快乐。

**反例:**

```ts

// Global variable referenced by following function.

// If we had another function that used this name, now it'd be an array and it could break it.

let name = 'Robert C. Martin';

function toBase64() {

  name = btoa(name);

}

toBase64(); // produces side effects to `name` variable

console.log(name); // expected to print 'Robert C. Martin' but instead 'Um9iZXJ0IEMuIE1hcnRpbg=='

```

**正例:**

```ts

// Global variable referenced by following function.

// If we had another function that used this name, now it'd be an array and it could break it.

const name = 'Robert C. Martin';

function toBase64(text:string):string {

  return btoa(text);

}

const encodedName = toBase64(name);

console.log(name);

```

**[⬆ 回到顶部](#目录)**

### 避免副作用 (part2)

In JavaScript, primitives are passed by value and objects/arrays are passed by reference. In the case of objects and arrays, if your function makes a change in a shopping cart array, for example, by adding an item to purchase, then any other function that uses that cart array will be affected by this addition. That may be great, however it can be bad too. Let's imagine a bad situation:  

在JavaScript中，原语通过值传递，对象/数组通过引用传递。在对象和数组的情况下，如果您的函数在购物车数组中进行了更改，例如，通过添加要购买的商品，那么使用该购物车数组的任何其他函数都将受到此添加的影响。这也许是好事，但也可能是坏事。让我们想象一个糟糕的情况:

The user clicks the "Purchase", button which calls a purchase function that spawns a network request and sends the cart array to the server. Because of a bad network connection, the purchase function has to keep retrying the request. Now, what if in the meantime the user accidentally clicks "Add to Cart" button on an item they don't actually want before the network request begins? If that happens and the network request begins, then that purchase function will send the accidentally added item because it has a reference to a shopping cart array that the *addItemToCart* function modified by adding an unwanted item.  

用户单击“购买”按钮，该按钮调用生成网络请求的购买函数，并将购物车数组发送到服务器。由于网络连接不好，购买功能必须不断重试请求。现在，如果用户在网络请求开始之前不小心单击了他们实际上不想要的项目上的“Add to Cart”按钮，该怎么办?如果发生这种情况，并且网络请求开始，那么purchase函数将发送意外添加的项，因为它引用了一个购物车数组，addItemToCart函数通过添加不需要的项修改了该数组。

A great solution would be for the *addItemToCart* to always clone the cart, edit it, and return the clone. This ensures that no other functions that are holding onto a reference of the shopping cart will be affected by any changes.  

一个很好的解决方案是addItemToCart总是克隆购物车、编辑购物车并返回克隆。这确保保存购物车引用的其他函数不会受到任何更改的影响。

Two caveats to mention to this approach:
关于这种方法有两点需要注意:

1. There might be cases where you actually want to modify the input object, but when you adopt this programming practice you will find that those cases are pretty rare. Most things can be refactored to have no side effects! (see [pure function](https://en.wikipedia.org/wiki/Pure_function))

在某些情况下，您可能确实想要修改输入对象，但是当您采用这种编程实践时，您会发现这种情况非常少见。大多数东西都可以重构，没有副作用!(见纯函数)

2. Cloning big objects can be very expensive in terms of performance. Luckily, this isn't a big issue in practice because there are great libraries that allow this kind of programming approach to be fast and not as memory intensive as it would be for you to manually clone objects and arrays.

就性能而言，克隆大型对象可能非常昂贵。幸运的是，这在实践中不是一个大问题，因为有一些很好的库允许这种编程方法快速，并且不像手动克隆对象和数组那样占用大量内存。


**反例:**

```ts

function addItemToCart(cart: CartItem[], item:Item):void {

  cart.push({ item, date: Date.now() });

};

```

**正例:**

```ts

function addItemToCart(cart: CartItem[], item:Item):CartItem[] {

  return [...cart, { item, date: Date.now() }];

};

```

**[⬆ 回到顶部](#目录)**

### 不要写全局函数
Don't write to global functions

Polluting globals is a bad practice in JavaScript because you could clash with another library and the user of your API would be none-the-wiser until they get an exception in production. Let's think about an example: what if you wanted to extend JavaScript's native Array method to have a diff method that could show the difference between two arrays? You could write your new function to the `Array.prototype`, but it could clash with another library that tried to do the same thing. What if that other library was just using `diff` to find the difference between the first and last elements of an array? This is why it would be much better to just use classes and simply extend the `Array` global.

污染全局变量在JavaScript中是一种不好的做法，因为您可能与另一个库发生冲突，而您的API的用户在生产环境中获得异常之前是不会知道的。让我们考虑一个例子:如果您想要扩展JavaScript的原生数组方法，使其具有一个可以显示两个数组之间差异的diff方法，该怎么办?可以将新函数写入数组。原型，但它可能与另一个尝试做同样事情的库冲突。如果另一个库只是使用diff来查找数组的第一个元素和最后一个元素之间的区别呢?这就是为什么只使用类并简单地扩展数组全局变量会更好的原因。

**反例:**

```ts

declare global {

  interface Array<T> {

    diff(other: T[]): Array<T>;

  }

}

if (!Array.prototype.diff){

  Array.prototype.diff = function <T>(other: T[]): T[] {

    const hash = new Set(other);

    return this.filter(elem => !hash.has(elem));

  };

}

```

**正例:**

```ts

class MyArray<T> extends Array<T> {

  diff(other: T[]): T[] {

    const hash = new Set(other);

    return this.filter(elem => !hash.has(elem));

  };

}

```

**[⬆ 回到顶部](#目录)**

### 比起命令式编程，更喜欢函数式编程

Favor functional programming over imperative programming

Favor this style of programming when you can.

尽可能喜欢这种编程风格。

**反例:**

```ts

const contributions = [

  {

    name: 'Uncle Bobby',

    linesOfCode: 500

  }, {

    name: 'Suzie Q',

    linesOfCode: 1500

  }, {

    name: 'Jimmy Gosling',

    linesOfCode: 150

  }, {

    name: 'Gracie Hopper',

    linesOfCode: 1000

  }

];

let totalOutput = 0;

for (let i = 0; i < contributions.length; i++) {

  totalOutput += contributions[i].linesOfCode;

}

```

**正例:**

```ts

const contributions = [

  {

    name: 'Uncle Bobby',

    linesOfCode: 500

  }, {

    name: 'Suzie Q',

    linesOfCode: 1500

  }, {

    name: 'Jimmy Gosling',

    linesOfCode: 150

  }, {

    name: 'Gracie Hopper',

    linesOfCode: 1000

  }

];

const totalOutput = contributions

  .reduce((totalLines, output) => totalLines + output.linesOfCode, 0)

```

**[⬆ 回到顶部](#目录)**

### 封装条件

**反例:**

```ts

if (subscription.isTrial || account.balance > 0) {

  // ...

}

```

**正例:**

```ts

function canActivateService(subscription: Subscription, account: Account) {

  return subscription.isTrial || account.balance > 0

}

if (canActivateService(subscription, account)) {

  // ...

}

```

**[⬆ 回到顶部](#目录)**

### 避免反向条件 Avoid negative conditionals

**反例:**

```ts

function isEmailNotUsed(email: string) {

  // ...

}

if (isEmailNotUsed(email)) {

  // ...

}

```

**正例:**

```ts

function isEmailUsed(email) {

  // ...

}

if (!isEmailUsed(node)) {

  // ...

}

```

**[⬆ 回到顶部](#目录)**

### 避免条件

This seems like an impossible task. Upon first hearing this, most people say, "how am I supposed to do anything without an `if` statement?" The answer is that you can use polymorphism to achieve the same task in many cases. The second question is usually, *"well that's great but why would I want to do that?"* The answer is a previous clean code concept we learned: a function should only do one thing. When you have classes and functions that have `if` statements, you are telling your user that your function does more than one thing. Remember, just do one thing.

这似乎是一项不可能完成的任务。第一次听到这个，大多数人会说，“如果没有if语句，我该怎么做呢?”答案是，在许多情况下，您可以使用多态性来完成相同的任务。第二个问题通常是，“这很好，但我为什么要这么做?”答案是我们之前学过的一个干净的代码概念:函数应该只做一件事。当你的类和函数有if语句时，你是在告诉你的用户你的函数不止做一件事。记住，只做一件事。

**反例:**

```ts

class Airplane {

  private type: string;

  // ...

  getCruisingAltitude() {

    switch (this.type) {

      case '777':

        return this.getMaxAltitude() - this.getPassengerCount();

      case 'Air Force One':

        return this.getMaxAltitude();

      case 'Cessna':

        return this.getMaxAltitude() - this.getFuelExpenditure();

      default:

        throw new Error('Unknown airplane type.');

    }

  }

}

```

**正例:**

```ts

class Airplane {

  // ...

}

class Boeing777 extends Airplane {

  // ...

  getCruisingAltitude() {

    return this.getMaxAltitude() - this.getPassengerCount();

  }

}

class AirForceOne extends Airplane {

  // ...

  getCruisingAltitude() {

    return this.getMaxAltitude();

  }

}

class Cessna extends Airplane {

  // ...

  getCruisingAltitude() {

    return this.getMaxAltitude() - this.getFuelExpenditure();

  }

}

```

**[⬆ 回到顶部](#目录)**

### 避免类型检查

TypeScript is a strict syntactical superset of JavaScript and adds optional static type checking to the language.Always prefer to specify types of variables, parameters and return values to leverage the full power of TypeScript features.It makes refactoring more easier.

TypeScript是JavaScript的一个严格的语法超集，它向语言中添加了可选的静态类型检查。总是喜欢指定变量、参数和返回值的类型，以充分利用TypeScript特性的强大功能。它使重构更容易。

**反例:**

```ts

function travelToTexas(vehicle: Bicycle | Car) {

  if (vehicle instanceof Bicycle) {

    vehicle.pedal(this.currentLocation, new Location('texas'));

  } else if (vehicle instanceof Car) {

    vehicle.drive(this.currentLocation, new Location('texas'));

  }

}

```

**正例:**

```ts

type Vehicle = Bicycle | Car;

function travelToTexas(vehicle: Vehicle) {

  vehicle.move(this.currentLocation, new Location('texas'));

}

```

**[⬆ 回到顶部](#目录)**

### 不要过度优化

Modern browsers do a lot of optimization under-the-hood at runtime. A lot of times, if you are optimizing then you are just wasting your time. There are good [resources](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers) for seeing where optimization is lacking. Target those in the meantime, until they are fixed if they can be.

现代浏览器在运行时进行大量的底层优化。很多时候，如果你在优化，那么你只是在浪费时间。有很好的资源可以看到哪里缺乏优化。在此期间锁定这些目标，直到它们能够被修复为止。

**反例:**

```ts

// On old browsers, each iteration with uncached `list.length` would be costly

// because of `list.length` recomputation. In modern browsers, this is optimized.

for (let i = 0, len = list.length; i < len; i++) {

  // ...

}

```

**正例:**

```ts

for (let i = 0; i < list.length; i++) {

  // ...

}

```

**[⬆ 回到顶部](#目录)**

### 删除dead code

Dead code is just as bad as duplicate code. There's no reason to keep it in your codebase.If it's not being called, get rid of it! It will still be safe in your version history if you still need it.

死代码和重复代码一样糟糕。没有理由把它保存在代码库中。如果它没有被调用，就删除它!如果您仍然需要它，它在您的版本历史中仍然是安全的。

**反例:**

```ts

function oldRequestModule(url: string) {

  // ...

}

function requestModule(url: string) {

  // ...

}

const req = requestModule;

inventoryTracker('apples', req, 'www.inventory-awesome.io');

```

**正例:**

```ts

function requestModule(url: string) {

  // ...

}

const req = requestModule;

inventoryTracker('apples', req, 'www.inventory-awesome.io');

```

**[⬆ 回到顶部](#目录)**

## 对象和数据结构

### 使用`getters`和`setters`

TypeScript supports getter/setter syntax.Using getters and setters to access data from objects that encapsulate behavior could be better that simply looking for a property on an object."Why?" you might ask. Well, here's a list of reasons:

TypeScript支持getter/setter语法。使用getter和setter从封装行为的对象访问数据可能比简单地在对象上查找属性要好。你可能会问:“为什么?”这里有一些原因:

* When you want to do more beyond getting an object property, you don't have to look up and change every accessor in your codebase.

当您想要做更多的事情而不仅仅是获取对象属性时，您不必查找并更改代码基中的每个访问器。

* Makes adding validation simple when doing a set.

使在执行集合时添加验证变得简单。

* Encapsulates the internal representation.

封装内部表示。

* Easy to add logging and error handling when getting and setting.

在获取和设置时很容易添加日志记录和错误处理。

* You can lazy load your object's properties, let's say getting it from a server.

您可以延迟加载对象的属性，比如从服务器获取它。

**反例:**

```ts

class BankAccount {

  balance: number = 0;

  // ...

}

const value = 100;

const account = new BankAccount();

if (value < 0) {

  throw new Error('Cannot set negative balance.');

}

account.balance = value;

```

**正例:**

```ts

class BankAccount {

  private accountBalance: number = 0;

  get balance(): number {

    return this.accountBalance;

  }

  set balance(value: number) {

    if (value < 0) {

      throw new Error('Cannot set negative balance.');

    }

    this.accountBalance = value;

  }

  // ...

}

const account = new BankAccount();

account.balance = 100;

```

**[⬆ 回到顶部](#目录)**

### 让对象拥有private/protected成员

TypeScript支持 `public` *(default)*, `protected` and `private` accessors on class members.  

**反例:**

```ts

class Circle {

  radius: number;

  

  constructor(radius: number) {

    this.radius = radius;

  }

  perimeter(){

    return 2 * Math.PI * this.radius;

  }

  surface(){

    return Math.PI * this.radius * this.radius;

  }

}

```

**正例:**

```ts

class Circle {

  constructor(private readonly radius: number) {

  }

  perimeter(){

    return 2 * Math.PI * this.radius;

  }

  surface(){

    return Math.PI * this.radius * this.radius;

  }

}

```

**[⬆ 回到顶部](#目录)**

### Prefer immutability 不变性

TypeScript's type system allows you to mark individual properties on an interface / class as readonly. This allows you to work in a functional way (unexpected mutation is bad).  

TypeScript的类型系统允许你将接口/类上的单个属性标记为只读。这允许您以一种功能性的方式工作(意外的突变是不好的)。

For more advanced scenarios there is a built-in type `Readonly` that takes a type `T` and marks all of its properties as readonly using mapped types (see [mapped types](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types)).

对于更高级的场景，有一个内置的类型Readonly，它接受类型T并使用映射类型(参见映射类型)将其所有属性标记为只读。

**反例:**

```ts

interface Config {

  host: string;

  port: string;

  db: string;

}

```

**正例:**

```ts

interface Config {

  readonly host: string;

  readonly port: string;

  readonly db: string;

}

```

**[⬆ 回到顶部](#目录)**

## 类

### 小、小、小，重要事情说三遍

The class' size is measured by it's responsibility. Following the *Single Responsibility principle* a class should be small.

类的大小是由它的职责来度量的。按照单一责任原则，一个类应该很小。

**反例:**

```ts

class Dashboard {

  getLanguage(): string { /* ... */ }

  setLanguage(language: string): void { /* ... */ }

  showProgress(): void { /* ... */ }

  hideProgress(): void { /* ... */ }

  isDirty(): boolean { /* ... */ }

  disable(): void { /* ... */ }

  enable(): void { /* ... */ }

  addSubscription(subscription: Subscription): void { /* ... */ }

  removeSubscription(subscription: Subscription): void { /* ... */ }

  addUser(user: User): void { /* ... */ }

  removeUser(user: User): void { /* ... */ }

  goToHomePage(): void { /* ... */ }

  updateProfile(details: UserDetails): void { /* ... */ }

  getVersion(): string { /* ... */ }

  // ...

}

```

**正例:**

```ts

class Dashboard {

  disable(): void { /* ... */ }

  enable(): void { /* ... */ }

  getVersion(): string { /* ... */ }

}

// split the responsibilities by moving the remaining methods to other classes

// ...

```

**[⬆ 回到顶部](#目录)**

### 高内聚低耦合

High cohesion and low coupling Cohesion defines the degree to which class members are related to each other. Ideally, all fields within a class should be used by each method. We then say that the class is maximally cohesive. In practice, this however is not always possible, nor even advisable. You should however prefer cohesion to be high.  

内聚定义了类成员之间相互关联的程度。理想情况下，每个方法都应该使用类中的所有字段。然后我们说这个类是最大内聚的。然而，在实践中，这并不总是可能的，甚至是不可取的。然而，您应该更喜欢高内聚力。

Coupling refers to how related or dependent are two classes toward each other. Classes are said to be low coupled if changes in one of them doesn't affect the other one.  

耦合指的是两个类之间的关联程度。如果其中一个类中的更改不影响另一个类，则称为低耦合类。

Good software design has **high cohesion** and **low coupling**.

好的软件设计具有高内聚性和低耦合性。

**反例:**

```ts

class UserManager {

  // Bad: each private variable is used by one or another group of methods.

  // It makes clear evidence that the class is holding more than a single responsibility.

  // If I need only to create the service to get the transactions for a user,

  // I'm still forced to pass and instance of emailSender.

  constructor(

    private readonly db: Database,

    private readonly emailSender: EmailSender) {

  }

  async getUser(id: number): Promise<User> {

    return await db.users.findOne({ id })

  }

  async getTransactions(userId: number): Promise<Transaction[]> {

    return await db.transactions.find({ userId })

  }

  async sendGreeting(): Promise<void> {

    await emailSender.send('Welcome!');

  }

  async sendNotification(text: string): Promise<void> {

    await emailSender.send(text);

  }

  async sendNewsletter(): Promise<void> {

    // ...

  }

}

```

**正例:**

```ts

class UserService {

  constructor(private readonly db: Database) {

  }

  async getUser(id: number): Promise<User> {

    return await db.users.findOne({ id })

  }

  async getTransactions(userId: number): Promise<Transaction[]> {

    return await db.transactions.find({ userId })

  }

}

class UserNotifier {

  constructor(private readonly emailSender: EmailSender) {

  }

  async sendGreeting(): Promise<void> {

    await emailSender.send('Welcome!');

  }

  async sendNotification(text: string): Promise<void> {

    await emailSender.send(text);

  }

  async sendNewsletter(): Promise<void> {

    // ...

  }

}

```

**[⬆ 回到顶部](#目录)**

### 组合大于继承

Prefer composition over inheritance 

As stated famously in [Design Patterns](https://en.wikipedia.org/wiki/Design_Patterns) by the Gang of Four, you should prefer composition over inheritance where you can. There are lots of good reasons to use inheritance and lots of good reasons to use composition. The main point for this maxim is that if your mind instinctively goes for inheritance, try to think if composition could model your problem better. In some cases it can.  

正如“四人帮”在设计模式中所指出的那样，在可能的情况下，您应该更喜欢组合而不是继承。有很多好的理由使用继承，也有很多好的理由使用组合。这句格言的主要观点是，如果你的头脑本能地倾向于继承，试着想想组合是否能更好地模拟你的问题。在某些情况下可以。  

You might be wondering then, "when should I use inheritance?" It depends on your problem at hand, but this is a decent list of when inheritance makes more sense than composition:

您可能会想，“我什么时候应该使用继承?”这取决于你手头的问题，但这是一个不错的列表，列出了什么时候继承比组合更有意义:

1. Your inheritance represents an "is-a" relationship and not a "has-a" relationship (Human->Animal vs. User->UserDetails).

您的继承代表的是“is-a”关系，而不是“has-a”关系(Human->Animal vs. User->UserDetails)

2. You can reuse code from the base classes (Humans can move like all animals).

您可以重用基类中的代码(人类可以像所有动物一样移动)。

3. You want to make global changes to derived classes by changing a base class. (Change the caloric expenditure of all animals when they move).

您希望通过更改基类对派生类进行全局更改。(改变所有动物在运动时的热量消耗)。

**反例:**

```ts

class Employee {

  constructor(

    private readonly name: string, 

    private readonly email:string) {

  }

  // ...

}

// Bad because Employees "have" tax data. EmployeeTaxData is not a type of Employee

class EmployeeTaxData extends Employee {

  constructor(

    name: string, 

    email:string,

    private readonly ssn: string, 

    private readonly salary: number) {

    super(name, email);

  }

  // ...

}

```

**正例:**

```ts

class Employee {

  private taxData: EmployeeTaxData;

  constructor(

    private readonly name: string, 

    private readonly email:string) {

  }

  setTaxData(ssn: string, salary: number): Employee {

    this.taxData = new EmployeeTaxData(ssn, salary);

    return this;

  }

  // ...

}

class EmployeeTaxData {

  constructor(

    public readonly ssn: string, 

    public readonly salary: number) {

  }

  // ...

}

```

**[⬆ 回到顶部](#目录)**

### 使用方法链

This pattern is very useful and commonly used in many libraries. It allows your code to be expressive, and less verbose. For that reason, use method chaining and take a look at how clean your code will be.

这个模式非常有用，在许多库中都经常使用。它允许您的代码具有表达性，并且不那么冗长。出于这个原因，请使用方法链接，并查看代码的整洁程度。

**反例:**

```ts

class QueryBuilder {

  private collection: string;

  private pageNumber: number = 1;

  private itemsPerPage: number = 100;

  private orderByFields: string[] = [];

  from(collection: string): void {

    this.collection = collection;

  }

  page(number: number, itemsPerPage: number = 100): void {

    this.pageNumber = number;

    this.itemsPerPage = itemsPerPage;

  }

  orderBy(...fields: string[]): void {

    this.orderByFields = fields;

  }

  build(): Query {

    // ...

  }

}

// ...

const query = new QueryBuilder();

query.from('users');

query.page(1, 100);

query.orderBy('firstName', 'lastName');

const query = queryBuilder.build();

```

**正例:**

```ts

class QueryBuilder {

  private collection: string;

  private pageNumber: number = 1;

  private itemsPerPage: number = 100;

  private orderByFields: string[] = [];

  from(collection: string): this {

    this.collection = collection;

    return this;

  }

  page(number: number, itemsPerPage: number = 100): this {

    this.pageNumber = number;

    this.itemsPerPage = itemsPerPage;

    return this;

  }

  orderBy(...fields: string[]): this {

    this.orderByFields = fields;

    return this;

  }

  build(): Query {

    // ...

  }

}

// ...

const query = new QueryBuilder()

  .from('users')

  .page(1, 100)

  .orderBy('firstName', 'lastName')

  .build();

```

**[⬆ 回到顶部](#目录)**

## SOLID原则

### 单一职责原则 (SRP)

As stated in Clean Code, "There should never be more than one reason for a class to change". It's tempting to jam-pack a class with a lot of functionality, like when you can only take one suitcase on your flight. The issue with this is that your class won't be conceptually cohesive and it will give it many reasons to change. Minimizing the amount of times you need to change a class is important. It's important because if too much functionality is in one class and you modify a piece of it, it can be difficult to understand how that will affect other dependent modules in your codebase.

**反例:**

```ts

class UserSettings {

  constructor(private readonly user: User) {

  }

  changeSettings(settings: UserSettings) {

    if (this.verifyCredentials()) {

      // ...

    }

  }

  verifyCredentials() {

    // ...

  }

}

```

**正例:**

```ts

class UserAuth {

  constructor(private readonly user: User) {

  }

  verifyCredentials() {

    // ...

  }

}

class UserSettings {

  private readonly auth: UserAuth;

  constructor(private readonly user: User) {

    this.auth = new UserAuth(user);

  }

  changeSettings(settings: UserSettings) {

    if (this.auth.verifyCredentials()) {

      // ...

    }

  }

}

```

**[⬆ 回到顶部](#目录)**

### 开闭原则 (OCP)

As stated by Bertrand Meyer, "software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification." What does that mean though? This principle basically states that you should allow users to add new functionalities without changing existing code.

**反例:**

```ts

class AjaxAdapter extends Adapter {

  constructor() {

    super();

  }

  // ...

}

class NodeAdapter extends Adapter {

  constructor() {

    super();

  }

  // ...

}

class HttpRequester {

  constructor(private readonly adapter: Adapter) {

  }

  async fetch<T>(url: string): Promise<T> {

    if (this.adapter instanceof AjaxAdapter) {

      const response = await makeAjaxCall<T>(url);

      // transform response and return

    } else if (this.adapter instanceof NodeAdapter) {

      const response = await makeHttpCall<T>(url);

      // transform response and return

    }

  }

}

function makeAjaxCall<T>(url: string): Promise<T> {

  // request and return promise

}

function makeHttpCall<T>(url: string): Promise<T> {

  // request and return promise

}

```

**正例:**

```ts

abstract class Adapter {

  abstract async request<T>(url: string): Promise<T>;

}

class AjaxAdapter extends Adapter {

  constructor() {

    super();

  }

  async request<T>(url: string): Promise<T>{

    // request and return promise

  }

  // ...

}

class NodeAdapter extends Adapter {

  constructor() {

    super();

  }

  async request<T>(url: string): Promise<T>{

    // request and return promise

  }

  // ...

}

class HttpRequester {

  constructor(private readonly adapter: Adapter) {

  }

  async fetch<T>(url: string): Promise<T> {

    const response = await this.adapter.request<T>(url);

    // transform response and return

  }

}

```

**[⬆ 回到顶部](#目录)**

### 里氏替换原则 (LSP)

This is a scary term for a very simple concept. It's formally defined as "If S is a subtype of T, then objects of type T may be replaced with objects of type S (i.e., objects of type S may substitute objects of type T) without altering any of the desirable properties of that program (correctness, task performed, etc.)." That's an even scarier definition.  

  

The best explanation for this is if you have a parent class and a child class, then the base class and child class can be used interchangeably without getting incorrect results. This might still be confusing, so let's take a look at the classic Square-Rectangle example. Mathematically, a square is a rectangle, but if you model it using the "is-a" relationship via inheritance, you quickly get into trouble.

**反例:**

```ts

class Rectangle {

  constructor(

    protected width: number = 0, 

    protected height: number = 0) {

  }

  setColor(color: string) {

    // ...

  }

  render(area: number) {

    // ...

  }

  setWidth(width: number) {

    this.width = width;

  }

  setHeight(height: number) {

    this.height = height;

  }

  getArea(): number {

    return this.width * this.height;

  }

}

class Square extends Rectangle {

  setWidth(width: number) {

    this.width = width;

    this.height = width;

  }

  setHeight(height: number) {

    this.width = height;

    this.height = height;

  }

}

function renderLargeRectangles(rectangles: Rectangle[]) {

  rectangles.forEach((rectangle) => {

    rectangle.setWidth(4);

    rectangle.setHeight(5);

    const area = rectangle.getArea(); // BAD: Returns 25 for Square. Should be 20.

    rectangle.render(area);

  });

}

const rectangles = [new Rectangle(), new Rectangle(), new Square()];

renderLargeRectangles(rectangles);

```

**正例:**

```ts

abstract class Shape {

  setColor(color: string) {

    // ...

  }

  render(area: number) {

    // ...

  }

  abstract getArea(): number;

}

class Rectangle extends Shape {

  constructor(

    private readonly width = 0, 

    private readonly height = 0) {

    super();

  }

  getArea(): number {

    return this.width * this.height;

  }

}

class Square extends Shape {

  constructor(private readonly length: number) {

    super();

  }

  getArea(): number {

    return this.length * this.length;

  }

}

function renderLargeShapes(shapes: Shape[]) {

  shapes.forEach((shape) => {

    const area = shape.getArea();

    shape.render(area);

  });

}

const shapes = [new Rectangle(4, 5), new Rectangle(4, 5), new Square(5)];

renderLargeShapes(shapes);

```

**[⬆ 回到顶部](#目录)**

### 接口隔离原则 (ISP)

ISP states that "Clients should not be forced to depend upon interfaces that they do not use.". This principle is very much related to the Single Responsibility Principle.

What it really means is that you should always design your abstractions in a way that the clients that are using the exposed methods do not get the whole pie instead. That also include imposing the clients with the burden of implementing methods that they don’t actually need.

**反例:**

```ts

interface ISmartPrinter {

  print();

  fax();

  scan();

}

class AllInOnePrinter implements ISmartPrinter {

  print() {

    // ...

  }  

  

  fax() {

    // ...

  }

  scan() {

    // ...

  }

}

class EconomicPrinter implements ISmartPrinter {

  print() {

    // ...

  }  

  

  fax() {

    throw new Error('Fax not supported.');

  }

  scan() {

    throw new Error('Scan not supported.');

  }

}

```

**正例:**

```ts

interface IPrinter {

  print();

}

interface IFax {

  fax();

}

interface IScanner {

  scan();

}

class AllInOnePrinter implements IPrinter, IFax, IScanner {

  print() {

    // ...

  }  

  

  fax() {

    // ...

  }

  scan() {

    // ...

  }

}

class EconomicPrinter implements IPrinter {

  print() {

    // ...

  }

}

```

**[⬆ 回到顶部](#目录)**

### 依赖反转原则(Dependency Inversion Principle)

This principle states two essential things:

1. High-level modules should not depend on low-level modules. Both should depend on abstractions.

2. Abstractions should not depend upon details. Details should depend on abstractions.

This can be hard to understand at first, but if you've worked with Angular, you've seen an implementation of this principle in the form of Dependency Injection (DI). While they are not identical concepts, DIP keeps high-level modules from knowing the details of its low-level modules and setting them up. It can accomplish this through DI. A huge benefit of this is that it reduces the coupling between modules. Coupling is a very bad development pattern because it makes your code hard to refactor.  

  

DIP is usually achieved by a using an inversion of control (IoC) container. An example of a powerful IoC container for TypeScript is [InversifyJs](https://www.npmjs.com/package/inversify)

**反例:**

```ts

import { readFile as readFileCb } from 'fs';

import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {

  // ..

}

class XmlFormatter {

  parse<T>(content: string): T {

    // Converts an XML string to an object T

  }

}

class ReportReader {

  // BAD: We have created a dependency on a specific request implementation.

  // We should just have ReportReader depend on a parse method: `parse`

  private readonly formatter = new XmlFormatter();

  async read(path: string): Promise<ReportData> {

    const text = await readFile(path, 'UTF8');

    return this.formatter.parse<ReportData>(text);

  }

}

// ...

const reader = new ReportReader();

await report = await reader.read('report.xml');

```

**正例:**

```ts

import { readFile as readFileCb } from 'fs';

import { promisify } from 'util';

const readFile = promisify(readFileCb);

type ReportData = {

  // ..

}

interface Formatter {

  parse<T>(content: string): T;

}

class XmlFormatter implements Formatter {

  parse<T>(content: string): T {

    // Converts an XML string to an object T

  }

}

class JsonFormatter implements Formatter {

  parse<T>(content: string): T {

    // Converts a JSON string to an object T

  }

}

class ReportReader {

  constructor(private readonly formatter: Formatter){

  }

  async read(path: string): Promise<ReportData> {

    const text = await readFile(path, 'UTF8');

    return this.formatter.parse<ReportData>(text);

  }

}

// ...

const reader = new ReportReader(new XmlFormatter());

await report = await reader.read('report.xml');

// or if we had to read a json report:

const reader = new ReportReader(new JsonFormatter());

await report = await reader.read('report.json');

```

**[⬆ 回到顶部](#目录)**

## 测试

Testing is more important than shipping. If you have no tests or an inadequate amount, then every time you ship code you won't be sure that you didn't break anything.

Deciding on what constitutes an adequate amount is up to your team, but having 100% coverage (all statements and branches)

is how you achieve very high confidence and developer peace of mind. This means that in addition to having a great testing framework, you also need to use a good coverage tool.

There's no excuse to not write tests. There are plenty of good JS test frameworks with typings support for TypeScript, so find one that your team prefers. When you find one that works for your team, then aim to always write tests for every new feature/module you introduce. If your preferred method is Test Driven Development (TDD), that is great, but the main point is to just make sure you are reaching your coverage goals before launching any feature, or refactoring an existing one.  

### TDD三法则
1. You are not allowed to write any production code unless it is to make a failing unit test pass.
2. You are not allowed to write any more of a unit test than is sufficient to fail; and compilation failures are failures.
3. You are not allowed to write any more production code than is sufficient to pass the one failing unit test.

**[⬆ 回到顶部](#目录)**

### F.I.R.S.T.规则

Clean tests should follow the rules:

* **Fast** tests should be fast because we want to run them frequently.

* **Independent** tests should not depend on each other. They should provide same output whether run independently or all together in any order.

* **Repeatable** tests should be repeatable in any environment and there should be no excuse for why they fail.

* **Self-Validating** a test should answer with either *Passed* or *Failed*. You don't need to compare log files to answer if a test passed.

* **Timely** unit tests should be written before the production code. If you write tests after the production code, you might find writing tests too hard.

**[⬆ 回到顶部](#目录)**

### Single concept per test

Tests should also follow the *Single Responsibility Principle*. Make only one assert per unit test.

**反例:**

```ts

import { assert } from 'chai';

describe('AwesomeDate', () => {

  it('handles date boundaries', () => {

    let date: AwesomeDate;

    date = new AwesomeDate('1/1/2015');

    date.addDays(30);

    assert.equal('1/31/2015', date);

    date = new AwesomeDate('2/1/2016');

    date.addDays(28);

    assert.equal('02/29/2016', date);

    date = new AwesomeDate('2/1/2015');

    date.addDays(28);

    assert.equal('03/01/2015', date);

  });

});

```

**正例:**

```ts

import { assert } from 'chai';

describe('AwesomeDate', () => {

  it('handles 30-day months', () => {

    const date = new AwesomeDate('1/1/2015');

    date.addDays(30);

    assert.equal('1/31/2015', date);

  });

  it('handles leap year', () => {

    const date = new AwesomeDate('2/1/2016');

    date.addDays(28);

    assert.equal('02/29/2016', date);

  });

  it('handles non-leap year', () => {

    const date = new AwesomeDate('2/1/2015');

    date.addDays(28);

    assert.equal('03/01/2015', date);

  });

});

```

**[⬆ 回到顶部](#目录)**

### The name of the test should reveal it's intention

When a test fail, it's name is the first indication of what may have gone wrong.

**反例:**

```ts

describe('Calendar', () => {

  it('2/29/2020', () => {

    // ...

  });

  it('throws', () => {

    // ...

  });

});

```

**正例:**

```ts

describe('Calendar', () => {

  it('should handle leap year', () => {

    // ...

  });

  it('should throw when format is invalid', () => {

    // ...

  });

});

```

**[⬆ 回到顶部](#目录)**

## 并发

### Promises优于Callbacks

Callbacks aren't clean, and they cause excessive amounts of nesting *(the callback hell)*.  

There are utilities that transform existing functions using the callback style to a version that returns promises 

(for Node.js see [`util.promisify`](https://nodejs.org/dist/latest-v8.x/docs/api/util.html#util_util_promisify_original), for general purpose see [pify](https://www.npmjs.com/package/pify), [es6-promisify](https://www.npmjs.com/package/es6-promisify))

**反例:**

```ts

import { get } from 'request';

import { writeFile } from 'fs';

function downloadPage(url: string, saveTo: string, callback: (error: Error, content?: string) => void){

  get(url, (error, response) => {

    if (error) {

      callback(error);

    } else {

      writeFile(saveTo, response.body, (error) => {

        if (error) {

          callback(error);

        } else {

          callback(null, response.body);

        }

      });

    }

  })

}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html', (error, content) => {

  if (error) {

    console.error(error);

  } else {

    console.log(content);

  }

});

```

**正例:**

```ts

import { get } from 'request';

import { writeFile } from 'fs';

import { promisify } from 'util';

const write = promisify(writeFile);

function downloadPage(url: string, saveTo: string): Promise<string> {

  return get(url)

    .then(response => write(saveTo, response))

}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html')

  .then(content => console.log(content))

  .catch(error => console.error(error));  

```

Promises supports a few patterns that could be useful in some cases:  

| Pattern                  | Description                                |  

| ------------------------ | -----------------------------------------  |  

| `Promise.resolve(value)` | Convert a value into a resolved promise.   |  

| `Promise.reject(error)`  | Convert an error into a rejected promise.  |  

| `Promise.all(promises)`  |Returns a new promise which is fulfilled with an array of fulfillment values for the passed promises or rejects with the reason of the first promise that rejects. |

| `Promise.race(promises)`|Returns a new promise which is fulfilled/rejected with the result/error of the first settled promise from the array of passed promises. |

`Promise.all` is especially useful when there is a need to run tasks in parallel. `Promise.race` makes it easier to implement things like timeouts for promises.

**[⬆ 回到顶部](#目录)**

### Async/Await优于Promises

With async/await syntax you can write code that is far cleaner and more understandable that chained promises. Within a function prefixed with `async` keyword you have a way to tell the JavaScript runtime to pause the execution of code on the `await` keyword (when used on a promise).

**反例:**

```ts

import { get } from 'request';

import { writeFile } from 'fs';

import { promisify } from 'util';

const write = util.promisify(writeFile);

function downloadPage(url: string, saveTo: string): Promise<string> {

  return get(url).then(response => write(saveTo, response))

}

downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html')

  .then(content => console.log(content))

  .catch(error => console.error(error));  

```

**正例:**

```ts

import { get } from 'request';

import { writeFile } from 'fs';

import { promisify } from 'util';

const write = promisify(writeFile);

async function downloadPage(url: string, saveTo: string): Promise<string> {

  const response = await get(url);

  await write(saveTo, response);

  return response;

}

// somewhere in an async function

try {

  const content = await downloadPage('https://en.wikipedia.org/wiki/Robert_Cecil_Martin', 'article.html');

  console.log(content);

} catch (error) {

  console.error(error);

}

```

**[⬆ 回到顶部](#目录)**

## 错误处理

Thrown errors are a good thing! They mean the runtime has successfully identified when something in your program has gone wrong and it's letting you know by stopping function

execution on the current stack, killing the process (in Node), and notifying you in the console with a stack trace.

### Always use Error for throwing or rejecting

JavaScript as well as TypeScript allow you to `throw` any object. A Promise can also be rejected with any reason object.  

It is advisable to use the `throw` syntax with an `Error` type. This is because your error might be caught in higher level code with a `catch` syntax.

It would be very confusing to catch a string message there and would make

[debugging more painful](https://basarat.gitbooks.io/typescript/docs/types/exceptions.html#always-use-error).  

For the same reason you should reject promises with `Error` types.

**反例:**

```ts

function calculateTotal(items: Item[]): number {

  throw 'Not implemented.';

}

function get(): Promise<Item[]> {

  return Promise.reject('Not implemented.');

}

```

**正例:**

```ts

function calculateTotal(items: Item[]): number {

  throw new Error('Not implemented.');

}

function get(): Promise<Item[]> {

  return Promise.reject(new Error('Not implemented.'));

}

// or equivalent to:

async function get(): Promise<Item[]> {

  throw new Error('Not implemented.');

}

```

The benefit of using `Error` types is that it is supported by the syntax `try/catch/finally` and implicitly all errors have the `stack` property which

is very powerful for debugging.  

There are also another alternatives, not to use the `throw` syntax and instead always return custom error objects. TypeScript makes this even easier.

Consider following example:

```ts

type Failable<R, E> = {

  isError: true;

  error: E;

} | {

  isError: false;

  value: R;

}

function calculateTotal(items: Item[]): Failable<number, 'empty'> {

  if (items.length === 0) {

    return { isError: true, error: 'empty' };

  }

  // ...

  return { isError: false, value: 42 };

}

```

For the detailed explanation of this idea refer to the [original post](https://medium.com/@dhruvrajvanshi/making-exceptions-type-safe-in-typescript-c4d200ee78e9).

**[⬆ 回到顶部](#目录)**

### 不能忽略捕获errors

Doing nothing with a caught error doesn't give you the ability to ever fix or react to said error. Logging the error to the console (`console.log`) isn't much better as often times it can get lost in a sea of things printed to the console. If you wrap any bit of code in a `try/catch` it means you think an error may occur there and therefore you should have a plan, or create a code path, for when it occurs.

**反例:**

```ts

try {

  functionThatMightThrow();

} catch (error) {

  console.log(error);

}

// or even worse

try {

  functionThatMightThrow();

} catch (error) {

  // ignore error

}

```

**正例:**

```ts

import { logger } from './logging'

try {

  functionThatMightThrow();

} catch (error) {

  logger.log(error);

}

```

**[⬆ 回到顶部](#目录)**

### Don't ignore rejected promises

For the same reason you shouldn't ignore caught errors from `try/catch`.

**反例:**

```ts

getUser()

  .then((user: User) => {

    return sendEmail(user.email, 'Welcome!');

  })

  .catch((error) => {

    console.log(error);

  });

```

**正例:**

```ts

import { logger } from './logging'

getUser()

  .then((user: User) => {

    return sendEmail(user.email, 'Welcome!');

  })

  .catch((error) => {

    logger.log(error);

  });

// or using the async/await syntax:

try {

  const user = await getUser();

  await sendEmail(user.email, 'Welcome!');

} catch (error) {

  logger.log(error);

}

```

**[⬆ 回到顶部](#目录)**

## 格式化

Formatting is subjective. Like many rules herein, there is no hard and fast rule that you must follow. The main point is *DO NOT ARGUE* over formatting. There are tons of tools to automate this. Use one! It's a waste of time and money for engineers to argue over formatting. The general rule to follow is *keep consistent formatting rules*.  

For TypeScript there is a powerful tool called [TSLint](https://palantir.github.io/tslint/). It's a static analysis tool that can help you improve dramatically the readability and maintainability of your code. There are ready to use TSLint configurations that you can reference in your projects:

* [TSLint Config Standard](https://www.npmjs.com/package/tslint-config-standard) - standard style rules

* [TSLint Config Airbnb](https://www.npmjs.com/package/tslint-config-airbnb) - Airbnb style guide

* [TSLint Clean Code](https://www.npmjs.com/package/tslint-clean-code) - TSLint rules inspired be the [Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.ca/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)

* [TSLint react](https://www.npmjs.com/package/tslint-react) - lint rules related to React & JSX

* [TSLint + Prettier](https://www.npmjs.com/package/tslint-config-prettier) - lint rules for [Prettier](https://github.com/prettier/prettier) code formatter

* [ESLint rules for TSLint](https://www.npmjs.com/package/tslint-eslint-rules) - ESLint rules for TypeScript

* [Immutable](https://www.npmjs.com/package/tslint-immutable) - rules to disable mutation in TypeScript

Refer also to this great [TypeScript StyleGuide and Coding Conventions](https://basarat.gitbooks.io/typescript/docs/styleguide/styleguide.html) source.

### Use consistent capitalization

Capitalization tells you a lot about your variables, functions, etc. These rules are subjective, so your team can choose whatever they want. The point is, no matter what you all choose, just *be consistent*.

**反例:**

```ts

const DAYS_IN_WEEK = 7;

const daysInMonth = 30;

const songs = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'];

const Artists = ['ACDC', 'Led Zeppelin', 'The Beatles'];

function eraseDatabase() {}

function restore_database() {}

class animal {}

class Container {}

```

**正例:**

```ts

const DAYS_IN_WEEK = 7;

const DAYS_IN_MONTH = 30;

const SONGS = ['Back In Black', 'Stairway to Heaven', 'Hey Jude'];

const ARTISTS = ['ACDC', 'Led Zeppelin', 'The Beatles'];

function eraseDatabase() {}

function restoreDatabase() {}

class Animal {}

class Container {}

```

Prefer using `PascalCase` for class, interface, type and namespace names.  

Prefer using `camelCase` for variables, functions and class members.

**[⬆ 回到顶部](#目录)**

### Function callers and callees should be close

If a function calls another, keep those functions vertically close in the source file. Ideally, keep the caller right above the callee.

We tend to read code from top-to-bottom, like a newspaper. Because of this, make your code read that way.

**反例:**

```ts

class PerformanceReview {

  constructor(private readonly employee: Employee) {

  }

  private lookupPeers() {

    return db.lookup(this.employee.id, 'peers');

  }

  private lookupManager() {

    return db.lookup(this.employee, 'manager');

  }

  private getPeerReviews() {

    const peers = this.lookupPeers();

    // ...

  }

  review() {

    this.getPeerReviews();

    this.getManagerReview();

    this.getSelfReview();

    // ...

  }

  private getManagerReview() {

    const manager = this.lookupManager();

  }

  private getSelfReview() {

    // ...

  }

}

const review = new PerformanceReview(employee);

review.review();

```

**正例:**

```ts

class PerformanceReview {

  constructor(private readonly employee: Employee) {

  }

  review() {

    this.getPeerReviews();

    this.getManagerReview();

    this.getSelfReview();

    // ...

  }

  private getPeerReviews() {

    const peers = this.lookupPeers();

    // ...

  }

  private lookupPeers() {

    return db.lookup(this.employee.id, 'peers');

  }

  private getManagerReview() {

    const manager = this.lookupManager();

  }

  private lookupManager() {

    return db.lookup(this.employee, 'manager');

  } 

  private getSelfReview() {

    // ...

  }

}

const review = new PerformanceReview(employee);

review.review();

```

**[⬆ 回到顶部](#目录)**

### 类型 vs 接口

Use type when you might need a union or intersection. Use interface when you want `extends` or `implements`. There is no strict rule however, use the one that works for you.  

参考这个关于Typescript中`type`和`interface`区别的[解释](https://stackoverflow.com/questions/37233735/typescript-interfaces-vs-types/54101543#54101543) 

**反例:**

```ts

interface EmailConfig {

  // ...

}

interface DbConfig {

  // ...

}

interface Config {

  // ...

}

//...

type Shape {

  // ...

}

```

**正例:**

```ts

type EmailConfig {

  // ...

}

type DbConfig {

  // ...

}

type Config  = EmailConfig | DbConfig;

// ...

interface Shape {

}

class Circle implements Shape {

  // ...

}

class Square implements Shape {

  // ...

}

```

**[⬆ 回到顶部](#目录)**

## 注释

The use of a comments is an indication of failure to express without them. Code should be the only source of truth.


> Don’t comment bad code—rewrite it.  

> — *Brian W. Kernighan and P. J. Plaugher*

### 代码应该自解释而不是注释

Comments are an apology, not a requirement. 
好代码即文档。

**反例:**

```ts

// Check if subscription is active.

if (subscription.endDate > Date.now) {  }

```

**正例:**

```ts

const isSubscriptionActive = subscription.endDate > Date.now;

if (isSubscriptionActive) { /* ... */ }

```

**[⬆ 回到顶部](#目录)**

### 不要将注释掉的代码留在代码库中
版本控制存在的一个理由，就是让旧代码成为历史。

**反例:**

```ts

class User {

  name: string;

  email: string;

  // age: number;

  // jobPosition: string;

}

```

**正例:**

```ts

class User {

  name: string;

  email: string;

}

```

**[⬆ 回到顶部](#目录)**

### 不要日记式注释

记住，使用版本控制！不需要保留无用代码(dead code)、注释代码，尤其是日记式注释，使用git log来获取历史。

**反例:**

```ts

/**

 * 2016-12-20: Removed monads, didn't understand them (RM)

 * 2016-10-01: Improved using special monads (JP)

 * 2016-02-03: Added type-checking (LI)

 * 2015-03-14: Implemented combine (JR)

 */

function combine(a:number, b:number): number {

  return a + b;

}

```

**正例:**

```ts

function combine(a:number, b:number): number {

  return a + b;

}

```

**[⬆ 回到顶部](#目录)**

### 避免位置标记

它们常常扰乱代码。让代码结构化，函数和变量有合适的缩进和格式。

作为一个可选项，你可以使用支持代码折叠的IDE (看下 Visual Studio Code [folding regions](https://code.visualstudio.com/updates/v1_17#_folding-regions)).

**反例:**

```ts

////////////////////////////////////////////////////////////////////////////////

// Client class

////////////////////////////////////////////////////////////////////////////////

class Client {

  id: number;

  name: string;

  address: Address;

  contact: Contact;

  ////////////////////////////////////////////////////////////////////////////////

  // public methods

  ////////////////////////////////////////////////////////////////////////////////

  public describe(): string {

    // ...

  }

  ////////////////////////////////////////////////////////////////////////////////

  // private methods

  ////////////////////////////////////////////////////////////////////////////////

  private describeAddress(): string {

    // ...

  }

  private describeContact(): string {

    // ...

  }

};

```

**正例:**

```ts

class Client {

  id: number;

  name: string;

  address: Address;

  contact: Contact;

  public describe(): string {

    // ...

  }

  private describeAddress(): string {

    // ...

  }

  private describeContact(): string {

    // ...

  }

};

```


**[⬆ 回到顶部](#目录)**

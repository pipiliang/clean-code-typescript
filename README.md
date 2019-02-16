# TypeScript 代码整洁之道
本项目是对[clean-code-typescript](https://github.com/labs42io/clean-code-typescript)项目的翻译及精简。由于个人水平有限，如有存在错误之处烦请指明!

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
  12. 翻译

## 简介

![Humorous image of software quality estimation as a count of how many expletives you shout when reading code](https://www.osnews.com/images/comics/wtfm.jpg)

这不是 TypeScript 的编码规范，而是将 Robert C. Martin 的软件工程著作 [*Clean Code*](https://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) 适用到 TypeScript，指导如何使用 TypeScript 编写[易读、可复用和易重构](https://github.com/ryanmcdermott/3rs-of-software-architecture)的软件。

并不是每个原则都要严格遵守，能被普遍认同的原则就更少了。虽然只是一些指导，但却是*Clean Code*作者对多年编码经验的总结及凝练。

软件工程技术已有50多年的历史了，我们仍然要学习很多的东西。当软件架构和架构本身一样古老的时候，也许我们会有更严格的规则来遵守。现在，让这些指导原则作为评估您和您的团队代码质量的试金石。

另外，理解这些原则不会立即让您成为优秀的程序员，也不意味着工作多年不会犯错。每一段代码都是从不完美开始的，通过走查不断趋于完美，就像黏土制作成陶艺一样，享受这个过程吧!


**[⬆ 回到顶部](#目录)**

## 变量

### 变量名要有意义

做有意义的区分，让读者更容易理解变量的含义。

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

### 对功能一致的变量采用统一命名

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

我们读代码要比写的多，所以易读性和可检索非常重要。如果不抽取并命名有意义的变量名，那就是坑读代码的人。代码要可检索，像[TSLint](https://palantir.github.io/tslint/rules/no-magic-numbers/) 就可以帮助识别未命名的常量。

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

### 避免思维映射

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

### 不添加无用的上下文

如果类名或对象名已经表达了某些信息，在内部变量名中不要再重复表达。

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

### 使用默认参数，而非短路或条件判断

通常，默认参数比短路更整洁。

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

限制参数个数，这样函数测试会更容易。超过三个参数会导致测试复杂度激增，需要测试众多不同参数的组合场景。
理想情况，只有一两个参数。如果有两个以上的参数，那么您的函数可能就太过复杂了。

如果需要很多参数，请您考虑使用对象。为了使函数的属性更清晰，可以使用[解构](https://basarat.gitbooks.io/typescript/docs/destructuring.html)，它有以下优点：

1. 当有人查看函数签名时，会立即清楚使用了哪些属性。

2. 解构对传递给函数的参数对象做深拷贝，这可预防副作用。(注意：**不会克隆**从参数对象中解构的对象和数组)

3. TypeScript 会对未使用的属性显示警告。

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
通过 TypeScript 的[类型别名](https://www.typescriptlang.org/docs/handbook/advanced-types.html#type-aliases)，可以进一步提高可读性。

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

这是目前软件工程中最重要的规则。如果函数做不止一件事，它就更难组合、测试以及理解。反之，函数只有一个行为，它就更易于重构、代码就更清晰。如果您只从本指南中了解到这一点，那么您就超过多数程序员了。

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

函数名就可以展示出函数实现的功能。

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

当有多个抽象级别时，函数应该是做得太多事了。拆分函数以便可复用，也让测试更容易。

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

重复乃万恶之源！重复意味着如果要修改某个逻辑，需要修改多处代码:cry:。
想象一下，如果你经营一家餐厅，要记录你的库存:所有的西红柿、洋葱、大蒜、香料等等。如果有多个库存列表，那维护起来是多么痛苦!

存在重复代码，是因为有两个或两个以上很近似的功能，只有一点不同，但是这些不同迫使你用多个独立的函数来做很多几乎相同的事情。删除重复代码，则意味着创建一个抽象，该抽象仅用一个函数/模块/类就可以处理这组不同的东西。

合理的抽象至关重要，这就是为什么您应该遵循[SOLID原则](#SOLID原则)。糟糕的抽象可能还不如重复代码，所以要小心！话虽如此，还是要做好抽象！尽量不要重复。

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

应该对复制代码持批评态度。有时，在重复代码和引入不必要的抽象而增加的复杂性之间，需要做权衡。当来自不同领域的两个不同模块的实现看起来相似，复制也是可以接受的，并且比抽取公共代码要好一点。因为抽取公共代码导致两个模块产生间接的依赖关系。


**[⬆ 回到顶部](#目录)**

### 使用`Object.assign`或`解构`来设置默认对象

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

为了避免副作用，不允许显式传递`undefined`或`null`值。参见 TypeScript 编译器的`--strictnullcheck`选项。

**[⬆ 回到顶部](#目录)**

### 不要使用Flag参数

Flag参数告诉用户这个函数做了不止一件事。如果函数使用布尔值实现不同的代码逻辑路径，则考虑将其拆分。

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

当函数产生除了“一个输入一个输出”之外的行为时，称该函数产生了副作用。比如写文件、修改全局变量或将你的钱全转给了一个陌生人等。

在某些情况下，程序确实需要有副作用。如先前例子中的写文件，这时应该将这些功能集中在一起，不要用多个函数/类修改某个文件。用且只用一个 service 完成这一需求。

重点是要规避常见陷阱，比如，在无结构对象之间共享状态、使用可变数据类型，以及不确定副作用发生的位置。如果你能做到这点，你才可能笑到最后！

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

在 JavaScript 中，原类型是值传递，对象、数组是引用传递。

有这样一种情况，如果您的函数修改了购物车数组，例如，要添加购买的商品，那么使用该购物车数组的任何其他函数都将受到此添加操作的影响。想象一个糟糕的情况:

用户点击“购买”按钮，该按钮调用购买函数，函数请求网络并将购物车数组发送到服务器。由于网络连接不好，购买功能必须不断重试请求。现在，如果用户在网络请求开始前，不小心点击了某个不想要的项目上的“Add to Cart”按钮，该怎么办？如果发生这种情况，并且网络请求开始，那么purchase函数将发送意外添加的项，因为它引用了一个购物车数组，addItemToCart函数通过添加不需要的项修改了该数组。

一个很好的解决方案是addItemToCart总是克隆购物车，编辑并返回购物车的克隆。这确保引用购物车的其他函数不会受到任何更改的影响。

注意两点:

1. 在某些情况下，您可能确实想要修改输入对象，这种情况非常少见。大多数可以重构，确保没副作用！(见[纯函数](https://en.wikipedia.org/wiki/Pure_function))

2. 性能方面，克隆大对象代价确实比较大。还好有一些很好的库，它提供了一些高效快速的方法，且不像手动克隆对象和数组那样占用大量内存。


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

在 JavaScript 中污染全局是一个非常不好的实践，这么做可能和其他库起冲突，且调用你的 API 的用户在实际环境中得到一个 exception 前对这一情况是一无所知的。

考虑这样一个例子：如果您想要扩展 JavaScript 的 `Array`，使其拥有一个可以显示两个数组之间差异的 diff 方法，该怎么做呢？可以将新函数写入`Array.prototype` ，但它可能与另一个尝试做同样事情的库冲突。如果另一个库只是使用`diff`来查找数组的第一个元素和最后一个元素之间的区别呢？

更好的做法是扩展`Array`，实现对应的函数功能。

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

### 函数式编程优于命令式编程

尽量使用函数式编程！

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

### 封装判断条件

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

### 避免“否定”的判断

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

### 避免判断条件

这看起来似乎不太可能完成啊。
大多数人听到后第一反应是，“没有if语句怎么实现功能呢？”在多数情况下，可以使用多态性来实现相同的功能。
接下来的问题是“为什么要这么做？”原因就是之前提到的：函数只做一件事。

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

TypeScript 是 JavaScript 的一个严格的语法超集，具有静态类型检查的特性。所以指定变量、参数和返回值的类型，以充分利用此特性，能让重构更容易。

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

现代浏览器在运行时进行大量的底层优化。很多时候，你做优化只是在浪费时间。这些优秀[资源](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers)可以帮助定位哪里需要优化，找到并修复它。

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

### 删除无用代码

无用代码和重复代码一样无需保留。如果没有地方调用它，请删除！如果仍然需要它，可以查看版本历史。

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

### 使用迭代器和生成器

Use generators and iterables when working with collections of data used like a stream.  
理由如下:

- decouples the callee from the generator implementation in a sense that callee decides how many
items to access
- lazy execution, items are streamed on demand
- built-in support for iterating items using the `for-of` syntax
- iterables allow to implement optimized iterator patterns

**反例:**

```ts
function fibonacci(n: number): number[] {
  if (n === 1) return [0];
  if (n === 2) return [0, 1];

  const items: number[] = [0, 1];
  while (items.length < n) {
    items.push(items[items.length - 2] + items[items.length - 1]);
  }

  return items;
}

function print(n: number) {
  fibonacci(n).forEach(fib => console.log(fib));
}

// Print first 10 Fibonacci numbers.
print(10);
```

**正例:**

```ts
// Generates an infinite stream of Fibonacci numbers.
// The generator doesn't keep the array of all numbers.
function* fibonacci(): IterableIterator<number> {
  let [a, b] = [0, 1];

  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

function print(n: number) {
  let i = 0;
  for (const fib in fibonacci()) {
    if (i++ === n) break;  
    console.log(fib);
  }  
}

// Print first 10 Fibonacci numbers.
print(10);
```

There are libraries that allow working with iterables in a similar way as with native arrays, by
chaining methods like `map`, `slice`, `forEach` etc. See [itiriri](https://www.npmjs.com/package/itiriri) for
an example of advanced manipulation with iterables (or [itiriri-async](https://www.npmjs.com/package/itiriri-async) for manipulation of async iterables).

```ts
import itiriri from 'itiriri';

function* fibonacci(): IterableIterator<number> {
  let [a, b] = [0, 1];
 
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

itiriri(fibonacci())
  .take(10)
  .forEach(fib => console.log(fib));
```

**[⬆ 回到顶部](#目录)**

## 对象和数据结构

### 使用`getters`和`setters`

TypeScript支持getter/setter语法。使用getter和setter从对象中访问数据可能比简单地在对象上查找属性要好。为什么？原因如下:

* 当您想要做更多的事情而不仅仅是获取对象属性时，您不必查找并更改代码中的每个访问器。*
* 使在执行集合时添加验证变得简单。*
* 封装内部表示。*
* 更容易添加日志和错误处理。*
* 您可以延迟加载对象的属性，比如从服务器获取它。*

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

### 让对象拥有 private/protected 成员

TypeScript 类成员支持 `public`*(默认)*、`protected` 以及 `private`的访问限制。

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

### 不变性

TypeScript 类型系统允许将接口、类上的单个属性设置为只读，能以函数的方式运行。

还有个高级场景，可以使用内置类型`Readonly`，它接受类型 T 并使用[映射类型](https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types)将其所有属性标记为只读。

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

### 类型 vs 接口

当您可能需要联合或交集时，请使用类型。如果需要扩展或实现，请使用接口。然而，没有严格的规则，只有适合的规则。

参考这个关于 Typescript 中`type`和`interface`区别的[解释](https://stackoverflow.com/questions/37233735/typescript-interfaces-vs-types/54101543#54101543) 

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


## 类

### 小、小、小！要事情说三遍

类的大小是由它的职责来度量的。按照*单一职责原则*，类要小。

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

内聚：定义了类成员之间相互关联的程度。理想情况下，高内聚类的每个方法都应该使用类中的所有字段，实际上这不可能也不可取。但我们依然提倡高内聚。

耦合：指的是两个类之间的关联程度。如果其中一个类的更改不影响另一个类，则称为低耦合类。

好的软件设计具有**高内聚性**和**低耦合性**。

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

正如”四人帮“在[设计模式](https://en.wikipedia.org/wiki/Design_Patterns)中所指出的那样，您尽可能使用组合而不是继承。组合和继承各有优劣。这个准则的主要观点是，如果你潜意识地倾向于继承，试着想想组合是否能更好地给你的问题建模，在某些情况下可以。  

什么时候应该使用继承？这取决于你手头的问题。以下场景使用继承比组合更好:

1. 继承代表的是“is-a”关系，而不是“has-a”关系 (人 -> 动物 vs. 用户 -> 用户详情)
2. 可复用基类的代码 (人类可以像所有动物一样移动)。
3. 希望通过更改基类对派生类进行全局更改。(改变所有动物在运动时的热量消耗)。

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

这个模式非常有用，在许多库中都可以看到。它让代码表达性更好，且不那么冗长，看起来更整洁。

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

正如 Clean Code 中所述，“类更改的原因不应该超过一个”。将很多功能打包在一个类看起来很诱人，就像在航班上您只能带一个手提箱。这样带来的问题是，在概念上类不具有内聚性，且有很多原因去修改类。而我们应该尽量减少修改类的次数。如果一个类中有很多的功能，修改了其中一处很难确定对代码库中其他依赖模块的影响。

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

正如 Bertrand Meyer 所说，“软件实体(类、模块、函数等)应该对扩展开放，对修改封闭。” 换句话说，就是允许在不更改现有代码的情况下添加新功能。

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

对一个非常简单的概念来说，这是个可怕的术语。
它的正式定义是：“如果 S 是 T 的一个子类型，那么类型 T 的对象可以被替换为类型 S 的对象，而不会改变程序的任何期望属性(正确性、执行的任务等)“。这是一个更可怕的定义。  

对此最好的解释是，如果您有一个父类和一个子类，那么父类和子类可以互换使用，而不会得到不正确的结果。这可能仍然令人困惑，所以让我们看一看经典的正方形矩形的例子。从数学上讲，正方形是矩形，但是如果您通过继承使用 “is-a” 关系对其建模，您很快就会遇到麻烦。

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

ISP 声明“客户不应该被迫依赖于他们不使用的接口。”这一原则与单一责任原则密切相关。否则会增加客户端的负担，因为他们需要实现一些不需要的方法。

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

这一原则有两个要点:
1. 高层模块不应该依赖于低层模块，两者都应该依赖于接口。
2. 接口应该脱离实现，实现应该依赖接口。

一开始这难以理解，但是如果你使用过 Angular，你就会看以依赖注入(DI)的方式实现了这一原则。虽然它们不是相同的概念，但是 DIP 阻止高级模块了解其低级模块的细节并进行设置。它可以通过 DI 实现这一点。这样做的一个巨大好处是减少了模块之间的耦合。耦合是一种非常糟糕的开发模式，它使代码难以重构。

DIP通常是通过使用控制反转(IoC)容器来实现的。比如：TypeScript 的IoC容器 [InversifyJs](https://www.npmjs.com/package/inversify)

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

测试比发货更重要。如果没有测试或数量不足，那么每次发布代码时都无法确保不引入问题。怎样才算是足够的测试数量？这取决于团队，但是拥有100%的覆盖率(所有语句和分支)会团队更有信心。这一切都要有好的测试框架以及覆盖率工具。

没有任何理由不编写测试。有很多优秀的JS测试框架都支持TypeScript，找一个你的团队喜欢的。然后总是为你引入的每个新特性/模块编写测试。如果您喜欢测试驱动开发(TDD)，那就太好了，重点是确保在开发任何特性或重构现有特性之前，代码已经达到了覆盖目标。 

### TDD（测试驱动开发）三定律

1. 在编写不能通过的单元测试前，不可编写生产代码。
2. 只可编写刚好无法通过的单元测试，不能编译也算不过。
3. 只可编写刚好足以通过当前失败测试的生产代码。

**[⬆ 回到顶部](#目录)**

### F.I.R.S.T.准则
整洁的测试应遵循以下准则:
- **快速**（Fast），测试应该快（及时反馈出业务代码的问题）。
- **独立**（Independent），每个测试流程应该独立。
- **可重复**（Repeatable），测试应该在任何环境上都能重复通过。
- **自我验证**（Self-Validating），测试结果应该明确*通过*或者*失败*。
- **及时**（Timely），测试代码应该在产品代码之前编写。

**[⬆ 回到顶部](#目录)**

### 单一的测试每个概念

测试也应该遵循*单一职责原则*。每个单元测试只做一个断言。

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

### 测试用例名称应该显示它的意图

当测试失败时，它的名称就能告诉你原因。

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

### 用 Promises 替代回调

回调不够整洁而且会导致过多的嵌套*(回调地狱)*。

有些工具可以将现有函数转换为promise对象：
- Node.js 参见[`util.promisify`](https://nodejs.org/dist/latest-v8.x/docs/api/util.html#util_util_promisify_original)
- 通用参见[pify](https://www.npmjs.com/package/pify), [es6-promisify](https://www.npmjs.com/package/es6-promisify)

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

Promises 有以下方法：

| 方法                     | 描述                                       |  
| ------------------------ | -----------------------------------------  |  
| `Promise.resolve(value)` | Convert a value into a resolved promise.   |  
| `Promise.reject(error)`  | Convert an error into a rejected promise.  |  
| `Promise.all(promises)`  | Returns a new promise which is fulfilled with an array of fulfillment values for the passed promises or rejects with the reason of the first promise that rejects. |
| `Promise.race(promises)`|Returns a new promise which is fulfilled/rejected with the result/error of the first settled promise from the array of passed promises. |


`Promise.all`在并行运行任务时尤其有用，`Promise.race`让为 Promises 实现超时变得更加容易。

**[⬆ 回到顶部](#目录)**

### `Async/Await` 比 `Promises` 更好

使用async/wait语法，您可以编写更简洁、更容易理解的链式promise的代码。在一个以async关键字为前缀的函数中，您可以告诉JavaScript运行时暂停wait关键字上的代码执行(当使用 promise 时)。

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

抛出错误是件好事!它们意味着运行时已经成功地识别出程序中的错误，并通过停止当前堆栈上的函数执行(在Node.js)终止进程以及在控制台中使用堆栈跟踪通知您来让您知道。

### 抛出`Error`或 reject `Error`

JavaScript和TypeScript允许你 `throw` 任何对象。Promise也可以用任何理由对象拒绝。
建议使用带有 `Error` 类型的 `throw` 语法。这是因为您的错误可能在具有 `catch` 语法的高级代码中被捕获。在那里捕获字符串消息会非常混乱，并且会使[调试更加痛苦](https://basarat.gitbooks.io/typescript/docs/types/exceptions.html#always-use-error)。出于同样的原因，您应该拒绝带有 `Error `类型的 promises。

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

使用 `Error` 类型的好处是 `try/catch/finally` 语法支持它，并且隐式地所有错误都具有  `stack` 属性，该属性对于调试非常强大。

还有另一种选择，即不使用 `throw` 语法，而总是返回自定义错误对象。TypeScript在这块更容易。考虑下面的例子:

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

关于这个想法的详细解释，请参考[原文](https://medium.com/@dhruvrajvanshi/making-exceptions-type-safe-in-typescript-c4d200ee78e9)。

**[⬆ 回到顶部](#目录)**

### 别忘了捕获错误

对捕获的错误不做任何处理并不会使您能够修复或对所述错误作出反应。将错误记录到控制台(console.log)也好不到哪里去，因为它常常会在打印到控制台的大量内容中丢失。如果您在     `try/catch` 中包装任何代码，这意味着您认为那里可能会发生错误，因此您应该有一个计划，或创建一个代码路径，以便在发生错误时使用。

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

### 不要忽略被拒绝的 promises

理由和不能在`try/catch`中忽略`Error`一样。

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

格式化是主观的。就像这里的许多规则一样，没有什么硬性规定是你必须遵守的。重点是不要争论格式。有很多工具可以实现自动化。使用一个! 对于工程师来说，争论格式是浪费时间和金钱的。遵循的一般规则是保持一致的格式规则。

For TypeScript there is a powerful tool called [TSLint](https://palantir.github.io/tslint/). It's a static analysis tool that can help you improve dramatically the readability and maintainability of your code. There are ready to use TSLint configurations that you can reference in your projects:

对于TypeScript，有一个强大的工具叫做TSLint。它是一个静态分析工具，可以帮助您显著提高代码的可读性和可维护性。已经准备好使用TSLint配置，您可以在您的项目中参考:

* [TSLint Config Standard](https://www.npmjs.com/package/tslint-config-standard) - standard style rules

* [TSLint Config Airbnb](https://www.npmjs.com/package/tslint-config-airbnb) - Airbnb style guide

* [TSLint Clean Code](https://www.npmjs.com/package/tslint-clean-code) - TSLint rules inspired be the [Clean Code: A Handbook of Agile Software Craftsmanship](https://www.amazon.ca/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882)

* [TSLint react](https://www.npmjs.com/package/tslint-react) - lint rules related to React & JSX

* [TSLint + Prettier](https://www.npmjs.com/package/tslint-config-prettier) - lint rules for [Prettier](https://github.com/prettier/prettier) code formatter

* [ESLint rules for TSLint](https://www.npmjs.com/package/tslint-eslint-rules) - ESLint rules for TypeScript

* [Immutable](https://www.npmjs.com/package/tslint-immutable) - rules to disable mutation in TypeScript

Refer also to this great [TypeScript StyleGuide and Coding Conventions](https://basarat.gitbooks.io/typescript/docs/styleguide/styleguide.html) source.

还可以参考这个很棒的[TypeScript风格指南和编码约定](https://basarat.gitbooks.io/typescript/docs/styleguide/styleguide.html)的源代码。

### 大小写一致

Capitalization tells you a lot about your variables, functions, etc. These rules are subjective, so your team can choose whatever they want. The point is, no matter what you all choose, just *be consistent*.

大写可以告诉你很多关于变量、函数等的信息。这些规则是主观的，所以你的团队可以选择他们想要的任何东西。关键是，无论你们选择什么，都要*一致*。

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

### 调用函数的函数和被调函数应靠近放置

当函数间存在相互调用的情况时，应将两者靠近放置。最好是应将调用者写在被调者的上方。这就像读报纸一样，我们都是从上往下读，那么读代码也是。

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

### 组织导入

使用整洁且易于阅读的`import`语句，您可以快速查看当前代码的依赖关系。导入语句应遵循以下做法:

- `Import`语句应该按字母顺序排列和分组。
- 应该删除未使用的导入语句。
- 命名导入必须按字母顺序(例如：`import {A, B, C} from 'foo';`)。
- 导入源必须在组中按字母顺序排列。 例如: `import * as foo from 'a'; import * as bar from 'b';`
- 导入组用空行隔开。
- 组内按照如下排序:
  - Polyfills (例如: `import 'reflect-metadata';`)
  - Node 内置模块 (例如: `import fs from 'fs';`)
  - 外部模块 (例如: `import { query } from 'itiriri';`)
  - 内部模块 (例如: `import { UserService } from 'src/services/userService';`)
  - 父目录中的模块 (例如: `import foo from '../foo'; import qux from '../../foo/qux';`)
  - 来自相同或兄弟目录的模块 (例如: `import bar from './bar'; import baz from './bar/baz';`)

**反例:**

```ts
import { TypeDefinition } from '../types/typeDefinition';
import { AttributeTypes } from '../model/attribute';
import { ApiCredentials, Adapters } from './common/api/authorization';
import fs from 'fs';
import { ConfigPlugin } from './plugins/config/configPlugin';
import { BindingScopeEnum, Container } from 'inversify';
import 'reflect-metadata';
```

**正例:**

```ts
import 'reflect-metadata';

import fs from 'fs';
import { BindingScopeEnum, Container } from 'inversify';

import { AttributeTypes } from '../model/attribute';
import { TypeDefinition } from '../types/typeDefinition';

import { ApiCredentials, Adapters } from './common/api/authorization';
import { ConfigPlugin } from './plugins/config/configPlugin';
```

**[⬆ 回到顶部](#目录)**

### 使用 typescript 别名

为了创建整洁漂亮的导入语句，可以在`tsconfig.json`中设置编译器选项的`paths`和`baseUrl`属性。

这样可以避免导入时使用较长的相对路径。

**反例:**

```ts
import { UserService } from '../../../services/UserService';
```

**正例:**

```ts
import { UserService } from '@services/UserService';
```

```js
// tsconfig.json
...
  "compilerOptions": {
    ...
    "baseUrl": "src",
    "paths": {
      "@services": ["services/*"]
    }
    ...
  }
...
```

**[⬆ 回到顶部](#目录)**


## 注释

写注释意味着没有注释就无法表达清楚，而最好用代码去表达。

> 不要注释坏代码，重写吧！— *Brian W. Kernighan and P. J. Plaugher*

### 代码自解释而不是用注释

代码即文档。

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

### 不要像写日记一样写注释

记住，使用版本控制！不需要保留无用代码、注释掉的代码，尤其像日记一样的注释。使用`git log`来获取历史。

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

### 避免使用注释标记位置

它们常常扰乱代码。要让代码结构化，函数和变量要有合适的缩进和格式。

另外，你可以使用支持代码折叠的IDE (看下 Visual Studio Code [代码折叠](https://code.visualstudio.com/updates/v1_17#_folding-regions)).

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

### TODO 注释 

当发现自己需要在代码中留下注释，以提醒后续改进时，使用`// TODO`注释。大多数IDE都对这类注释提供了特殊的支持，你可以快速浏览整个`TODO`列表。

但是，请记住**TODO**注释并不是坏代码的借口。

**反例:**

```ts
function getActiveSubscriptions(): Promise<Subscription[]> {
  // ensure `dueDate` is indexed.
  return db.subscriptions.find({ dueDate: { $lte: new Date() } });
}
```

**正例:**

```ts
function getActiveSubscriptions(): Promise<Subscription[]> {
  // TODO: ensure `dueDate` is indexed.
  return db.subscriptions.find({ dueDate: { $lte: new Date() } });
}
```

**[⬆ 回到顶部](#目录)**

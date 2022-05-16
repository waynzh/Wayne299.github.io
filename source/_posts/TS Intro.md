---

title: TypeScript Intro
date: 2021-8-17 4:42
category: FE
tags: TS 

---

# 基础类型
## 数组
1. `let list: number[] = [1, 2, 3]`
2. 使用数组泛型，Array<元素类型>：`let list: Array<number> = [1, 2, 3]`

## 元组 Tuple
- 表示一个已知元素**数量**和**类型**的数组: `let tuple: [number, string] = [1, "2"]`
- 访问越界的元素，会使用联合类型替代：`tuple[1].toString()` 

## 枚举 enum
<!--more-->
- 用一组数值赋予友好的名字代替: 默认以0开始编号
```ts
enum Color {Red = 1, Green, Blue}
let c: Color = Color.Green; // 2
```

- 也可以通过枚举的值得到名字
```ts
enum Color {Red = 1, Green, Blue}
let colorName: string = Color[2]; // Green
```

## void / undefined / null
- 没有任何类型: function 不返回时
- void类型的变量只能声明 `undefined`和`null`
- `undefined`和`null` 可以是任意类型的子类型

## Never
- 永不存在的值的类型: 例如
  - 死循环 / 总是会抛出异常 
- 只有never类型可以给never类型赋值；never是任意类型的子类型


## 类型断言
1. “尖括号”: `let strLength: number = (<string>someValue).length;`
2. `as`: `let strLength: number = (someValue as string).length;`

## 解构
- 注意必填项 和 默认值
```js
function f({ a, b = 0 } = { a: "" }): void {
    // ...
}
f(); // ok, default to {a: ""}, which then defaults b = 0
f({}); // error, 'a' is required if you supply an argument
```

## 扩展
- 扩展可枚举属性，意味着**方法会丢失**
- 在对象中，顺序在后的会覆盖前面的


# 接口 interface
## 类型检查的替换
- 多余属性检查，防止使用不属于接口的属性和方法
```ts
function printLabel(labelledObj: { label: string }) {
  console.log(labelledObj.label);
}

// 用接口 interface替换
interface labelledValue {
    label: string;
}

function printLable(labelledObj: labelledValue) {
    console.log(labelledObj.label)
}
```

## 可选属性 `?:`
## 只读属性 `readonly x: number`
- 在**对象初次创建**的时候修改其值，※不是初次赋值
- 把整个`ReadonlyArray<T>`赋值到一个普通数组也不可以，除非使用类型断言

## 额外属性 检查
- 如果将额外的属性 赋值给变量或作为参数传递传入, 会得到一个错误
  - 用类型断言就可以绕开检测: `{} as labelledValue`
  - 可以添加一个字符串索引签名：`[propName: string]: any`
  - 直接传一个变量 不会经过额外属性检查

## 函数类型
- 接口除了描述带有属性的对象外，也可以描述函数类型
- 要求**对应位置**上的参数类型是兼容的
- 函数定义时不想指定类型，TS也会推断出参数类型

```ts
interface SearchFunc {
    (source: string, subString: string): boolean;
}

let mySearch : SearchFunc;
mySearch = function(src, sub) {
    let result = src.search(sub);
    return result > -1;
}
```

## 可索引的类型
- 描述对象**索引的类型**，还有相应的**索引返回值类型**。
- 数字索引的返回值 ∈ 字符串索引返回值（是字符串返回值的子类型）


```ts
interface StringArray {
    readonly [index: number]: string;
}

let strArr: StringArray;
strArr = ["wayne", "zhang"]
let myName:string = strArr[0] + strArr[1];
```

- 给索引签名设置为只读，防止给其赋值
- ∵ 索引可以是字符串或者数字，所以避免index的返回值和其他类型冲突
```ts
interface NumberDictionary {
  readonly [index: string]: number;
  length: number;    // 可以，length是number类型 且 必有
  name: string       // obj["name"] 错误，`name`的类型与索引类型返回值的类型不匹配 
}
```

## 类类型
### 实现接口 `implements`
- 强制一个类去符合某种契约: 接口描述了类的**公共（实例）部分**(定义值类型 或者 描述一个方法)
```ts
interface ClockInterface {
    currentTime: Date;
    setTime(d: Date);
}

class myClock implements ClockInterface {
    currentTime: Date;
    setTime(d: Date) {
        this.currentTime = d;
    }
    constructor(h: number, m: number){}
}
```

### 静态部分 和 实例部分
- 不能直接检查私有部分，比如constructor。∴ 用一个构造函数**接受类**来创建实例 或者 class expression。
```ts
interface ClockConstructor {
  new (hour: number, minute: number);
}

interface ClockInterface {
  tick();
}
// class expression
const Clock: ClockConstructor = class Clock implements ClockInterface {
  constructor(h: number, m: number) {}
  tick() {
    console.log("beep beep");
  }
};

let c =new Clock(12,4);
```

## 继承接口 `extends`
- 和类一样，接口也可以相互继承。方便**从一个接口**里复制成员**到另一个接口里**。
- 一个接口可以继承多个接口
```ts
interface Shape {
    color: string;
}

interface Stock {
    stock: number;
}

interface Square extends Shape, Stock {
    sideLength: number;
}

let square = <Squre>{}; // square只接收 `{} as Square`
square.color = "b";
square.sideLength = 3;
```

## 混合类型
- 想要一个对象可以同时做为函数和对象使用，并带有额外的属性: 定义一个函数进行处理
```ts
interface Counter {
    (start: number): string;
    interval: number;
    reset(): void;
}

function getCounter(): Counter {
    let counter = function (start: number) {...} as Counter;
    counter.interval = 1;
    counter.reset = function() {...};
    return counter;
}

let c = getCounter();
```

## 接口继承类
- 当接口继承类类型时，会继承类的**成员**但不包括其**实现**
- 约等于：接口申明了所有成员但不提供实现
- 接口在类中有`private`和`protected`时也会继承，但这个接口只有该类和其子类才能实现(implements)
```js
class Control {
    private state: any;
}

interface SelectableControl extends Control {
    select(): void;
}

// 是其子类，所以可以实现
class Button extends Control implements SelectableControl {
    select() { }
}

// 错误：“Image”类型缺少“state”属性。
class Image implements SelectableControl {
    select() { }
}
```

# 类 class
- 继承需要：`super()` 和 `super.func()`
```ts
class Animal {
    name: string;
    construtor(theName: string) {
        this.name = theName;
    }
    move(distance: number) {
        console.log(`${this.name} moved ${distance}m.`);
    }
}

class Snake extends Animal {
    constructor(name: string) {
        super(name);
    }
    move(distance = 5) {
        console.log("Snake is moving~");
        super.move(distance);
    }
}
```

## private / protected
- 当存在 `private / protected` 时，
  - 只有当另外一个类型中也存在这样一个 `private`成员，并且都是**来自同一处声明**才认为这两个类型是兼容的. 在外部不能访问
  -  `protected`成员在**派生类中**(`extends Animal`)仍然可以访问。构造函数也可以protected，不能在类外(包含自身)被实例化，但是能被继承的类实例

## 静态属性 static
- 存在于类本身上面而不是类的实例上. 每个实例想要访问这个属性时，都要在加上类名（类似于`this.`）

```ts
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistance(point: {x: number; y: number;}){
        /** 定义对象结构
         * point: {
         *      x: number;
         *      y: number;
         *  } 
        */
        let xDist = point.x - Grid.origin.x;
        let yDist = point.y - Grid.origin.y;
        return Math.sqrt(xDist**2 + yDist**2) / this.scale;
    }
    // 参数属性: 等于this.number = number
    constructor(public scale: number) {}
}

let grid1 = new Grid(1.0);
grid1.calculateDistance({x: 10, y:20})
```

## 抽象类 abstract
- 一般不会直接被实例化，专门用来**被继承的类**
- 可以包含成员的实现细节。但抽象类中的抽象方法还是不包含具体实现 并且 必须在派生类中实现
- 方法在声明的抽象类中不存在：报错

# 函数
## 可选 ?:
## 默认值 
- 没填 或者 填`undefined`时: 默认值
- 可选

## this参数在回调函数里 ?
## 重载
```ts
function pickCard(x: {suit: string; card: number; }[]): number;
function pickCard(x: number): {suit: string; card: number; };
// any 不是重载列表的一部分，因此这里只有两个重载 
function pickCard(x): any {
    if (typeof x == "object") {
       // ...
    }
    // Otherwise just let them pick the card
    else if (typeof x == "number") {
       // ...
    }
}
```

# 泛型
- 是一种 **类型变量**
```ts
// 比 any多了一个信息：输入和输出的类型一样
function identity<T>(arg: T): T {
    return arg;
}

// 1. 传入所有的参数 类型参数 + 函数参数
let output = identity<string>("isString")
// 2. 类型推论：自动确定 T 的类型
let output = identity("isString")
```

- 为了提高灵活性，在操作数组时，可以规定T是数组中的基础类型，而不是整个类型
```ts
function arrayFunc<T> (arg: T[]): T[] {...}
function arrayFunc<T> (arg: Array<T>): Array<T> {...}
```

## 泛型类型
```ts
let myIdentity: <U>(arg: U) => U = identity;

// 调用签名的对象字面量来定义泛型函数
let myIdentity: {<T>(arg: T): T} = identity;
```

### 泛型接口
1. 将对象字面量 当作接口
2. 把**泛型参数**当作整个接口的一个参数: `interface Iidentity<T> {}`
```ts
// 1.
interface GenericIdentityFn {
    <T>(arg: T): T;
}
let myIdentity: GenericIdentityFn = identity;

// 2. 传入泛型参数 这里是 number
interface GenericIdentityFn<T> {
    (arg: T): T;
}
let myIdentity: GenericIdentityFn<number> = identity;
```

### 泛型类
- 泛型类指的是**实例部分**的类型, 静态属性不能使用这个泛型类型
```ts
class GenericNumber<T> {
    zeroValue: T;
    add: (x: T, y: T) => T;
}

let myGenericNumber = new GenericNumber<number>();
```

## 泛型约束
- 定义一个接口来描述约束条件：比如包含 `.length`属性的接口
```ts
interface Lengthwise {
    length: number;
}

function loggingIdentity<T extends Lengthwise>(arg: T): T {
    // no more error for `.length`
    console.log(arg.length); 
    return arg;
}

loggingIdentity({length: 10, value: 3});
loggingIdentity("asd");
```

# 枚举
- 不带初始化的枚举不能放在非常量枚举之后
  - 枚举表达式字面量（数/字符串）
  - 常量枚举成员的引用
  - 一元运算符和二元运算符 
  - 带括号的常量枚举表达式
- 不会为字符串枚举成员生成反向映射

## const枚举
- 在编译阶段会被删除, 只在使用的地方会被内联进来
```ts
const enum Enum {
    A = 1,
    B = A * 3,
    X
}

function f(obj: { X: number }) {
    return obj.X;
}

console.log(Enum["X"]); // 4
f(Enum) // error
```

## 外部枚举
- 对于非常数的外部枚举而言，没有初始化方法时被当做需要经过计算的。

# 类型兼容性
- TypeScript里的类型兼容性是基于**结构子类型**的，而不是基于名义类型

## 原始类型和对象类型
- 检查y是否能赋值给x：编译器检查x中的每个属性，看是否能在y中也找到对应属性
- 检查函数参数，只要有目标类型的成员即可
```ts
interface Named {
    name: string;
}
let x: Named;
// y包含x 就能赋值给x
let y = { name: 'Alice', location: 'Seattle' };

x = y; // OK
```

## 两个函数
### 参数
- 相反：y的每个参数必须能在x里找到对应类型的参数
```ts
let x = (a: number) => 0;
let y = (b: number, s: string) => 0;

y = x; // OK： 允许忽略参数
x = y; // Error: y必须有第两个参数，但x没有
```

### 返回值
- 源函数的返回值类型(x) 必须是 目标函数返回值类型(y)的子类型
```ts
let x = () => ({name: 'Alice'});
let y = () => ({name: 'Alice', location: 'Seattle'});

x = y; // OK
```

## 两个类
- 与对象字面量相似，`x=y`
- 只有实例的成员会被比较。 静态成员和构造函数不在比较的范围内。

### private / protected
- 如果包含一个私有成员，那么源类型必须包含来自同一个类的这个私有成员
- 只能允许子类赋值给父类，但不能赋值给其它有同样类型的类

# 交叉类型 `&` / 联合类型 `|`
```ts
function extend<T, U>(first: T, second: U): T & U {
    let result = <T & U>{};
    // 。。。
}
```

# 类型保护
## 用户自定保护
- 定义一个函数，它的返回值是一个 **类型谓词**：`parameterName is Type`

```ts
function isFish(pet: Fish | Bird): pet is Fish {
    return (<Fish>pet).swim() !== undefined;
}

if(isFish) {
    pet.swim(); // 这个pet是 <Fish>
} else {
    pet.fly(); // 类型保护还知道 这个pet是 <Bird>
}

// 简化
if((<Fish>pet).swim) {
    (<Fish>pet).swim();
} else {
    (<Bird>pet).fly()
}
```

## `typeof` 保护
- 不必抽象成一个函数，直接在代码里用也可以识别成类型保护
- `typeof v === "typename"和 typeof v !== "typename"`
```ts
function padLeft(value: string, padding: number | string) {
    if(typeof padding === "number") {
        // Array(3): 三个空数组用“ ”连接 → 两个“ ”
        return Array(padding + 1).join(" ") + value;
    } 
    if(typeof padding === "string") {
        return padding + value;
    }
    throw new Error("Expected string or number");
}
```

## `instanceof`保护
- `instanceof` 的右侧要求是一个构造函数
```ts
interface Padder {
    getPaddingString(): string
}

class SpaceRepeatingPadder implements Padder {
    constructor(private numSpaces: number) { }
    getPaddingString() {
        return Array(this.numSpaces + 1).join(" ");
    }
}

class StringPadder implements Padder {
    constructor(private value: string) { }
    getPaddingString() {
        return this.value;
    }
}

// 类型为SpaceRepeatingPadder | StringPadder
let padder: Padder = getRandomPadder();

if (padder instanceof SpaceRepeatingPadder) {
    padder; // 类型细化为'SpaceRepeatingPadder'
}
if (padder instanceof StringPadder) {
    padder; // 类型细化为'StringPadder'
}
```

## 去除null
- 短路运算`||` 或者 `??`: `name = name || "default"`
- 添加 `!` 后缀: `name = name!`

# 类型别名 `type`
- 类型别名有时和接口很像，但是可以作用于原始值，联合类型，元组以及其它任何你需要手写的类型。
- 创建了一个新名字来**引用类型**
- 别名不能被 `extends`和 `implements`（但可以通过交叉类型`&`来实现）
```ts
type Tree<T> = {
    value: T;
    left: Tree<T>;
    right: Tree<T>;
}
```

## 与接口的不同点
- 接口只能声明对象，无法描述一个基本类型或者类型表达式，需要使用联合类型或元组类型时，就需要使用`type`
- 接口会检查扩展的接口是否可以赋值。`&`会全部组合在一起
- 接口可以定义多次，多次的声明会合并。但是`type`定义多次会报错

## 可辨识联合 （标签）
- 添加完整性检查
```ts
interface Square {
    kind: "square";
    size: number;
}
interface Rectangle {
    kind: "rectangle";
    width: number;
    height: number;
}
interface Circle {
    kind: "circle";
    radius: number;
}

// kind是普通的单例类型属性：标签
type Shape = Square | Rectangle | Circle;
// 完整性检查: never
function assertNever(x: never): never {
    throw new Error("Unexpected object: " + x);
}

function area(s: Shape) {
    // 利用标签完成操作
    switch (s.kind) {
        case "square": return s.size * s.size;
        case "rectangle": return s.height * s.width;
        case "circle": return Math.PI * s.radius ** 2;
        default: return assertNever(s); // error here if there are missing cases
    }
}
```

## 字符串字面量
```ts
// 三种允许的字符中选择其一
type Easing = "ease-in" | "ease-out" | "ease-in-out";
```

# 索引类型
## 索引类型查询操作符 `keyof T`
- 结果为 T上已知的**公共属性名的联合**
- `'name' | 'age'` 就等同于`keyof Person`,但后者会自动更新
```ts
interface Person {
    name: string;
    age: number;
}

let personProps: keyof Person; // 'name' | 'age'
```

## 索引访问操作符 `T[K]`
- `K extends keyof T`后即可 `T[K]`
```ts
function getProperty<T, K extends keyof T>(o: T, name: K): T[K] {
    return o[name]; // o[name] is of type T[K]
}
```

```ts
interface Map<T> {
    [key: string]: T;
}
let keys: keyof Map<number>; // string
let value: Map<number>['foo']; // number
```

# 映射类型
- 新类型以相同的形式去转换旧类型里每个属性

```ts
type Options = option1 | option2;
type Flags = {
    [option in Options]: boolean;
}
// 等于
type Flags = {
    option1: boolean;
    option2: boolean;
}
```

- 类似索引签名的语法 + for ... in
- Readonly， Partial和 Pick是同态的，但 Record不是
  > 同态的在添加任何新属性之前可以拷贝属性修饰符。 例如，假设 `Person.name`是只读的，那么 `Partial<Person>.name`也将是只读的且为可选的。 
```ts
// 都变为只读的
type Readonly<T> = {
    readonly [P in keyof Person]: Person[P];
}
// 都变为可选的
type Partial<T> = {
    [P in keyof Person]?: Person[P];
}
// 将某个类型中的子属性挑出来
type Pick<T, K extends keyof T> = {
    [P in K]: T[P];
}
// 所有的属性的值转化为 T 类型
type Record<K extends string, T> = {
    [P in K]: T;
}

type PersonPartial = Partial<Person>;
```

## 预定义的有条件类型

# 导入/导出
```ts
// 导出
export default class MyClass;
export {OtherClass as OC} from "./OtherClass";

// 导入
import { ZipCodeValidator } from "./ZipCodeValidator";
```

## `export =` 和 `import = require()`
```ts
// 导出
export = MyClass;
// 导入
import zip = require("./ZipCodeValidator");
```

# 命名空间
- 接口和类在命名空间之外也是可访问的，所以需要使用 export。 
- 变量是实现的细节，不需要导出，在外不可访问。

## 多文件中的命名空间
- 可以同名，加引用标签告诉编译器文件之间的关联
```ts
// Validation.ts
namespace Validation {
    export interface StringValidator {
        isAcceptable(s: string): boolean;
    }
}

// LettersOnlyValidator.ts
/// <reference path="Validation.ts" />
namespace Validation {
    const lettersRegexp = /^[A-Za-z]+$/;
    export class LettersOnlyValidator implements StringValidator {
        isAcceptable(s: string) {
            return lettersRegexp.test(s);
        }
    }
}
```

## 别名 `import q = x.y.z`
- 为指定的符号创建一个别名。和 `import q = require("./")`语法不同

# 声明合并
## 合并接口
- 把双方的成员放到一个同名的接口里
- 同名问题：
  - 同名非函数成员类型不同会报错
  - 同名函数会被当作重载，之后的接口优先级更高
  - 字符串字面量会被提升

## 合并命名空间
- 之后的命名空间的**导出成员**会被加到已经存在的那个模块里。
- 非导出成员仅在其原有的（合并前的）命名空间内可见。从其他合并进来的不可访问
  
## 命名空间 与 类和函数和枚举类型合并
- 类似于 类和函数和枚举类型的扩展

## 非法合并
- **类** 不能与 其它类或变量合并

## 模块扩展
```ts
// observable.ts
export class Observable<T> {...}

// 全局扩展
declare global {
    interface Array<T> {
        toObservable(): Observable<T>;
    }
}

Array.prototype.toObservable = function () {
    // ...
}
```
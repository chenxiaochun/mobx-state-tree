# 入门指南

本教程将通过构建一个能够给用户分配待办事项的 TODO 应用来为你介绍 mobx-state-tree (MST) 的基础知识。

## 预备知识
开始此教程之前，我假设你已经对 React 有了基础的了解。否则，建议首先去阅读一下这份 [React 教程](https://facebook.github.io/react/tutorial/tutorial.html)。

### 我需要提前学习 MobX 吗?
MST 深度依赖于 MobX。因此，如果你使用过 MobX 的话，将对你处理一些复杂的情况和怎样把数据与 React 组件连接起来非常有帮助。如果你没有使用过 MobX 也没关系，因为使用 MST 不需要任何 MobX API 方面的知识。

## 如何跟随这门教程
你可以使用 [codesandbox](https://codesandbox.io/) 在浏览器中直接写代码，也可以使用你偏爱的某种编辑器（例如：VSCode）都是可以的。

### 在浏览器中写代码
我们为每一个实例都提供了 codesandbox 链接。你可以按照知识点一步接一步的去学习教程，如果在什么地方卡住了，可以直接参考下一个实例链接哈 :)。

### 在编辑器中写代码
配置 React 项目的开发环境是一件让人觉得很繁琐的事情，你可能需要引入各种编译器、打包器等等。幸好有`create-react-app`这样的脚手架使这种事情简单到只需要在终端中输入几条命令即可。

```
npm install -g create-react-app
create-react-app mst-todo
```
你需要安装一下 mobx、mobx-react 和 mobx-state-tree。
```
npm install mobx mobx-react mobx-state-tree --save
```
然后执行`npm run start`之后，一个基础的 React 页面就会展示在你面前。

## 概述
mobx-state-tree is a state container that combines the simplicity and ease of mutable data with the traceability of immutable data and the reactiveness and performance of observable data.

如果你对这个说明很困惑，不用担心。让我们一起一步步地来探索它。

## 入门指南
When building applications with MST, the first exercise that will help you building your application is thinking which is the minimal set of entities and their relative attributes of our application.

在我们的示例应用中处理的是 todo 代办事项，因此我们需要一个 todo 实体，它包含 name 和 todo 两个属性，并且 todo 属性有两种状态：完成与未完成。我们还需要一个 user 实体，用来分配代办事项。

## 创建第一个 model
MST 的最主要概念就是一个动态树。这个树结构由可变的，但是受严格的运行时类型信息保护的对象组成。换句话说，每个树都是由模型（类型信息）和状态（数据）组成的。从这个动态树中可以自动生成不可变的、结构上共享的快照。

这意味着如果想让应用运行起来，我们需要向 MST 描述清楚我们的实体模型。知道了这些信息，MST 就可以自动为我们生成所有的边界，以避免犯一些愚蠢的错误。比如，把字符串赋值给了价格字段或者把布尔值赋值给了本来期望是数组的字段。

使用 MST 定义 model 实体的一个最简单方式就是提供一个将来会被用作默认值的数据给 types.model 方法。

```javascript
import { types } from "mobx-state-tree"

const Todo = types.model({
    name: "",
    done: false
})

const User = types.model({
    name: ""
})
```
[View sample in playground](https://codesandbox.io/s/98x2n959ky)

上面的代码会创建两种类型数据，分别是 User 和 Todo 类型。但我们前面说过，一个 model tree 是由类型信息和状态组成的，那我们怎样定义一个 Todo 和 User 类型的实例呢？

## 创建 model 实例 (tree nodes)
可以很容易的在 User 和 Todo 类型上调用`.create()`方法来完成这件事情。

```javascript
import { types } from "mobx-state-tree"

const Todo = types.model({
    name: "",
    done: false
})

const User = types.model({
    name: ""
})

const john = User.create()
const eat = Todo.create()

console.log("John:", john.toJSON())
console.log("Eat TODO:", eat.toJSON())

```
[View sample in playground](https://codesandbox.io/s/6jo1o9n9qk)

正如你所见，使用 models 可以确保所有被定义字段都是预先定义过的默认值。但如果你想在创建 model 实例时改变它的值，可以简单的给 create 方法传递一个对象即可。

```javascript
const eat = Todo.create({ name: "eat" })

console.log("Eat TODO:", eat.toJSON()) // => will print {name: "eat", done: false}
```
[View sample in playground](https://codesandbox.io/s/ymqpj71oj9)

## 遇见 types
当你运行下面代码时，发现它会抛出错误异常：

```javascript
const eat = Todo.create({ name: "eat", done: 1 })
```
```
Error: [mobx-state-tree] Error while converting `{"name":"eat","done":1}` to `AnonymousModel`:
at path "/done" value `1` is not assignable to type: `boolean`.
```

这是因为 MST 的节点都是强类型的，你不能给一个布尔类型提供一个数值类型的值。这种特性对构建应用非常有好处，可以保证你的状态类型始终如一，不会有非法的状态类型插入进来。下面是定义 model 的一种快捷方式。

```javascript
const Todo = types.model({
    name: types.optional(types.string, ""),
    done: types.optional(types.boolean, false)
})

const User = types.model({
    name: types.optional(types.string, "")
})
```
[View sample in playground](https://codesandbox.io/s/j27j41828v)

MST 中的命名空间类型还内置了很多实用的类型，例如：array、map、maybe、refinements 和 unions。如果你对他们感兴趣，可以去查阅 api 文档。我们现在将 types 和定义的一个 RootStore 结合起来用于约束 users 和 todos。

注意：如果你没有给`type`的`create`方法传递值，那么`types.optional`的第二个参数值就不能省略。

```js
const User = types.model({
    name: types.optional(types.string, "")
})
```

比如在上面这段代码中，我们给`name`属性定义的是`string`类型值，假如你省略`optional`函数的第二个参数之后。`name`的默认值就变成了 undefined ，并不是`string`类型，代码也就自然会抛出异常了。

例如，你想在调用`create`时让 Todo 的 name 属性值成为必填项，可以去掉`optional`方法并且把`types.string`传进去就可以了。

```javascript
import { types } from "mobx-state-tree"

const Todo = types.model({
    name: types.optional(types.string, ""),
    done: types.optional(types.boolean, false)
})

const User = types.model({
    name: types.optional(types.string, "")
})

const RootStore = types.model({
    users: types.map(User),
    todos: types.optional(types.map(Todo), {})
})

const store = RootStore.create({
    users: { } // users is required here because it's not marked as optional
})
```
[View sample in playground](https://codesandbox.io/s/5wyx1xvvj4)

## 修改数据
MST 的树节点（也就是 model 实例）可以使用 action 来修改它。可以很容易的通过在`types`上调用`action`方法，给它传递一个回调函数，回调函数的参数为 model 实例，然后在回调函数内部将修改之后的 model 实例返回就可以了。 

如下示例中，在 Todo 类型上定义了`action`方法，你就可以在它里面通过提供的 Todo 实例来切换`done`的状态和设置`name`属性值。

```javascript
const Todo = types.model({
    name: types.optional(types.string, ""),
    done: types.optional(types.boolean, false)
}).actions(self => {
    function setName(newName) {
        self.name = newName
    }

    function toggle() {
        self.done = !self.done
    }

    return {setName, toggle}
})

const User = types.model({
    name: types.optional(types.string, "")
})

const RootStore = types.model({
    users: types.map(User),
    todos: types.optional(types.map(Todo), {})
}).actions(self => {
    function addTodo(id, name) {
        self.todos.set(id, Todo.create({ name }))
    }

    return {addTodo}
})
```
[View sample in playground](https://codesandbox.io/s/928l6pw7pr)

这里你需要知道，`self`的对象结构就是由你创建的 model 实例构成的。并且`action`方法已经被正确地绑定了`this`作用域，所以`self`指向的就是上面定义的 model 实例。

通过调用这些`action`方法就可以如此简单的实现 js 类，然后通过一个 model 实例来调用它们的方法了。

```javascript
store.addTodo(1, "Eat a cake")
store.todos.get(1).toggle()
```
[View sample in playground](https://codesandbox.io/s/928l6pw7pr)

## 强大的快照！
Dealing with mutable data and objects makes it easy to change data on the fly, but on the other hand it makes testing hard. Immutable data makes that very easy.  Is there a way to have the best of both worlds? Nature is a great example of that. Beings are living and mutable, but we may eternalize nature's beauty by taking awesome snapshots. Can we do the same with the state of our application?

Thanks to MST's knowledge of models and relative property types, MST is able to generate serializable snapshots of our store! You can easily get a snapshot of the store by using the `getSnapshot` function exported by the MST package.

```javascript
console.log(getSnapshot(store))
/*
{
    "users": {},
    "todos": {
        "1": {
            "name": "Eat a cake",
            "done": true
        }
    }
}
*/
```

Note: The `.toJSON()` you have used before in the tutorial is just a shortcut to `getSnapshot`!

Because the nature of state is mutable, a snapshot will be emitted whenever the state is mutated! To listen to those new snapshot, you can use `onSnapshot(store, snapshot => console.log(snapshot))` to log them as they are emitted!

## 从快照到 model
正如我们所见，从一个 model 实例获取一个快照是相当的简单，但是能不能很灵巧的把一个快照恢复成一个 model 呢？当然是可以的。

That basically means that you can restore your objects with your custom methods by just knowing the type of the tree and its snapshot! 你有两种方法可以完成这个操作。

第一种方法就是创建一个新的 model 实例，然后把快照作为参数传递给它。That means that you will need to update all your store references, if used in React components, to the new one.

第二种方法是通过把快照应用到一个已存在的 model 实例上，来避免出现这种引用问题。属性会被更新，但是存储引用不会发生变化。这将会触发一个被称为“调和”的操作，我们在后面会讨论这个。

```javascript
// 1st
const store = RootStore.create({
    "users": {},
    "todos": {
        "1": {
            "name": "Eat a cake",
            "done": true
        }
    }
})

// 2nd
applySnapshot(store, {
    "users": {},
    "todos": {
        "1": {
            "name": "Eat a cake",
            "done": true
        }
    }
})
```
[View sample in playground](https://codesandbox.io/s/xjm99kkopp)

## 时间旅行
获取和应用快照的能力使得用户能够很容易的实现时间旅行。你只需要基于快照的监听，存储并重新应用他们就可以实现时间旅行。

下面是一个简单的实现：

```javascript
import { applySnapshot, onSnapshot } from "mobx-state-tree"

var states = []
var currentFrame = -1

onSnapshot(store, snapshot => {
    if (currentFrame === states.length - 1) {
        currentFrame++
        states.push(snapshot)
    }
})

export function previousState() {
    if (currentFrame === 0) return
    currentFrame--
    applySnapshot(store, states[currentFrame])
}

export function nextState() {
    if (currentFrame === states.length - 1) return
    currentFrame++
    applySnapshot(store, states[currentFrame])
}
```

## 和界面关联起来
MST 完全兼容 MobX 的 autorun、reaction、observe 等功能特性。你可以使用 mobx-react 将 MST 的 store 和 React 组件关联起来，更多详细信息可查阅 mobx-react 的文档。MST 可以很容易的和任何视图引擎整合起来，只要对快照进行监听并进行相应的更新即可。

```javascript
const App = observer(props => <div>
    <button onClick={e => props.store.addTodo(randomId(), 'New Task')}>Add Task</button>
    {props.store.todos.values().map(todo =>
        <div>
            <input type="checkbox" checked={todo.done} onChange={e => todo.toggle()} />
            <input type="text" value={todo.name} onChange={e => todo.setName(e.target.value)} />
        </div>
    )}
</div>
)
```
[View sample in playground](https://codesandbox.io/s/r54o5pp8z4)

## 提高渲染性能
如果你已经安装了 React 开发工具，并且勾选上“Highlight Updates”选项，你会发现无论是切换 todo 的状态还是修改它的名称都会导致整个应用被重新渲染。这个太丢人了，因为如果你的列表中存在大量 todo 的话，就会产生性能问题。

MobX 提供了颗粒级的更新能力，可以很容易的修复此类问题。你只需要把单个的 todo 渲染分离到一个单独的组件里，这时再改变 todo 的数据就只会重新渲染此单独组件了。

```javascript
const TodoView = observer(props =>
        <div>
            <input type="checkbox" checked={props.todo.done} onChange={e => props.todo.toggle()} />
            <input type="text" value={props.todo.name} onChange={e => props.todo.setName(e.target.value)} />
        </div>)

const AppView = observer(props =>
        <div>
            <button onClick={e => props.store.addTodo(randomId(), 'New Task')}>Add Task</button>
            {props.store.todos.values().map(todo => <TodoView todo={todo} />)}
        </div>
)
```
[View sample in playground](https://codesandbox.io/s/m3rw1wll79)

经过改造之后，基本上每一个`observer`声明会促使只有被观测的数据改变之后，相对应的 React 组件才会被重新渲染。可是 App 组件依然在观测着所有事情，所以无论你修改什么，它还是会被重新渲染。

现在我们把渲染逻辑分离到一个单独的`observer`中，这时 todo 就只会在它本身数据有变化时才会重新渲染。而 App 组件也只是在添加/删除 todo 时才会被重新渲染，因为它现在只是在观测 todo 的长度而已。

## 计算属性
我们想在应用里展示所有 todo 的数目，以便让用户知道还有哪些 todo 还未完成。这就意味着我们要计算出那些“done”属性被设置为 false 的 todo 数量。因此，我们要修改一下 RootStore 声明，并且通过调用`.views`方法去添加 get 属性，以便能够计算出哪些 todo 还未完成。

```javascript
const RootStore = types.model({
    users: types.map(User),
    todos: types.optional(types.map(Todo), {}),
}).views(self => ({
    get pendingCount() {
        return self.todos.values().filter(todo => !todo.done).length
    },
    get completedCount() {
        return self.todos.values().filter(todo => todo.done).length
    }
})).actions(self => {
    function addTodo(id, name) {
        self.todos.set(id, Todo.create({ name }))
    }

    return {addTodo}
})
```
[View sample in playground](https://codesandbox.io/s/x3qlr3xpjo)

这些计算属性会一直追踪着被观察字段的变化，一旦有使用的字段值发生了改变，就会自动重新计算。这里其实有值得优化的性能问题，比如，改变 todo 的 name 其实不应该影响 todo 的完成与未完成数量。

我们可以通过添加一个额外的组件来单独观察完成与未完成的数量。现在再使用 React 开发者工具的“Highlight Updates”功能，你会发现改变 todo 的 name 就不会触发 todo 的数量也会重新渲染了，而只有当你切换它们的完成状态时，才会触发数量的重新渲染。

```javascript
const TodoCounterView = observer(props =>
        <div>
            {props.store.pendingCount} pending, {props.store.completedCount} completed
        </div>
)

const AppView = observer(props =>
        <div>
            <button onClick={e => props.store.addTodo(randomId(), 'New Task')}>Add Task</button>
            {props.store.todos.values().map(todo => <TodoView todo={todo} />)}
            <TodoCounterView store={props.store} />
        </div>
)
```
[View sample in playground](https://codesandbox.io/s/x3qlr3xpjo)

如果你`console.log`一下快照会发现，计算属性并没有出现在快照中。这其实故意就是这样设计的，因为计算属性必须是基于树的其它属性变化，才会被重新计算。因此，当你在快照中添加一个计算属性时，就会抛出异常。

## model 视图
你有可能需要在应用程序的不同位置使用过滤之后的待办事项列表。虽然每次去访问过滤一下待办事项列表也是一个可行的方案，但是，如果你的过滤器逻辑很复杂或者它随着时间在不断的变化，你就会发现这个方案并不是那么可行了。

MST 通过可以声明 model 视图来解决这个问题。A model views is declared as a function over the properties (first argument) of the model declaration. Model view 可以接受若干参数，并且仅能从 store 中读取数据。如果你尝试在 model view 中改变 store 的话，MST 就会抛出异常以阻止你这么做。

```javascript
const RootStore = types.model({
    users: types.map(User),
    todos: types.optional(types.map(Todo), {})
}).views(self => ({
    get pendingCount() {
        return self.todos.values().filter(todo => !todo.done).length
    },
    get completedCount() {
        return self.todos.values().filter(todo => todo.done).length
    },
    getTodosWhereDoneIs(done) {
        return self.todos.values().filter(todo => todo.done === done)
    }
})).actions(self => {
    function addTodo(id, name) {
        self.todos.set(id, Todo.create({ name }))
    }

    return {addTodo}
})
```
[View sample in playground](https://codesandbox.io/s/zkrkwj91p3)

Notice that the `getTodosWhereDoneIs` view can also be used outside of its model, for example it can be used inside views.

## 更进一步：引用
好了，这个基础的 TODO 应用已经完成了。还记得我上面说过的话吗，要让它具备能够给用户分配待办事项的能力。

在开始开发此功能之前，假设我们已经有了一个用户列表数据。你可以任意方式实现它或者可以给应用添加一个用户管理属性`users`，然后给它填充一些简单的数据映射即可。

```javascript
const store = RootStore.create({
    "users": {
        "1": {
            name: "mweststrate"
        },
        "2": {
            name: "mattiamanzati"
        },
        "3": {
            name: "johndoe"
        }
    },
    "todos": {
        "1": {
            "name": "Eat a cake",
            "done": true
        }
    }
})
```
[View sample in playground](https://codesandbox.io/s/7zqlw3ro11)

现在需要改写一下我们的 Todo model，以便让它能够将任务分配给相应的`user`。当然，你可以通过存储用户`id`，然后指定一个经过计算过的`user`来完成此功能。但是，这可能会最终导致你需要写大量的代码。（你可以自己做下练习）。

MST 提供了开箱即用的功能：引用。这意味着我们可以在 Todo model 上定义一个`user`属性，它是`User`对象的一个引用。当获取快照的时候，属性值就是`User`的标识符；当读取的时候，它会指定一个正确的`User model`实例；当设置它的值时，你可以提供`User model`实例或者`User`标识符都是可以的。

### 标识符
为了使引用能够工作起来，我们首先需要在目标 model 上创建一个引用的类型标识符，还需要告诉 MST 哪一个属性是`user`的标识符。

一旦 model 实例被创建并不会使标识符属性产生突变。这也就意味着如果你使用不同的标识符尝试把一个快照应用到那个 model 上，它就会抛出异常。换句话说，提供一个标识符可以帮助 MST 去理解 map 和 array 里的元素。并且在可能的情况下，使它能够在 maps/arrays 里正确的去重新使用 model 实例。

为了定义一个标识符，你需要先使用`types.identifier`定义一个元类型属性。比如，在这里我们期望标识符为字符串类型。

```javascript
const User = types.model({
    id: types.identifier(types.string),
    name: types.optional(types.string, "")
})
```

就像我以前说过的，标识符在元素的创建上是必需的，并且不会被突变。因此如果你收到了这样的错误提示，那是因为你还需要在使用 RootStore 创建的快照中也添加上相应的 id。

```
Error: [mobx-state-tree] Error while converting `{"users":{"1":{"name":"mweststrate"},"2":{"name":"mattiamanzati"},"3":{"name":"johndoe"}},"todos":{"1":{"name":"Eat a cake","done":true}}}` to `AnonymousModel`:
at path "/users/1/id" value `undefined` is not assignable to type: `identifier(string)`, expected an instance of `identifier(string)` or a snapshot like `identifier(string)` instead.
at path "/users/2/id" value `undefined` is not assignable to type: `identifier(string)`, expected an instance of `identifier(string)` or a snapshot like `identifier(string)` instead.
at path "/users/3/id" value `undefined` is not assignable to type: `identifier(string)`, expected an instance of `identifier(string)` or a snapshot like `identifier(string)` instead.
```
我们可以通过提供一个正确的快照来很容易的修复这个问题。
```javascript
const store = RootStore.create({
    "users": {
        "1": {
            id: "1",
            name: "mweststrate"
        },
        "2": {
            id: "2",
            name: "mattiamanzati"
        },
        "3": {
            id: "3",
            name: "johndoe"
        }
    },
    "todos": {
        "1": {
            "name": "Eat a cake",
            "done": true
        }
    }
})
```
[View sample in playground](https://codesandbox.io/s/mzvx6o7r0j)

### 如何定义引用
可以很容易地通过`types.reference(User)`来定义引用。但有时候这种方式可能会导致循环引用。To postpone the resolution of the type, you can use `types.late(() => User)` instead of just `User` and that will hoist the type and defer its evaluation. The user assignee for the Todo could also be omitted, so we will use `types.maybe(...)` to allow the user property to be null and be initialized as null.

```javascript
const Todo = types.model({
    name: types.optional(types.string, ""),
    done: types.optional(types.boolean, false),
    user: types.maybe(types.reference(types.late(() => User)))
}).actions(self => {
    function setName(newName) {
        self.name = newName
    }
    function toggle() {
        self.done = !self.done
    }

    return {setName, toggle}
})
```
[View sample in playground](https://codesandbox.io/s/mzvx6o7r0j)

### 设置引用的值
引用的值可以通过标识符或者 model 实例来提供。首先，我们需要定义一个可以改变 todo 的 user 的`action`。

```javascript
const Todo = types.model({
    name: types.optional(types.string, ""),
    done: types.optional(types.boolean, false),
    user: types.maybe(types.reference(types.late(() => User)))
}).actions(self => {
    function setName(newName) {
        self.name = newName
    }
    function setUser(user) {
        if (user === "") { // When selected value is empty, set as null
            self.user = null
        } else {
            self.user = user
        }
    }
    function toggle() {
        self.done = !self.done
    }

    return {setName, setUser, toggle}
})
```

Now we need to edit our views to display a select along with each todo, where the user can choose the assignee for that task. To do so, we will create a separate component (the UserPickerView) and use it inside the TodoView component to trigger the setUser call. That's it!

```javascript
const UserPickerView = observer(props =>
    <select value={props.user ? props.user.id : ""} onChange={e => props.onChange(e.target.value)}>
        <option value="">-none-</option>
        {props.store.users.values().map(user => <option value={user.id}>{user.name}</option>)}
    </select>
)

const TodoView = observer(props =>
        <div>
            <input type="checkbox" checked={props.todo.done} onChange={e => props.todo.toggle()} />
            <input type="text" value={props.todo.name} onChange={e => props.todo.setName(e.target.value)} />
            <UserPickerView user={props.todo.user} store={props.store} onChange={userId => props.todo.setUser(userId)} />
        </div>
)

const TodoCounterView = observer(props =>
        <div>
            {props.store.pendingCount} pending, {props.store.completedCount} completed
        </div>
)

const AppView = observer(props =>
        <div>
            <button onClick={e => props.store.addTodo(randomId(), 'New Task')}>Add Task</button>
            {props.store.todos.values().map(todo => <TodoView store={props.store} todo={todo} />)}
            <TodoCounterView store={props.store} />
        </div>
)
```
[View sample in playground](https://codesandbox.io/s/mzvx6o7r0j)

## References are safe!
One neat feature of references, is that they will throw if you accidentally remove a model that is required by a computed! If you try to remove a user that's used by a reference, you'll get something like this:

```
[mobx-state-tree] Failed to resolve reference of type <late>: '1' (in: /todos/1/user)
```

## Next up
In part 2 of this tutorial, we will discover how to use MST lifecycle hooks and local state to fetch user data from an XHR endpoint, and see how environments will help dealing with dependency injection of the parameters needed to fetch our endpoint. We will implement auto-save using mobx helpers and learn more about patches and actions event streams.

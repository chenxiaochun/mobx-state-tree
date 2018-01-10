# Getting Started

本教程将通过构建一个能够给用户分配待办事项的 TODO 应用来为你介绍 mobx-state-tree (MST) 的基础知识。

## 预备知识
开始此教程之前，我假设你已经对 React 有了基础的了解。否则，建议首先去阅读一下这份 [React 教程](https://facebook.github.io/react/tutorial/tutorial.html)。

### 我需要提前学习 MobX 吗?
MST 深度依赖于 MobX。因此，如果你使用过 MobX 的话，将对你处理一些复杂的情况和怎样把数据与 React 组件连接起来非常有帮助。如果你没有使用过 MobX 也没关系，因为使用 MST 不需要任何 MobX API 方面的知识。

## 如何追随这门教程
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
Then you can run `npm run start` and a basic React page will show up. You can now start editing all you need!

## 概述
mobx-state-tree is a state container that combines the simplicity and ease of mutable data with the traceability of immutable data and the reactiveness and performance of observable data.

If this sentence confused you, don't worry. We will dive together and explore what it means step by step.

## Getting Started
When building applications with MST, the first exercise that will help you building your application is thinking which is the minimal set of entities and their relative attributes of our application.

In our example application we will deal with todos, so we need a Todo entity. The Todo entity will have a name and a done attribute to store if that todo is done or not. We will also have a knowledge of Users (name will be enough for now), so we need an User entity, whose TODO will be assigned to.

In the end we will have something like:

User
* name

Todo
* name
* done

## 创建第一个 model
Central to MST (mobx-state-tree) is the concept of a living tree. The tree consists of mutable, but strictly protected objects enriched with runtime type information. In other words; each tree has a shape (type information) and state (data). From this living tree, immutable, structurally shared, snapshots are generated automatically.

This means that in order to make our application work, we need to describe how our entities are shaped to MST. Knowing that, MST will be able to automatically generate all those boundaries, and help us avoiding silly mistakes, like putting strings in price fields or booleans where an array is expected.

The simplest way to define a model for an entity in MST, is by providing a sample data that will be used as defaults for it, and pass it to the types.model function.

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

The code above will create two types, a User and a Todo type, but as we said before, a tree model in MST consists of type information (and we just saw how to define them) and state (the instance data). So how do we create an instance of the Todo and User type?

## 创建 model 实例 (tree nodes)
That could be easily done by calling .create() on the User and Todo type we just defined.
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

As you will see, using models ensures that all the fields defined will be always present and defaulted to the predefined value. If you want to change that value when creating the model instance, you can simply pass an object with the values to use into the create function.

```javascript
const eat = Todo.create({ name: "eat" })

console.log("Eat TODO:", eat.toJSON()) // => will print {name: "eat", done: false}
```
[View sample in playground](https://codesandbox.io/s/ymqpj71oj9)

## 遇见 types
When playing with this feature and passing in values to the create method, you may encounter an error like this:
```javascript
const eat = Todo.create({ name: "eat", done: 1 })
```
```
Error: [mobx-state-tree] Error while converting `{"name":"eat","done":1}` to `AnonymousModel`:
at path "/done" value `1` is not assignable to type: `boolean`.
```
What does this mean? As I said before, MST nodes are type-enriched. This means that providing a value (number) of the wrong type (expected boolean) will make MST throw an error. This is very helpful when building applications, as it will keep your state consistent and avoid entering in illegal states due to data of the wrong type. To be honest with you, I lied when I told you how to define models. The syntax you used was only a shortcut for the following syntax:

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

The types namespace provided in the MST package provides a lot of useful types and utility types like array, map, maybe, refinements and unions. If you are interested in them, feel free to check out the API documentation for the whole list and their parameters.

We can now use this knowledge to combine types and define the root type of our store that will hold users and todos. It will have a map of users and todos stored under the according key.

Notice that the types.optional second argument is required as long you don't pass a value in the create method of the type. If you want, for example, to make the name or todos property required when calling create, just remove the types.optional function call and just pass the types.* included inside.

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
MST tree nodes (model instances) can be modified using actions. Actions are colocated with your types and can be easily defined by declaring `.actions` over your model type and passing it a functions that accept the model instance and returns an object with the functions that modify that tree node.

For example, the following actions will be defined on the Todo type, and will allow you to toggle the done and set the name of the provided Todo instance.
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

Please notice the use of "self". Self is the object being constructed when an instance of your model is created. Thanks to the self object instance actions are "this-free", allowing you to be sure that they are correclty bound.

Calling those actions is as simple as what you would do with plain JavaScript classes, you simply call them on a model instance!

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
As we just saw, getting a snapshot from a model instance is pretty easy, but wouldn't it be neat to be able to restore a model from a snapshot? The good news is that you can!

That basically means that you can restore your objects with your custom methods by just knowing the type of the tree and its snapshot! You can perform this operation in two ways.

First is by creating a new model instance, and pass in the snapshot as argument to create. That means that you will need to update all your store references, if used in React components, to the new one.

The second option avoids this reference problem by applying the snapshot to an existing model instance. Properties will be updated, but the store reference will remain the same. This will trigger an operation called "reconciliation". We will speak later about this phase.

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
The ability of getting snapshots and applying them makes implementing time travel really easy in user-land. What you need to do is basically listen for snapshots, store them and reapply them to enable time travel!

A sample implementation would look like this:
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
MST loves MobX, and is fully compatible with it's autorun, reaction, observe, etc! You can use the mobx-react package to connect a MST store to a React component!
More details can be found on the mobx-react package documentation, but keep in mind that any view engine could be easily integrated with MST, just listen to onSnapshot and update accordingly!

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
If you have the React DevTools installed, using the "Highlight Updates" check you will see that the entire application will re-render whenever a todo is toggled or name is changed. That's a shame, as this can cause performance issues if there are a lots of todos in our list!

Thanks to the ability of MobX to emit granular updates, fixing that becomes pretty easy! You just need to split the rendering of a Todo into another component to only re-render that component whenever the todo data changes.

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

Basically each `observer` declaration will enable the React component to only re-render if any of it's observed data changes. Since our App component was observing everything, it was basically re-rendering whenever you changed something.

Now that we have split the rendering logic out into a separate observer, the Todo will re-render only if that todo changes, and App will re-render only if a new todo is added/removed since it's observing only the length of the todo map.

## 计算属性

We now want to display the count of TODOs to be done in our application, to help users know how many TODOs are left. That means that we need to count the number of TODOs with "done" set to false. To do this, we just need to modify the RootStore declaration, and add a getter property over our model by calling `.views` that will count how many TODOs are left.

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

Those properties are called "computed" because they keep track of the changes of the observed fields and recompute automatically if anything used by that field changes. This allows for performance savings; for example changing the name of a TODO won't affect the number of pending and completed count, as such it wont trigger a recalculation of those counters.

We can easily see that by creating an additional component in our application that observes the store and renders those counters. Using the React Dev Tools and tracing updates, you'll see that changing the name of a TODO won't re-render that counters, while checking completed or uncompleted will re-render the todo and the counters.

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

If you're console.log your snapshot, you'll notice that computed properties won't appear in snapshots. Thats fine and intended, since those properties must be computed over the other properties of the tree, they can be reproduced by knowing just their definition. For the same reason, if you provide a computed value in a snapshot you'll end up with an error when you attempt to apply it.

## model 视图
你有可能需要在应用程序的不同位置使用过滤之后的待办事项列表。虽然每次去访问过滤一下待办事项列表也是一个可行的方案，但是，如果你的过滤器逻辑很复杂或者它随着时间在不断的变化，你就会发现这个方案并不是那么可行了。

MST 通过可以声明 model 视图来解决这个问题。A model views is declared as a function over the properties (first argument) of the model declaration. Model views can accept parameters and only read data from our store. If you try to change your store from a model view, MST will throw and prevent you from doing so.

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

MST 提供了开箱即用的引用。这意味着我们可以在 Todo model 上定义一个`user`属性，它是`User`对象的一个引用。当获取快照的时候，属性值就是`User`的标识符；当读取的时候，它会指定一个正确的`User model`实例；当设置它的值时，你可以提供`User model`实例或者`User`标识符都是可以的。

### 标识符
为了使引用能够工作起来，我们首先需要在目标 model 上创建一个引用的类型标识符，还需要告诉 MST 哪一个属性是`user`的标识符。

In order to make our reference work, we first need to set up identifier in the targetted model Type of the reference. We need to tell MST which attribute is the identifier of the user.

The identifier attribute cannot be mutated once the model instance has been created. That also means that if you try to apply a snapshot with a different identifier on that model, it will throw. On the other hand, providing an identifier helps MST understand elements in maps and arrays, and allows it to correctly reuse model instances in arrays/maps when possible.

To define an identifier, you will need to define a property using the `types.identifier` type composer. For example, we want the identifier to be a string.

```javascript
const User = types.model({
    id: types.identifier(types.string),
    name: types.optional(types.string, "")
})
```

As I said before, identifiers are required upon creation of the element and cannot be mutated, so if you end up receiving an error like this, that's because you also need to provide ids for the users in the snapshot for the .create of the RootStore.

```
Error: [mobx-state-tree] Error while converting `{"users":{"1":{"name":"mweststrate"},"2":{"name":"mattiamanzati"},"3":{"name":"johndoe"}},"todos":{"1":{"name":"Eat a cake","done":true}}}` to `AnonymousModel`:
at path "/users/1/id" value `undefined` is not assignable to type: `identifier(string)`, expected an instance of `identifier(string)` or a snapshot like `identifier(string)` instead.
at path "/users/2/id" value `undefined` is not assignable to type: `identifier(string)`, expected an instance of `identifier(string)` or a snapshot like `identifier(string)` instead.
at path "/users/3/id" value `undefined` is not assignable to type: `identifier(string)`, expected an instance of `identifier(string)` or a snapshot like `identifier(string)` instead.
```
we can easily fix that by providing a correct snapshot.
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

### How to define the reference
The reference we are looking for can be easily defined as `types.reference(User)`. Sometimes this can lead to circular references that may use a type before it's declared. To postpone the resolution of the type, you can use `types.late(() => User)` instead of just `User` and that will hoist the type and defer its evaluation. The user assignee for the Todo could also be omitted, so we will use `types.maybe(...)` to allow the user property to be null and be initialized as null.

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

### Setting a reference value
The reference value can be set by providing either the identifier or a model instance. First of all, we need to define an action that will allow you to change the user of the todo.

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

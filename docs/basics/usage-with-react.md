# Usage with React

## Installing React Redux

## Presentational and Container Components

绑定 React 的 redux 接纳表现与容器组件分离 (separating presentional and container components) 的观点。如果你不熟悉这些术语，[read about them first](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0), 然后在回来继续看。他们很重要，我们等着你。

读完了吗？让我们看下他们的差异：

<table>
    <thead>
        <tr>
            <th></th>
            <th scope="col" style="text-align:left">Presentational Components</th>
            <th scope="col" style="text-align:left">Container Components</th>
        </tr>
    </thead>
    <tbody>
        <tr>
          <th scope="row" style="text-align:right">Purpose</th>
          <td>How things look (markup, styles)</td>
          <td>How things work (data fetching, state updates)</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Aware of Redux</th>
          <td>No</th>
          <td>Yes</th>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">To read data</th>
          <td>Read data from props</td>
          <td>Subscribe to Redux state</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">To change data</th>
          <td>Invoke callbacks from props</td>
          <td>Dispatch Redux actions</td>
        </tr>
        <tr>
          <th scope="row" style="text-align:right">Are written</th>
          <td>By hand</td>
          <td>Usually generated by React Redux</td>
        </tr>
    </tbody>
</table>

我们将要写的大多组件都是展示性的，但我们需要生成一些容器组件将他们连接至 Redux store. 这里和下面的设计概要并不意味着容器组件必须接近组件树的顶层。如果容器组件太复杂 (如，有数不尽的传递下来的回调的严重嵌套的展示组件), 引入组件树中的另一个组件，[faq](../faq/ReactRedux.md#react-multiple-components) 中有介绍。

技术上来讲，你可以使用 [store.subscribe()](../api/Store.md#subscribe) 手动编写容器组件。我们不建议你这样做，因为 React Redux 做了很多性能优化，你自己做的话会很难。如此，我们将使用 React Redux 提供的 [connect()](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) 生成容器组件，下面你会见到的。

## Designing Component Hierarchy

记得我们是如何设计根 state 对象 (reducers.md) 的吗。现在我们要设计匹配他的 UI。这不是 Redux 的任务。[Thinking in React](https://facebook.github.io/react/docs/thinking-in-react.html) 解释的很好。

我们设计的简单。我们想展示一个待办事项 (todo itmes). 点击时，待办项钩上标识完成。我们想要展示一个字段，让用户用来添加新的待办事项。在脚栏中，我们想显示 一个可以切换所有事项，仅完成事项，或仅活动事项的功能。

### Designing Presentational Components

* **`TodoList`** 是一个显示可见 todos 的列表。

  - `todos: Array` 形为 `{id, text, completed }` 的 todo 项。

  - `onTodoClick(id: number)` 是当 todo 点击时调用的回调。

* **`Todo`** 单个 todo 项。

  - `text: string` 是要显示的文本。

  - `completed: boolean` todo 项是否打钩。

  - `onClick()` 当点击 todo 时调用的回调

* **`Link`** 带回调的链接

  - `onClick` 当链接点击时调用的回调

* **`Footer`** 在这里用户可以改变当前可见的待办事项 (todos)

* **`App`** 渲染所有组件的根组件

他们描述外观但不知数扰来自何处，或怎样改变数据。他们只把给他们的东西渲染出来。如果你迁移 Redux 至别的，你将能够保持原来所有的组件。他们并不依赖 Redux.

### Designing Container Components

我们也需要一些容器组件将展示组件与 Redux 连接起来。例如，`TodoList` 展示组件需要一个类似 `VisibleTodoList` 样的组件订阅 Redux store 并且知道怎样应用当前的 visibility filter. 要更改  visibility filter, 我们提供一个 `FilterLink` 容器组件渲染一个 `Link` 当点击时触发相应的 action.

* **`VisibleTodoList`** 根据当前可见过滤器 (current visibility filter) 过滤待办事项。

* **`FilterLink`** 获取当前可见性过滤器并渲染一个 `Link`.
  - `filter: string` 表示可见性过滤器

### Designing Other Components

有时很难辩别一些组件是否应该是个展示性组件或者容器。例如，有时表单和功能是耦和在一起的，如这里的这个小组件：

* **`AddTodo`** 是一个有 "Add" 按钮的输入域。

技术上讲我们可以将其分割成两个组件，但此时可能太早了。小的组件混合展示逻辑是 ok 的，随着组件进化，分割组件会更明显，目前我们先把展示和逻辑混和在一起。

## Implementing Components

让我们开始写组件吧！我们从展示性组件开始，所以还不需要考虑绑定 Redux.

### Implements Presentatin Components

这些都是正常的 React 组件，所以不需要详细的审查他。除非需要本地状态或生命周期方法否则我们使用用无状态的函数组件。这并不意味着展示性组件必须要写成函数性的，这样只是方法定义而已。当需要添加本地状态 (local state), 生命周期方法，或性能优化时，你可能把他们转换成类。

__components/Todo.js__

```js
import React from 'react';
import PropTypes from 'prop-types';

const Todo = ({ onClick, completed, text }) => (
    <li onClick={onClick} style={{ textDecoration: completed ? 'line-through' : 'none' }}>
        {text}
    </li>
);

Todo.PropTypes = {
    onClick: PropTypes.func.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
};

export default Todo;
```

__components/TodoList.js__

```js
import React from 'react';
import PropTypes from 'prop-types';
import Todo from './Todo';


const TodoList = ({ todos, onTodoClick }) => (
    <ul>
        {todos.map((todo, index) => (
            <Todo key={index} {...todo} onClick={() => onTodoClick(index)} />
        ))}
    </ul>
);

TodoList.propTypes = {
    todos: PropTypes.arrayOf(
        PropTypes.shape({
            id: PropTypes.number.isRequired,
            completed: PropTypes.bool.isRequired,
            text: PropTypes.string.isRequired
        }).isRequired,
    ),
    onTodoClick: PropTypes.func.isRequired
};

export default TodoList;
```

__components/Links.js__

```js
import React from 'react';
import PropTypes from 'prop-types';

const Link = ({ active, children, onClick }) => {
    if(active){
        return <span>{ children }</span>;
    }
    
    return (
        <a href="" onClick={e => {
            e.preventDefault();
            onClick();
        }}>{ children }</a>
    );
};

Link.propTypes = {
    active: PropTypes.bool.isRequired,
    children: PropTypes.node.isRequired,
    onClick: PropTypes.func.isRequired
};

export default Link;
```

__components/Footer.js__

```js
import React from 'react';
import FilterLink from '../containers/FilterLink';
import { VisibilityFilters } from '../actions';

const Footer = () => (
    <p>
        Show:
        {' '}
        <FilterLink filter={VisibilityFilters.SHOW_ALL}>
            All
        </FilterLink>
        {' '}
        <FilterLink filter={VisibilityFilters.SHOW_ACTIVE}>
            Active
        </FilterLink>
        {', '}
        <FilterLink filter={VisibilityFilters.SHOW_COMPLETED}>
            Completed
        </FilterLink>
    </p>
);

export default Footer;
```

## Implementing Container Components

现在是时候创建一些容器将展示性组件与 Redux 连接起来了。技术上讲一个容器组件仅仅是使用 [store.subscribe()](api/store.md#subscrite) 读取 Redux 状态树的一部分并给展示性组件提供属性渲染。你可以手写容器组件，但我们建议使用 React Redux 库的 [connect()](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options) 生成容器组件，他提供了许多有用的优化以防止不要的重绘。(结果之一是你不需要担心 [React performance suggestion](https://facebook.github.io/react/docs/advanced-performance.html) of implementing `shouldComponentUpdate` yourself.)

要使用 `connect()`, 要定义一个号 `mapStateToProps` 的特殊函数，告知怎样将当前的 Redux store 状态转换成属性传递给你包装的展示性组件。如 `VisibleTodoList` 需要计算传递给 `TodoList` 的 `todos`, 所以我们可以定义一个函数根据 `state.visibilityFilter` 过滤 `state.todos`，在 `mapStateToProps` 中使用他：

```js
const getVisibleTodos = (todos, filter) => {
    switch(filter){
        case 'SHOW_COMPLETED':
            return todos.filter( t => t.completed);
        case 'SHOW_ACTIVE':
            return todos.filter(t => !t.completed);
        case 'SHOW_ALL':
        default:
            return todos;
    }
};

// 这里的 state 参数指 Redux store 中的 state
const mapStateToProps = state => {
    return {
        todos: getVisibleTodos(state.todos, state.visibilityFilter)
    };
};
```

除了读取 state，容器组件还可以发出 actions. 你可以按相似的形式定义一个叫 `mapDispatchToProps()` 的函数，接收 [dispatch()](api/store#dispatch) 方法返回调属性，你可将回调属性注入展示性组件。如，我们想 `VisibleTodoList` 往 `TodoList` 组件注入一个叫 `onTodoClick` 属性，而且我们想要 `onTodoClick` 可以发出 `TOGGLE_TODO` action:

```js
const mapDispatchToProps = dispatch => {
    return {
        onTodoClick: id => {
            // dispatch
            dispatch(toggleTodo(id))
        }
    };
};
```

最后，我们调用 `connect()` 传递这两个函数创建 `VisibleTodoList`：

```js
import { connect } from 'react-redux';

const VisibleTodoList = connect(
    mapStateToProps,
    mapDispatchToProps
)(TodoList);

export default VisibleTodoList;
```

这些是基本的 React Redux API, 但也有一些快捷方式和强大我选项，我们建议你看下[文档](https://github.com/reactjs/react-redux) 了解下细节。如果某种情形你担心 `mapStateToProps` 太经常创建新对象，你可能想学习下 [computing derived data](recipies/computing-derived-data.md) 及 [reselect](https://github.com/reactjs/reselect)

找下下面定义的其他的容器组件：

__containers/FilterLink.js__

```js
import { connect } from 'react-redux';
import { setVisibilityFilter } from '../acitons';
import Link from '../components/Link';

const mapStateToProps = (state, ownProps) => {
    return {
        active: ownProps.filter === state.visibilityFilter
    };
};

const mapDispatchToProps = (dispatch, ownProps) => {
    return {
        onClick: () => {
            dispatch(setVisibilityFilter(ownProps.filter));
        }
    };
};

const FilterLink = connect(
    mapStateToProps,
    mapDispatchToProps
)(Link);

export default FilterLink;
```

__containers/VisibleTodoList.js__

```js
import { connect } from 'react-redux';
import { toggleTodo } from '../actions';
import TodoList form '../components/TodoList';

const getVisibleTodos = (todos, filter) => {
    switch(filter){
        case 'SHOW_ALL':
            return todos;
        case 'SHOW_COMPLETED':
            return todos.filter(t => t.completed);
        case 'SHOW_ACTIVE':
            return todos.filter(t => !t.completed);
    }
};

const mapStateToProps = state => {
    return {
        todos: getVisibleTodos(state.todos, state.visibilityFilter)
    };
};

const mapDispatchToProps = dispatch => {
    return {
        onTodoClick: id => {
            dispatch(toggleTodo(id))
        }
    };
};

const VisibleTodoList = connect(
    mapStateToProps,
    mapDispatchToProps
)(TodoList);

export default VisibileTodoList;
```

#### Implementing Other Components

__containers/AddTodo.js__

回想下我们[前面提到的](#designing-other-components), `AddTodo` 的展示和逻辑定义在了一处。

```js
import React from 'react';
import { connect } from 'react-redux';
import { addTodo } from '../actions';

let AddTodo = ({ dispatch }) => {
    let input;
    return (
        <div>
            <form onSubmit={ e => {
                e.preventDefault();
                if(!input.value.trim()){
                    return
                }
                dispatch(addTodo(input.value));
                input.value = '';
            }}>
                <input ref={(node) => input = node }/>
                <button type="submit">
                    Add Todo
                </button>
            </form>
        </div>
    );
};

AddTodo = connect()(AddTodo);

export default AddTodo;
```

如果你不熟悉 `ref` 特性，请读下这篇[文档](https://facebook.github.io/react/docs/refs-and-the-dom.html) 熟悉下这个特性的推荐用法。

### Tying the containers together within a component

__components/App.js__

```js
import React from 'react';
import Footer from './Footer';
import AddTodo from '../containers/AddTodo';
import VisibleTodoList from '../containers/VisibleTodoList';

const App = () => (
    <div>
        <AddTodo />
        <VisibleTodoList />
        <Footer />
    </div>
);

export default App;
```

## Passing the Store

所有的容器组件都需要访问 Redux store 这样才能订阅他。选项之一是将 store 作为属性传递给每个容器组件。不过烦人，你必须要把 `store` 穿过展示性组件仅仅因谡他们碰巧要渲染组件树中深的容器组件。

我们推荐的做法是使用一个特殊的 React Redux 组件 [`<Provider>`](https://github.com/reactjs/react-redux/blob/master/docs/api.md#provider-store) 以[魔法地](https://facebook.github.io/react/docs/context.html) 使得应用中的所有容器组件都可用 store. 而不用明确地传递。你只需要在渲染根元素时使用一次就行。

__index.js__

```js
import React from 'react';
import { render } from 'react-dom';
import { Provider } from 'react-redux';
import { createStore } from 'redux';
import todoApp from './reducers';

const store = createStore(todoApp);

render(
    <Provider store={store}>
        <App />
    </Provider>,
    document.getElementById('root')
);
```

## Next Steps

[完成这篇教程的源代码](example-todo-list.md), 好好消化你学到的知识。然后，看看[高级教程](avanced/readme.md) 学习下如何处理网络请求及路由！

#

注意区分 React 组件的 state 与 Redux 的 state, 前者指组件的状态，后者指应用的状态。

ShouldComponentUpdate
---
title: useReducer
---

<Intro>

`useReducer`는 컴포넌트에 [reducer](/learn/extracting-state-logic-into-a-reducer)를 추가할 수 있게 해주는 React Hook입니다.

```js
const [state, dispatch] = useReducer(reducer, initialArg, init?)
```

</Intro>

<InlineToc />

---

## 레퍼런스 {/*reference*/}

### `useReducer(reducer, initialArg, init?)` {/*usereducer*/}

`useReducer`를 컴포넌트의 최상위에 호출하고, [reducer](/learn/extracting-state-logic-into-a-reducer)를 이용해 state를 관리합니다.

```js
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

[아래에서 더 많은 예시를 확인하세요.](#usage)

#### 매개변수 {/*parameters*/}

* `reducer`: state가 어떻게 업데이트되는지를 지정하는 리듀서 함수입니다. 리듀서 함수는 반드시 순수 함수여야 하며, state와 action을 인수로 받아야 하고, 다음 state를 반환해야 합니다. state와 action에는 모든 데이터 타입이 할당될 수 있습니다.
* `initialArg`: 초기 state가 계산되는 값입니다. 모든 데이터 타입이 할당될 수 있습니다. 초기 state가 어떻게 계산되는지는 다음 `init` 인수에 따라 달라집니다.
* **선택사항** `init`: 초기 state를 반환하는 초기화 함수입니다. 이 함수가 인수에 할당되지 않으면 초기 state는 `initialArg`로 설정됩니다. 할당되었다면 초기 state는 `init(initialArg)`를 호출한 결과가 할당됩니다.

#### 반환값 {/*returns*/}

`useReducer`는 2개의 엘리먼트로 구성된 배열을 반환합니다.

1. 현재 state. 첫번째 렌더링에서의 state는 `init(initialArg)` 또는 `initialArg`로 설정됩니다 (`init`이 없을 경우 `initialArg`로 설정됩니다).
2. [`dispatch` 함수](#dispatch). `dispatch`는 state를 새로운 값으로 업데이트하고 리렌더링을 일으킵니다.

#### 주의 사항 {/*caveats*/}

* `useReducer`는 Hook이므로 **컴포넌트의 최상위** 또는 커스텀 Hook에서만 호출할 수 있습니다. 반복문이나 조건문에서는 사용할 수 없습니다. 필요한 경우 새로운 컴포넌트를 추출하고 해당 컴포넌트로 state를 이동해서 사용할 수 있습니다.
* 엄격 모드에서는 [우연한 비순수성](#my-reducer-or-initializer-function-runs-twice)을 찾아내기 위해 reducer와 init 함수를 두번 호출합니다. 개발 환경에서만 한정된 동작이며, 배포 모드에는 영향을 미치지 않습니다. reducer와 init 함수가 순수 함수라면(그래야만 하듯이) 로직에 어떠한 영향도 미치지 않습니다. 호출 중 하나의 결과는 무시합니다.

---

### `dispatch` 함수 {/*dispatch*/}

`useReducer`에 의해 반환되는 `dispatch` 함수는 state를 새로운 값으로 업데이트하고 리렌더링을 일으킵니다. `dispatch`의 유일한 인수는 action입니다.

```js
const [state, dispatch] = useReducer(reducer, { age: 42 });

function handleClick() {
  dispatch({ type: 'incremented_age' });
  // ...
```

리액트는 현재 `state`와 `dispatch`를 통해 전달된 action을 제공받아 호출된 `reducer`의 반환값을 통해 다음 state값을 설정합니다. 

#### 매개변수 {/*dispatch-parameters*/}

* `action`: 사용자에 의해 수행된 활동입니다. 모든 데이터 타입이 할당될 수 있습니다. 컨벤션에 의해 action은 일반적으로 action을 정의하는 `type` 프로퍼티와 추가적인 정보를 표현하는 기타 프로퍼티를 포함한 객체로 구성됩니다.

#### 반환값 {/*dispatch-returns*/}

`dispatch` 함수는 어떤 값도 반환하지 않습니다.

#### 주의 사항 {/*setstate-caveats*/}

* `dispatch` 함수는 **오직 *다음* 렌더링에 사용할 state 변수만 업데이트 합니다.** 만약 `dispatch` 함수를 호출한 직후에 state 변수를 읽는다면 호출 이전의 [최신화되지 않은 값을 참조할 것입니다.](#ive-dispatched-an-action-but-logging-gives-me-the-old-state-value)

* [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) 비교를 통해 새롭게 제공된 값과 현재 `state`를 비교한 값이 같을 경우, 리액트는 컴포넌트와 해당 컴포넌트의 자식 요소들의 리렌더링을 건너뜁니다. 이것은 최적화에 관련된 동작입니다. 리액트는 결과를 무시하기 전에 컴포넌트를 호출해야 하지만, 호출이 코드에 영향을 미치지 않아야 합니다.

* 리액트는 [state의 업데이트를 배치합니다.](/learn/queueing-a-series-of-state-updates) **이벤트 핸들러의 모든 코드가 수행**되고 `set` 함수가 모두 호출된 후에 화면을 업데이트 합니다. 이는 하나의 이벤트에 리렌더링이 여러번 일어나는 것을 방지합니다. DOM 접근 등 이른 화면 업데이트를 강제해야할 특수한 상황이 있을 경우 [`flushSync`](/reference/react-dom/flushSync)를 사용할 수 있습니다.

---

## 사용법 {/*usage*/}

### 컴포넌트에 reducer 추가하기 {/*adding-a-reducer-to-a-component*/}

state를 [reducer](/learn/extracting-state-logic-into-a-reducer)로 관리하기 위해 `useReducer`를 컴포넌트의 최상단에서 호출합니다.

```js [[1, 8, "state"], [2, 8, "dispatch"], [4, 8, "reducer"], [3, 8, "{ age: 42 }"]]
import { useReducer } from 'react';

function reducer(state, action) {
  // ...
}

function MyComponent() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });
  // ...
```

`useReducer`는 정확히 2개의 항목이 포함된 배열 반환합니다.

1. state 변수의 <CodeStep step={1}>현재 state</CodeStep>. 최초에는 사용자가 제공한 <CodeStep step={3}>초기 state</CodeStep>로 초기화됩니다.
2. <CodeStep step={2}>`dispatch` 함수</CodeStep>. 상호작용에 대응하여 state를 변경합니다.

화면을 업데이트하려면 사용자가 수행한 활동을 의미하는 *action* 객체를 인수로하여 `dispatch` 함수를 호출하세요.

```js [[2, 2, "dispatch"]]
function handleClick() {
  dispatch({ type: 'incremented_age' });
}
```

리액트는 현재 state와 action을 <CodeStep step={4}>reducer 함수</CodeStep>로 전달합니다. reducer는 다음 state를 계산한 후 반환합니다. 리액트는 다음 state를 저장한 뒤에 컴포넌트와 함께 렌더링 하고 UI를 업데이트 합니다.

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  if (action.type === 'incremented_age') {
    return {
      age: state.age + 1
    };
  }
  throw Error('Unknown action.');
}

export default function Counter() {
  const [state, dispatch] = useReducer(reducer, { age: 42 });

  return (
    <>
      <button onClick={() => {
        dispatch({ type: 'incremented_age' })
      }}>
        Increment age
      </button>
      <p>Hello! You are {state.age}.</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

`useReducer`는 [`useState`](/reference/react/useState)와 매우 유사하지만, state 업데이트 로직을 이벤트 핸들러에서 컴포넌트 외부의 단일함수로 분리할 수 있다는 차이점이 있습니다. 자세한 사항은 [`useState`와 `useReducer` 비교하기](/learn/extracting-state-logic-into-a-reducer#comparing-usestate-and-usereducer)를 읽어보세요.

---

### reducer 함수 작성하기 {/*writing-the-reducer-function*/}

reducer 함수는 아래와 같이 선언합니다.

```js
function reducer(state, action) {
  // ...
}
```

이후 다음 state를 계산할 코드를 작성하고, 계산된 state를 반환합니다. 보통은 컨벤션에 따라 [`switch` 문](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/switch)을 사용합니다. `switch`는 각 `case`를 이용해 다음 state를 계산하고 반환합니다.

```js {4-7,10-13}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

Actions은 다양한 형태가 될 수 있습니다. 하지만 컨벤션에 따라 액션이 무엇인지 정의하는 `type` 프로퍼티를 포함한 객체로 선언하는 것이 일반적입니다. `type`은 reducer가 다음 state를 계산하는데 필요한 최소한의 정보를 포함해야 합니다.

```js {5,9-12}
function Form() {
  const [state, dispatch] = useReducer(reducer, { name: 'Taylor', age: 42 });
  
  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    });
  }
  // ...
```

action type 이름은 컴포넌트 내에서 지역적입니다. [각 action은 단일 상호작용을 설명하며, 데이터에 여러 변경 사항을 초래하더라도 하나의 상호작용만을 나타냅니다.](/learn/extracting-state-logic-into-a-reducer#writing-reducers-well) state의 형태는 임의적이지만, 일반적으로 객체나 배열일 것입니다.

자세한 내용은 [state 로직을 reducer로 작성하기](/learn/extracting-state-logic-into-a-reducer)를 읽어보세요.

<Pitfall>

State is read-only. Don't modify any objects or arrays in state:

```js {4,5}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 Don't mutate an object in state like this:
      state.age = state.age + 1;
      return state;
    }
```

Instead, always return new objects from your reducer:

```js {4-8}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ Instead, return a new object
      return {
        ...state,
        age: state.age + 1
      };
    }
```

Read [updating objects in state](/learn/updating-objects-in-state) and [updating arrays in state](/learn/updating-arrays-in-state) to learn more.

</Pitfall>

<Recipes titleText="Basic useReducer examples" titleId="examples-basic">

#### Form (object) {/*form-object*/}

In this example, the reducer manages a state object with two fields: `name` and `age`.

<Sandpack>

```js
import { useReducer } from 'react';

function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        name: state.name,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      return {
        name: action.nextName,
        age: state.age
      };
    }
  }
  throw Error('Unknown action: ' + action.type);
}

const initialState = { name: 'Taylor', age: 42 };

export default function Form() {
  const [state, dispatch] = useReducer(reducer, initialState);

  function handleButtonClick() {
    dispatch({ type: 'incremented_age' });
  }

  function handleInputChange(e) {
    dispatch({
      type: 'changed_name',
      nextName: e.target.value
    }); 
  }

  return (
    <>
      <input
        value={state.name}
        onChange={handleInputChange}
      />
      <button onClick={handleButtonClick}>
        Increment age
      </button>
      <p>Hello, {state.name}. You are {state.age}.</p>
    </>
  );
}
```

```css
button { display: block; margin-top: 10px; }
```

</Sandpack>

<Solution />

#### Todo list (array) {/*todo-list-array*/}

In this example, the reducer manages an array of tasks. The array needs to be updated [without mutation.](/learn/updating-arrays-in-state)

<Sandpack>

```js App.js
import { useReducer } from 'react';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(tasks, action) {
  switch (action.type) {
    case 'added': {
      return [...tasks, {
        id: action.id,
        text: action.text,
        done: false
      }];
    }
    case 'changed': {
      return tasks.map(t => {
        if (t.id === action.task.id) {
          return action.task;
        } else {
          return t;
        }
      });
    }
    case 'deleted': {
      return tasks.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false }
];
```

```js AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Add</button>
    </>
  )
}
```

```js TaskList.js hidden
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

</Sandpack>

<Solution />

#### Writing concise update logic with Immer {/*writing-concise-update-logic-with-immer*/}

If updating arrays and objects without mutation feels tedious, you can use a library like [Immer](https://github.com/immerjs/use-immer#useimmerreducer) to reduce repetitive code. Immer lets you write concise code as if you were mutating objects, but under the hood it performs immutable updates:

<Sandpack>

```js App.js
import { useImmerReducer } from 'use-immer';
import AddTask from './AddTask.js';
import TaskList from './TaskList.js';

function tasksReducer(draft, action) {
  switch (action.type) {
    case 'added': {
      draft.push({
        id: action.id,
        text: action.text,
        done: false
      });
      break;
    }
    case 'changed': {
      const index = draft.findIndex(t =>
        t.id === action.task.id
      );
      draft[index] = action.task;
      break;
    }
    case 'deleted': {
      return draft.filter(t => t.id !== action.id);
    }
    default: {
      throw Error('Unknown action: ' + action.type);
    }
  }
}

export default function TaskApp() {
  const [tasks, dispatch] = useImmerReducer(
    tasksReducer,
    initialTasks
  );

  function handleAddTask(text) {
    dispatch({
      type: 'added',
      id: nextId++,
      text: text,
    });
  }

  function handleChangeTask(task) {
    dispatch({
      type: 'changed',
      task: task
    });
  }

  function handleDeleteTask(taskId) {
    dispatch({
      type: 'deleted',
      id: taskId
    });
  }

  return (
    <>
      <h1>Prague itinerary</h1>
      <AddTask
        onAddTask={handleAddTask}
      />
      <TaskList
        tasks={tasks}
        onChangeTask={handleChangeTask}
        onDeleteTask={handleDeleteTask}
      />
    </>
  );
}

let nextId = 3;
const initialTasks = [
  { id: 0, text: 'Visit Kafka Museum', done: true },
  { id: 1, text: 'Watch a puppet show', done: false },
  { id: 2, text: 'Lennon Wall pic', done: false },
];
```

```js AddTask.js hidden
import { useState } from 'react';

export default function AddTask({ onAddTask }) {
  const [text, setText] = useState('');
  return (
    <>
      <input
        placeholder="Add task"
        value={text}
        onChange={e => setText(e.target.value)}
      />
      <button onClick={() => {
        setText('');
        onAddTask(text);
      }}>Add</button>
    </>
  )
}
```

```js TaskList.js hidden
import { useState } from 'react';

export default function TaskList({
  tasks,
  onChangeTask,
  onDeleteTask
}) {
  return (
    <ul>
      {tasks.map(task => (
        <li key={task.id}>
          <Task
            task={task}
            onChange={onChangeTask}
            onDelete={onDeleteTask}
          />
        </li>
      ))}
    </ul>
  );
}

function Task({ task, onChange, onDelete }) {
  const [isEditing, setIsEditing] = useState(false);
  let taskContent;
  if (isEditing) {
    taskContent = (
      <>
        <input
          value={task.text}
          onChange={e => {
            onChange({
              ...task,
              text: e.target.value
            });
          }} />
        <button onClick={() => setIsEditing(false)}>
          Save
        </button>
      </>
    );
  } else {
    taskContent = (
      <>
        {task.text}
        <button onClick={() => setIsEditing(true)}>
          Edit
        </button>
      </>
    );
  }
  return (
    <label>
      <input
        type="checkbox"
        checked={task.done}
        onChange={e => {
          onChange({
            ...task,
            done: e.target.checked
          });
        }}
      />
      {taskContent}
      <button onClick={() => onDelete(task.id)}>
        Delete
      </button>
    </label>
  );
}
```

```css
button { margin: 5px; }
li { list-style-type: none; }
ul, li { margin: 0; padding: 0; }
```

```json package.json
{
  "dependencies": {
    "immer": "1.7.3",
    "react": "latest",
    "react-dom": "latest",
    "react-scripts": "latest",
    "use-immer": "0.5.1"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

</Sandpack>

<Solution />

</Recipes>

---

### Avoiding recreating the initial state {/*avoiding-recreating-the-initial-state*/}

React saves the initial state once and ignores it on the next renders.

```js
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, createInitialState(username));
  // ...
```

Although the result of `createInitialState(username)` is only used for the initial render, you're still calling this function on every render. This can be wasteful if it's creating large arrays or performing expensive calculations.

To solve this, you may **pass it as an _initializer_ function** to `useReducer` as the third argument instead:

```js {6}
function createInitialState(username) {
  // ...
}

function TodoList({ username }) {
  const [state, dispatch] = useReducer(reducer, username, createInitialState);
  // ...
```

Notice that you’re passing `createInitialState`, which is the *function itself*, and not `createInitialState()`, which is the result of calling it. This way, the initial state does not get re-created after initialization.

In the above example, `createInitialState` takes a `username` argument. If your initializer doesn't need any information to compute the initial state, you may pass `null` as the second argument to `useReducer`.

<Recipes titleText="The difference between passing an initializer and passing the initial state directly" titleId="examples-initializer">

#### Passing the initializer function {/*passing-the-initializer-function*/}

This example passes the initializer function, so the `createInitialState` function only runs during initialization. It does not run when component re-renders, such as when you type into the input.

<Sandpack>

```js App.js hidden
import TodoList from './TodoList.js';

export default function App() {
  return <TodoList username="Taylor" />;
}
```

```js TodoList.js active
import { useReducer } from 'react';

function createInitialState(username) {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: username + "'s task #" + (i + 1)
    });
  }
  return {
    draft: '',
    todos: initialTodos,
  };
}

function reducer(state, action) {
  switch (action.type) {
    case 'changed_draft': {
      return {
        draft: action.nextDraft,
        todos: state.todos,
      };
    };
    case 'added_todo': {
      return {
        draft: '',
        todos: [{
          id: state.todos.length,
          text: state.draft
        }, ...state.todos]
      }
    }
  }
  throw Error('Unknown action: ' + action.type);
}

export default function TodoList({ username }) {
  const [state, dispatch] = useReducer(
    reducer,
    username,
    createInitialState
  );
  return (
    <>
      <input
        value={state.draft}
        onChange={e => {
          dispatch({
            type: 'changed_draft',
            nextDraft: e.target.value
          })
        }}
      />
      <button onClick={() => {
        dispatch({ type: 'added_todo' });
      }}>Add</button>
      <ul>
        {state.todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

<Solution />

#### Passing the initial state directly {/*passing-the-initial-state-directly*/}

This example **does not** pass the initializer function, so the `createInitialState` function runs on every render, such as when you type into the input. There is no observable difference in behavior, but this code is less efficient.

<Sandpack>

```js App.js hidden
import TodoList from './TodoList.js';

export default function App() {
  return <TodoList username="Taylor" />;
}
```

```js TodoList.js active
import { useReducer } from 'react';

function createInitialState(username) {
  const initialTodos = [];
  for (let i = 0; i < 50; i++) {
    initialTodos.push({
      id: i,
      text: username + "'s task #" + (i + 1)
    });
  }
  return {
    draft: '',
    todos: initialTodos,
  };
}

function reducer(state, action) {
  switch (action.type) {
    case 'changed_draft': {
      return {
        draft: action.nextDraft,
        todos: state.todos,
      };
    };
    case 'added_todo': {
      return {
        draft: '',
        todos: [{
          id: state.todos.length,
          text: state.draft
        }, ...state.todos]
      }
    }
  }
  throw Error('Unknown action: ' + action.type);
}

export default function TodoList({ username }) {
  const [state, dispatch] = useReducer(
    reducer,
    createInitialState(username)
  );
  return (
    <>
      <input
        value={state.draft}
        onChange={e => {
          dispatch({
            type: 'changed_draft',
            nextDraft: e.target.value
          })
        }}
      />
      <button onClick={() => {
        dispatch({ type: 'added_todo' });
      }}>Add</button>
      <ul>
        {state.todos.map(item => (
          <li key={item.id}>
            {item.text}
          </li>
        ))}
      </ul>
    </>
  );
}
```

</Sandpack>

<Solution />

</Recipes>

---

## Troubleshooting {/*troubleshooting*/}

### I've dispatched an action, but logging gives me the old state value {/*ive-dispatched-an-action-but-logging-gives-me-the-old-state-value*/}

Calling the `dispatch` function **does not change state in the running code**:

```js {4,5,8}
function handleClick() {
  console.log(state.age);  // 42

  dispatch({ type: 'incremented_age' }); // Request a re-render with 43
  console.log(state.age);  // Still 42!

  setTimeout(() => {
    console.log(state.age); // Also 42!
  }, 5000);
}
```

This is because [states behaves like a snapshot.](/learn/state-as-a-snapshot) Updating state requests another render with the new state value, but does not affect the `state` JavaScript variable in your already-running event handler.

If you need to guess the next state value, you can calculate it manually by calling the reducer yourself:

```js
const action = { type: 'incremented_age' };
dispatch(action);

const nextState = reducer(state, action);
console.log(state);     // { age: 42 }
console.log(nextState); // { age: 43 }
```

---

### I've dispatched an action, but the screen doesn't update {/*ive-dispatched-an-action-but-the-screen-doesnt-update*/}

React will **ignore your update if the next state is equal to the previous state,** as determined by an [`Object.is`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is) comparison. This usually happens when you change an object or an array in state directly:

```js {4-5,9-10}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // 🚩 Wrong: mutating existing object
      state.age++;
      return state;
    }
    case 'changed_name': {
      // 🚩 Wrong: mutating existing object
      state.name = action.nextName;
      return state;
    }
    // ...
  }
}
```

You mutated an existing `state` object and returned it, so React ignored the update. To fix this, you need to ensure that you're always [updating objects in state](/learn/updating-objects-in-state) and [updating arrays in state](/learn/updating-arrays-in-state) instead of mutating them:

```js {4-8,11-15}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ✅ Correct: creating a new object
      return {
        ...state,
        age: state.age + 1
      };
    }
    case 'changed_name': {
      // ✅ Correct: creating a new object
      return {
        ...state,
        name: action.nextName
      };
    }
    // ...
  }
}
```

---

### A part of my reducer state becomes undefined after dispatching {/*a-part-of-my-reducer-state-becomes-undefined-after-dispatching*/}

Make sure that every `case` branch **copies all of the existing fields** when returning the new state:

```js {5}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      return {
        ...state, // Don't forget this!
        age: state.age + 1
      };
    }
    // ...
```

Without `...state` above, the returned next state would only contain the `age` field and nothing else.

---

### My entire reducer state becomes undefined after dispatching {/*my-entire-reducer-state-becomes-undefined-after-dispatching*/}

If your state unexpectedly becomes `undefined`, you're likely forgetting to `return` state in one of the cases, or your action type doesn't match any of the `case` statements. To find why, throw an error outside the `switch`:

```js {10}
function reducer(state, action) {
  switch (action.type) {
    case 'incremented_age': {
      // ...
    }
    case 'edited_name': {
      // ...
    }
  }
  throw Error('Unknown action: ' + action.type);
}
```

You can also use a static type checker like TypeScript to catch such mistakes.

---

### I'm getting an error: "Too many re-renders" {/*im-getting-an-error-too-many-re-renders*/}

You might get an error that says: `Too many re-renders. React limits the number of renders to prevent an infinite loop.` Typically, this means that you're unconditionally dispatching an action *during render*, so your component enters a loop: render, dispatch (which causes a render), render, dispatch (which causes a render), and so on. Very often, this is caused by a mistake in specifying an event handler:

```js {1-2}
// 🚩 Wrong: calls the handler during render
return <button onClick={handleClick()}>Click me</button>

// ✅ Correct: passes down the event handler
return <button onClick={handleClick}>Click me</button>

// ✅ Correct: passes down an inline function
return <button onClick={(e) => handleClick(e)}>Click me</button>
```

If you can't find the cause of this error, click on the arrow next to the error in the console and look through the JavaScript stack to find the specific `dispatch` function call responsible for the error.

---

### My reducer or initializer function runs twice {/*my-reducer-or-initializer-function-runs-twice*/}

In [Strict Mode](/reference/react/StrictMode), React will call your reducer and initializer functions twice. This shouldn't break your code.

This **development-only** behavior helps you [keep components pure.](/learn/keeping-components-pure) React uses the result of one of the calls, and ignores the result of the other call. As long as your component, initializer, and reducer functions are pure, this shouldn't affect your logic. However, if they are accidentally impure, this helps you notice the mistakes.

For example, this impure reducer function mutates an array in state:

```js {4-6}
function reducer(state, action) {
  switch (action.type) {
    case 'added_todo': {
      // 🚩 Mistake: mutating state
      state.todos.push({ id: nextId++, text: action.text });
      return state;
    }
    // ...
  }
}
```

Because React calls your reducer function twice, you'll see the todo was added twice, so you'll know that there is a mistake. In this example, you can fix the mistake by [replacing the array instead of mutating it](/learn/updating-arrays-in-state#adding-to-an-array):

```js {4-11}
function reducer(state, action) {
  switch (action.type) {
    case 'added_todo': {
      // ✅ Correct: replacing with new state
      return {
        ...state,
        todos: [
          ...state.todos,
          { id: nextId++, text: action.text }
        ]
      };
    }
    // ...
  }
}
```

Now that this reducer function is pure, calling it an extra time doesn't make a difference in behavior. This is why React calling it twice helps you find mistakes. **Only component, initializer, and reducer functions need to be pure.** Event handlers don't need to be pure, so React will never call your event handlers twice.

Read [keeping components pure](/learn/keeping-components-pure) to learn more.

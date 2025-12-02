# Explain some things:

1. Why getNewId() is called twice?
2. Why do we need keys?
3. Why settings a state doesn't work?

```js
import React, { useState } from 'react';

let _id = 0;
const getNewId = () => {
  return _id++;
};

export const TodoComponent = () => {
  const [todoList, setTodoList] = useState(() => [
    // getNewId() is called on every render
    { name: 'feed the dog', checked: false, id: getNewId() },
  ]);

  const addNewTodo = () => {
    todoList.push({
      name: 'buy groceries',
      checked: false,
      id: getNewId(),
    });
    setTodoList(todoList);
    // setTodoList(prev => {
    //   return prev.concat({
    //     name: 'buy groceries',
    //     checked: false,
    //     id: getNewId(),
    //   });
    // });
  };

  return (
    <div style={{ margin: 42 }}>
      <button onClick={addNewTodo}>add todo item</button>
      <ul>
        {todoList.map(todo => ( // Add key
          <li>
            {todo.id} {todo.checked ? '✓' : ' '} {todo.name}
          </li>
        ))}
      </ul>
    </div>
  );
};
```

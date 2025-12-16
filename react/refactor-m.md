# Look at this code, refactor as you see fit

```jsx

// dataService.js
import {ClientType as _ClientType} from './root';

export function getData(state) {
    const response = fetch('/api/companies', {
        method: 'POST',
        body: JSON.stringify({
            id: state.client.id,
            type: _ClientType.company
        })
    });

    return response
}

export const ClientType = _ClientType;

// component.jsx
import * as React from 'react';
import {getData, ClientType} from './dataService';

function Component(props) {
    const { store } = props;
    const [items, setItems] = React.useState([]);

    React.useEffect(() => {
        (async () => {
            const response = getData(store.getState());
            const data = await response.json();

            setItems(data.items);
        })();
    }, []);

    return (
        <div>
            {items.map((item) => (
                <div className={`item item_type_${item.type}`}>
                    {item.type === ClientType.person ? (
                        <span>{item.id}: {item.name}</span>
                    ) : (
                        <span>Компания {item.id}</span>
                    )}
                    <button onClick={() => {console.log(item.id)}}>choose</button>
                </div>
            ))}
        </div>
    );
}

export default Component;

// root.jsx
import * as React from 'react';
import Component from './component';
import store from './store';

export const ClientType = {
    'company': 'company',
    'person': 'person',
};

function Root() {
    return (
        <Component store={store}/>
    );
}

export default Root;

// app.jsx
import * as React from 'react';
import ReactDOM from 'react-dom';
import Root from './root';

ReactDOM.render(<Root />, document.querySelector('#app'));

```

# Solution

Things to consider:

1. Stable keys for lists
2. Use TypeScript
3. Handle loading & error states
4. Extract item rendering into a subcomponent
5. Uncouple from global store shape (state.client.id)
6. Improve naming
7. Make modules cohesive (ClientType)
8. Make enum for ClientType
9. Use consistent exports
10. Use `createRoot` instead of `ReactDOM.render`
11. `getData` should be `async` and return parsed data, not a raw `Response`
12. Create `useItems(store, getData)`
13. Keeps the effect body small and readable
14. Instead of logging inside the component, accept `onChoose(item)` as a prop
15. `'/api/companies'` should be in a config file or at least a constant
16. Internationalization / text constants

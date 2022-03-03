[use fetch hook reference article](https://www.smashingmagazine.com/2020/07/custom-react-hook-fetch-cache-data/)

## React Fetch Data Example

```javascript
import { useState, useEffect } from 'react';

const [status, setStatus] = useState('idle');
const [query, setQuery] = useState('');
const [data, setData] = useState([]);

useEffect(() => {
    if (!query) return;

    const fetchData = async () => {
        setStatus('fetching');
        const response = await fetch(
            `https://hn.algolia.com/api/v1/search?query=${query}`
        );
        const data = await response.json();
        setData(data.hits);
        setStatus('fetched');
    };

    fetchData();
}, [query]);
```


## React Fetch Data Hook Example

```javascript
const useFetch = (url) => {
    const [status, setStatus] = useState('idle');
    const [data, setData] = useState([]);

    useEffect(() => {
        if (!url) return;
        const fetchData = async () => {
            setStatus('fetching');
            const response = await fetch(url);
            const data = await response.json();
            setData(data);
            setStatus('fetched');
        };

        fetchData();
    }, [url]);

    return { status, data };
};
```

Example of use: 

```javascript
const [query, setQuery] = useState('');

const url = query && `https://hn.algolia.com/api/v1/search?query=${query}`;
const { status, data } = useFetch(url);
```

## React Fetch Data Hook with Cashing

```javascript
const cache = {};

const useFetch = (url) => {
    const [status, setStatus] = useState('idle');
    const [data, setData] = useState([]);

    useEffect(() => {
        if (!url) return;

        const fetchData = async () => {
            setStatus('fetching');
            if (cache[url]) {
                const data = cache[url];
                setData(data);
                setStatus('fetched');
            } else {
                const response = await fetch(url);
                const data = await response.json();
                cache[url] = data; // set response in cache;
                setData(data);
                setStatus('fetched');
            }
        };

        fetchData();
    }, [url]);

    return { status, data };
};
```


## React Fetch Data Hook using Ref to store cached data

```javascript
const useFetch = (url) => {
    const cache = useRef({});
    const [status, setStatus] = useState('idle');
    const [data, setData] = useState([]);

    useEffect(() => {
        if (!url) return;
        const fetchData = async () => {
            setStatus('fetching');
            if (cache.current[url]) {
                const data = cache.current[url];
                setData(data);
                setStatus('fetched');
            } else {
                const response = await fetch(url);
                const data = await response.json();
                cache.current[url] = data; // set response in cache;
                setData(data);
                setStatus('fetched');
            }
        };

        fetchData();
    }, [url]);

    return { status, data };
};
```

## Make better with useReducer

```javascript
const initialState = {
    status: 'idle',
    error: null,
    data: [],
};

const [state, dispatch] = useReducer((state, action) => {
    switch (action.type) {
        case 'FETCHING':
            return { ...initialState, status: 'fetching' };
        case 'FETCHED':
            return { ...initialState, status: 'fetched', data: action.payload };
        case 'FETCH_ERROR':
            return { ...initialState, status: 'error', error: action.payload };
        default:
            return state;
    }
}, initialState);
```

## Add clean up with canel request

```javascript
useEffect(() => {
    let cancelRequest = false;
    if (!url) return;

    const fetchData = async () => {
        dispatch({ type: 'FETCHING' });
        if (cache.current[url]) {
            const data = cache.current[url];
            dispatch({ type: 'FETCHED', payload: data });
        } else {
            try {
                const response = await fetch(url);
                const data = await response.json();
                cache.current[url] = data;
                if (cancelRequest) return;
                dispatch({ type: 'FETCHED', payload: data });
            } catch (error) {
                if (cancelRequest) return;
                dispatch({ type: 'FETCH_ERROR', payload: error.message });
            }
        }
    };

    fetchData();

    return function cleanup() {
        cancelRequest = true;
    };
}, [url]);
```

## Final example

```javascript
import { useEffect, useRef, useReducer } from 'react';

export const useFetch = (url) => {

	const cache = useRef({});

	const initialState = {
		status: 'idle',
		error: null,
		data: [],
	};


	// Reducer
	const [state, dispatch] = useReducer((state, action) => {

		switch (action.type) {
			case 'FETCHING':
				return { ...initialState, status: 'fetching' };

			case 'FETCHED':
				return { ...initialState, status: 'fetched', data: action.payload };

			case 'FETCH_ERROR':
				return { ...initialState, status: 'error', error: action.payload };

			default:
				return state;
		}

	}, initialState);

	useEffect(() => {
		let cancelRequest = false;
		if (!url || !url.trim()) return;
		
		const fetchData = async () => {
			dispatch({ type: 'FETCHING' });
			if (cache.current[url]) {
				const data = cache.current[url];
				dispatch({ type: 'FETCHED', payload: data });
			} else {
				try {
					const response = await fetch(url);
					const data = await response.json();
					cache.current[url] = data;

					if (cancelRequest) return;
					dispatch({ type: 'FETCHED', payload: data });
				} catch (error) {
					if (cancelRequest) return;
						dispatch({ type: 'FETCH_ERROR', payload: error.message });
					}
				}

		};

		fetchData();

		return function cleanup() {
			cancelRequest = true;
		};

	}, [url]);

	return state;

};

```
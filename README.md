# Zustand
A comprehensive guide/cheatsheet for learning zustand
# Zustand Learning Guide

![Zustand Logo](https://github.com/pmndrs/zustand/raw/main/bear.jpg)

A comprehensive guide to mastering Zustand - a small, fast, and scalable state management solution for React applications.

## Table of Contents

- [What is Zustand?](#what-is-zustand)
- [Why Choose Zustand?](#why-choose-zustand)
- [Installation](#installation)
- [Basic Concepts](#basic-concepts)
- [Getting Started](#getting-started)
- [Core Features](#core-features)
- [Advanced Patterns](#advanced-patterns)
- [Best Practices](#best-practices)
- [Common Patterns & Examples](#common-patterns--examples)
- [Testing](#testing)
- [Migration Guides](#migration-guides)
- [Ecosystem & Integrations](#ecosystem--integrations)
- [Performance Tips](#performance-tips)
- [Troubleshooting](#troubleshooting)
- [Learning Resources](#learning-resources)

## What is Zustand?

Zustand (German for "state") is a lightweight state management library for React applications. It provides a simple and intuitive API for managing both local and global state without the boilerplate typically associated with other state management solutions.

### Key Characteristics

- **Minimal boilerplate**: No providers, actions, or reducers required
- **TypeScript first**: Excellent TypeScript support out of the box
- **Flexible**: Works with both React and vanilla JavaScript
- **Performant**: Only re-renders components that subscribe to changed state
- **Small bundle size**: ~2KB gzipped

## Why Choose Zustand?

### Compared to Redux
- ✅ Less boilerplate code
- ✅ No need for providers or context
- ✅ Built-in async support
- ✅ Simpler learning curve

### Compared to Context API
- ✅ Better performance (selective subscriptions)
- ✅ No provider hell
- ✅ Works outside React components

### Compared to other solutions
- ✅ Framework agnostic core
- ✅ Intuitive API design
- ✅ Great developer experience

## Installation

```bash
# npm
npm install zustand

# yarn
yarn add zustand

# pnpm
pnpm add zustand
```

## Basic Concepts

### Store
A store is where your application state lives. It's created using the `create` function and contains both state and actions to update that state.

### State
The data you want to manage in your application.

### Actions
Functions that update the state. They have access to the current state and the `set` function to update it.

### Selectors
Functions that extract specific pieces of state from the store, allowing components to subscribe to only the data they need.

## Getting Started

### Your First Store

```javascript
import { create } from 'zustand'

const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}))
```

### Using the Store in Components

```jsx
import React from 'react'
import { useCounterStore } from './store'

function Counter() {
  const { count, increment, decrement, reset } = useCounterStore()

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  )
}
```

### Selective Subscriptions

```jsx
// Only subscribe to count, not the actions
const count = useCounterStore((state) => state.count)

// Subscribe to multiple values
const { count, increment } = useCounterStore((state) => ({
  count: state.count,
  increment: state.increment
}))
```

## Core Features

### 1. Simple State Updates

```javascript
const useStore = create((set, get) => ({
  user: null,
  setUser: (user) => set({ user }),
  clearUser: () => set({ user: null }),
  
  // Access current state with get()
  updateUserName: (name) => set({
    user: { ...get().user, name }
  })
}))
```

### 2. Async Actions

```javascript
const useStore = create((set, get) => ({
  users: [],
  loading: false,
  error: null,
  
  fetchUsers: async () => {
    set({ loading: true, error: null })
    try {
      const response = await fetch('/api/users')
      const users = await response.json()
      set({ users, loading: false })
    } catch (error) {
      set({ error: error.message, loading: false })
    }
  }
}))
```

### 3. Computed Values

```javascript
const useStore = create((set, get) => ({
  items: [],
  filter: 'all',
  
  addItem: (item) => set((state) => ({ 
    items: [...state.items, item] 
  })),
  
  setFilter: (filter) => set({ filter }),
  
  // Computed property
  get filteredItems() {
    const { items, filter } = get()
    if (filter === 'completed') {
      return items.filter(item => item.completed)
    }
    if (filter === 'active') {
      return items.filter(item => !item.completed)
    }
    return items
  }
}))
```

### 4. TypeScript Support

```typescript
interface CounterState {
  count: number
  increment: () => void
  decrement: () => void
  reset: () => void
}

const useCounterStore = create<CounterState>()((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
}))
```

## Advanced Patterns

### 1. Store Slices

```javascript
// User slice
const createUserSlice = (set, get) => ({
  user: null,
  setUser: (user) => set((state) => ({ ...state, user })),
  clearUser: () => set((state) => ({ ...state, user: null }))
})

// Posts slice
const createPostsSlice = (set, get) => ({
  posts: [],
  addPost: (post) => set((state) => ({
    ...state,
    posts: [...state.posts, post]
  }))
})

// Combine slices
const useStore = create((set, get) => ({
  ...createUserSlice(set, get),
  ...createPostsSlice(set, get)
}))
```

### 2. Middleware

#### Persist Middleware
```javascript
import { persist } from 'zustand/middleware'

const useStore = create(
  persist(
    (set) => ({
      count: 0,
      increment: () => set((state) => ({ count: state.count + 1 }))
    }),
    {
      name: 'counter-storage', // unique name
      storage: createJSONStorage(() => sessionStorage) // optional
    }
  )
)
```

#### Immer Middleware
```javascript
import { immer } from 'zustand/middleware/immer'

const useStore = create(
  immer((set) => ({
    users: [],
    addUser: (user) => set((state) => {
      state.users.push(user) // Direct mutation with Immer
    })
  }))
)
```

#### DevTools Middleware
```javascript
import { devtools } from 'zustand/middleware'

const useStore = create(
  devtools(
    (set) => ({
      count: 0,
      increment: () => set(
        (state) => ({ count: state.count + 1 }),
        false,
        'increment' // Action name for devtools
      )
    }),
    {
      name: 'counter-store'
    }
  )
)
```

### 3. Store Composition

```javascript
// Base store
const useBaseStore = create((set) => ({
  theme: 'light',
  setTheme: (theme) => set({ theme })
}))

// Extended store
const useAppStore = create((set, get) => ({
  ...useBaseStore.getState(),
  user: null,
  setUser: (user) => set({ user }),
  
  // Access base store methods
  toggleTheme: () => {
    const currentTheme = useBaseStore.getState().theme
    useBaseStore.getState().setTheme(
      currentTheme === 'light' ? 'dark' : 'light'
    )
  }
}))
```

## Best Practices

### 1. Store Organization

```javascript
// ✅ Good: Organize related state and actions together
const useUserStore = create((set, get) => ({
  // State
  user: null,
  isAuthenticated: false,
  loading: false,
  
  // Actions
  login: async (credentials) => { /* ... */ },
  logout: () => { /* ... */ },
  updateProfile: (data) => { /* ... */ }
}))

// ❌ Avoid: Mixing unrelated concerns
const useMixedStore = create((set) => ({
  user: null,
  todos: [],
  theme: 'light',
  shoppingCart: []
}))
```

### 2. Action Naming

```javascript
// ✅ Good: Clear, descriptive action names
const useStore = create((set) => ({
  items: [],
  addItem: (item) => set((state) => ({ items: [...state.items, item] })),
  removeItem: (id) => set((state) => ({ 
    items: state.items.filter(item => item.id !== id) 
  })),
  updateItem: (id, updates) => set((state) => ({
    items: state.items.map(item => 
      item.id === id ? { ...item, ...updates } : item
    )
  }))
}))
```

### 3. Selective Subscriptions

```javascript
// ✅ Good: Subscribe only to what you need
const TodoItem = ({ id }) => {
  const todo = useStore(state => 
    state.todos.find(todo => todo.id === id)
  )
  
  return <div>{todo.text}</div>
}

// ❌ Avoid: Subscribing to entire store
const TodoItem = ({ id }) => {
  const { todos, user, settings } = useStore() // Unnecessary subscriptions
  const todo = todos.find(todo => todo.id === id)
  
  return <div>{todo.text}</div>
}
```

### 4. Error Handling

```javascript
const useStore = create((set, get) => ({
  data: null,
  loading: false,
  error: null,
  
  fetchData: async () => {
    set({ loading: true, error: null })
    
    try {
      const response = await api.getData()
      set({ data: response, loading: false })
    } catch (error) {
      set({ 
        error: error.message, 
        loading: false,
        data: null 
      })
    }
  },
  
  clearError: () => set({ error: null })
}))
```

## Common Patterns & Examples

### 1. Todo App

```javascript
const useTodoStore = create((set, get) => ({
  todos: [],
  filter: 'all',
  
  addTodo: (text) => set((state) => ({
    todos: [...state.todos, {
      id: Date.now(),
      text,
      completed: false,
      createdAt: new Date()
    }]
  })),
  
  toggleTodo: (id) => set((state) => ({
    todos: state.todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    )
  })),
  
  removeTodo: (id) => set((state) => ({
    todos: state.todos.filter(todo => todo.id !== id)
  })),
  
  setFilter: (filter) => set({ filter }),
  
  get filteredTodos() {
    const { todos, filter } = get()
    switch (filter) {
      case 'active':
        return todos.filter(todo => !todo.completed)
      case 'completed':
        return todos.filter(todo => todo.completed)
      default:
        return todos
    }
  },
  
  get stats() {
    const todos = get().todos
    return {
      total: todos.length,
      completed: todos.filter(todo => todo.completed).length,
      active: todos.filter(todo => !todo.completed).length
    }
  }
}))
```

### 2. Shopping Cart

```javascript
const useCartStore = create((set, get) => ({
  items: [],
  
  addItem: (product) => set((state) => {
    const existingItem = state.items.find(item => item.id === product.id)
    
    if (existingItem) {
      return {
        items: state.items.map(item =>
          item.id === product.id
            ? { ...item, quantity: item.quantity + 1 }
            : item
        )
      }
    }
    
    return {
      items: [...state.items, { ...product, quantity: 1 }]
    }
  }),
  
  removeItem: (id) => set((state) => ({
    items: state.items.filter(item => item.id !== id)
  })),
  
  updateQuantity: (id, quantity) => set((state) => ({
    items: quantity <= 0 
      ? state.items.filter(item => item.id !== id)
      : state.items.map(item =>
          item.id === id ? { ...item, quantity } : item
        )
  })),
  
  clearCart: () => set({ items: [] }),
  
  get total() {
    return get().items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  },
  
  get itemCount() {
    return get().items.reduce((sum, item) => sum + item.quantity, 0)
  }
}))
```

### 3. Authentication Store

```javascript
const useAuthStore = create((set, get) => ({
  user: null,
  token: null,
  isAuthenticated: false,
  loading: false,
  error: null,
  
  login: async (credentials) => {
    set({ loading: true, error: null })
    
    try {
      const response = await authAPI.login(credentials)
      const { user, token } = response
      
      set({
        user,
        token,
        isAuthenticated: true,
        loading: false
      })
      
      // Store token in localStorage
      localStorage.setItem('token', token)
      
    } catch (error) {
      set({
        error: error.message,
        loading: false
      })
    }
  },
  
  logout: () => {
    localStorage.removeItem('token')
    set({
      user: null,
      token: null,
      isAuthenticated: false,
      error: null
    })
  },
  
  refreshToken: async () => {
    const token = localStorage.getItem('token')
    if (!token) return
    
    try {
      const response = await authAPI.refreshToken(token)
      set({ token: response.token })
    } catch (error) {
      get().logout()
    }
  },
  
  initializeAuth: () => {
    const token = localStorage.getItem('token')
    if (token) {
      set({ token, isAuthenticated: true })
      get().refreshToken()
    }
  }
}))
```

## Testing

### Testing Stores

```javascript
import { act, renderHook } from '@testing-library/react'
import { useCounterStore } from './counter-store'

describe('Counter Store', () => {
  beforeEach(() => {
    // Reset store before each test
    useCounterStore.setState({ count: 0 })
  })
  
  it('should increment count', () => {
    const { result } = renderHook(() => useCounterStore())
    
    act(() => {
      result.current.increment()
    })
    
    expect(result.current.count).toBe(1)
  })
  
  it('should decrement count', () => {
    const { result } = renderHook(() => useCounterStore())
    
    // Set initial state
    act(() => {
      useCounterStore.setState({ count: 5 })
    })
    
    act(() => {
      result.current.decrement()
    })
    
    expect(result.current.count).toBe(4)
  })
})
```

### Testing Components with Stores

```javascript
import { render, screen, fireEvent } from '@testing-library/react'
import { useCounterStore } from './counter-store'
import Counter from './Counter'

// Mock the store
const mockIncrement = jest.fn()
const mockDecrement = jest.fn()

jest.mock('./counter-store', () => ({
  useCounterStore: jest.fn()
}))

describe('Counter Component', () => {
  beforeEach(() => {
    useCounterStore.mockReturnValue({
      count: 0,
      increment: mockIncrement,
      decrement: mockDecrement
    })
  })
  
  it('displays count and handles clicks', () => {
    render(<Counter />)
    
    expect(screen.getByText('Count: 0')).toBeInTheDocument()
    
    fireEvent.click(screen.getByText('+'))
    expect(mockIncrement).toHaveBeenCalled()
    
    fireEvent.click(screen.getByText('-'))
    expect(mockDecrement).toHaveBeenCalled()
  })
})
```

## Migration Guides

### From Redux to Zustand

```javascript
// Redux
const initialState = { count: 0 }

const counterReducer = (state = initialState, action) => {
  switch (action.type) {
    case 'INCREMENT':
      return { count: state.count + 1 }
    case 'DECREMENT':
      return { count: state.count - 1 }
    default:
      return state
  }
}

const increment = () => ({ type: 'INCREMENT' })
const decrement = () => ({ type: 'DECREMENT' })

// Zustand equivalent
const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 }))
}))
```

### From Context API to Zustand

```jsx
// Context API
const CounterContext = createContext()

const CounterProvider = ({ children }) => {
  const [count, setCount] = useState(0)
  const increment = () => setCount(c => c + 1)
  const decrement = () => setCount(c => c - 1)
  
  return (
    <CounterContext.Provider value={{ count, increment, decrement }}>
      {children}
    </CounterContext.Provider>
  )
}

// Zustand equivalent - no provider needed!
const useCounterStore = create((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 }))
}))
```

## Ecosystem & Integrations

### Popular Middleware

- **zustand/middleware/persist** - Persist store to localStorage/sessionStorage
- **zustand/middleware/immer** - Use Immer for immutable updates
- **zustand/middleware/devtools** - Redux DevTools integration
- **zustand/middleware/subscribeWithSelector** - Enhanced subscriptions
- **zustand/middleware/combine** - Combine multiple stores

### Third-party Integrations

- **React Query + Zustand** - Server state + client state
- **Zustand + React Router** - Navigation state management
- **Zustand + Form libraries** - Form state management

## Performance Tips

### 1. Use Selective Subscriptions

```javascript
// ✅ Good: Only subscribe to needed state
const name = useUserStore(state => state.user?.name)

// ❌ Avoid: Subscribing to entire store
const { user, posts, settings } = useUserStore()
const name = user?.name
```

### 2. Memoize Selectors

```javascript
import { useShallow } from 'zustand/react/shallow'

// ✅ Good: Prevent unnecessary re-renders
const { name, email } = useUserStore(
  useShallow(state => ({ name: state.name, email: state.email }))
)
```

### 3. Split Large Stores

```javascript
// ✅ Good: Separate concerns
const useUserStore = create(/* user-related state */)
const usePostsStore = create(/* posts-related state */)
const useUIStore = create(/* UI-related state */)

// ❌ Avoid: One large store for everything
const useAppStore = create(/* everything */)
```

## Troubleshooting

### Common Issues

1. **State not updating**: Check if you're calling `set` correctly
2. **Unnecessary re-renders**: Use selective subscriptions
3. **TypeScript errors**: Ensure proper type definitions
4. **Persistence not working**: Check middleware configuration
5. **DevTools not connecting**: Verify devtools middleware setup

### Debug Tips

```javascript
// Add logging to actions
const useStore = create((set, get) => ({
  count: 0,
  increment: () => {
    console.log('Before increment:', get().count)
    set((state) => ({ count: state.count + 1 }))
    console.log('After increment:', get().count)
  }
}))

// Use store outside React for debugging
console.log(useStore.getState())
useStore.getState().increment()
```

## Learning Resources

### Official Documentation
- [Zustand GitHub Repository](https://github.com/pmndrs/zustand)
- [API Reference](https://docs.pmnd.rs/zustand/getting-started/introduction)

### Tutorials & Articles
- [Zustand vs Redux: When to Use What](https://blog.example.com)
- [Building Real-World Apps with Zustand](https://tutorial.example.com)
- [TypeScript Best Practices with Zustand](https://guide.example.com)

### Video Content
- [Zustand Crash Course](https://youtube.com/watch?v=example)
- [Advanced Zustand Patterns](https://youtube.com/watch?v=example)

### Community
- [Discord Server](https://discord.gg/poimandres)
- [GitHub Discussions](https://github.com/pmndrs/zustand/discussions)

## Conclusion

Zustand provides a simple, powerful, and flexible solution for state management in React applications. Its minimal API surface and excellent TypeScript support make it an ideal choice for projects of all sizes. Start small with basic stores and gradually adopt advanced patterns as your application grows.

Remember: The best state management solution is the one that fits your project's needs and your team's preferences. Zustand excels at being simple to learn, powerful to use, and easy to scale.

---

**Happy coding with Zustand! 🐻**

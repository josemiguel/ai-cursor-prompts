# Setting Up a React + TypeScript + Vite + Tailwind Project

This guide provides step-by-step instructions for setting up a modern React application with TypeScript, Vite, and Tailwind CSS, including common troubleshooting tips.

## Prerequisites

- Node.js (v14.0.0 or higher)
- npm (v6.0.0 or higher)

## Step 1: Create a New Vite Project

```bash
# Create a new project with Vite
npm create vite@latest my-app --template react-ts

# Navigate to the project directory
cd my-app
```

## Step 2: Install Core Dependencies

```bash
# Install the base dependencies
npm install
```

## Step 3: Set Up Tailwind CSS

When setting up Tailwind CSS, pay close attention to the module system (CommonJS vs ES modules) to avoid configuration errors.

```bash
# Install Tailwind CSS and its dependencies
# Note: Use specific versions known to work well with Vite
npm install -D tailwindcss@3.3.5 postcss@8.4.31 autoprefixer@10.4.16
```

## Step 4: Configure Tailwind CSS

Create configuration files for Tailwind CSS and PostCSS. Make sure to use the correct export syntax based on your project's module system.

### For projects with "type": "module" in package.json (ES modules):

```bash
# Initialize Tailwind CSS
npx tailwindcss init -p
```

Then update the generated files:

**tailwind.config.js**
```javascript
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      // Add custom theme extensions here
      animation: {
        'spin-slow': 'spin 20s linear infinite',
      },
    },
  },
  plugins: [],
}
```

**postcss.config.js**
```javascript
export default {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

### For projects without "type": "module" (CommonJS):

**tailwind.config.js**
```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
  content: [
    "./index.html",
    "./src/**/*.{js,ts,jsx,tsx}",
  ],
  theme: {
    extend: {
      // Add custom theme extensions here
    },
  },
  plugins: [],
}
```

**postcss.config.js**
```javascript
module.exports = {
  plugins: {
    tailwindcss: {},
    autoprefixer: {},
  },
}
```

## Step 5: Add Tailwind Directives to CSS

Create or update your main CSS file (usually `src/index.css`):

```css
@tailwind base;
@tailwind components;
@tailwind utilities;

/* Custom styles can be added below */
```

## Step 6: Create Reusable Components

Create a components directory to organize your React components:

```bash
mkdir -p src/components
```

### Example Card Component

```tsx
// src/components/Card.tsx
import React, { ReactNode } from 'react';

interface CardProps {
  title?: string;
  children: ReactNode;
  className?: string;
}

const Card: React.FC<CardProps> = ({ title, children, className = '' }) => {
  return (
    <div className={`bg-white dark:bg-gray-800 p-6 rounded-lg shadow-md ${className}`}>
      {title && <h2 className="text-xl font-semibold mb-4 text-gray-800 dark:text-white">{title}</h2>}
      {children}
    </div>
  );
};

export default Card;
```

### Example Button Component

```tsx
// src/components/Button.tsx
import React, { ButtonHTMLAttributes } from 'react';

interface ButtonProps extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: 'primary' | 'secondary' | 'outline';
  size?: 'sm' | 'md' | 'lg';
  fullWidth?: boolean;
}

const Button: React.FC<ButtonProps> = ({
  children,
  variant = 'primary',
  size = 'md',
  fullWidth = false,
  className = '',
  ...props
}) => {
  const baseClasses = 'font-medium rounded-md transition-colors focus:outline-none focus:ring-2 focus:ring-offset-2';
  
  const variantClasses = {
    primary: 'bg-indigo-600 text-white hover:bg-indigo-700 focus:ring-indigo-500',
    secondary: 'bg-gray-200 text-gray-800 hover:bg-gray-300 focus:ring-gray-500',
    outline: 'bg-transparent border border-gray-300 text-gray-700 hover:bg-gray-50 focus:ring-gray-500',
  };
  
  const sizeClasses = {
    sm: 'px-3 py-1.5 text-sm',
    md: 'px-4 py-2',
    lg: 'px-6 py-3 text-lg',
  };
  
  const widthClass = fullWidth ? 'w-full' : '';
  
  const classes = `${baseClasses} ${variantClasses[variant]} ${sizeClasses[size]} ${widthClass} ${className}`;
  
  return (
    <button className={classes} {...props}>
      {children}
    </button>
  );
};

export default Button;
```

### Example Todo Component

```tsx
// src/components/Todo.tsx
import React, { useState } from 'react';
import Card from './Card';
import Button from './Button';

interface TodoItem {
  id: number;
  text: string;
  completed: boolean;
}

const Todo: React.FC = () => {
  const [todos, setTodos] = useState<TodoItem[]>([]);
  const [input, setInput] = useState('');

  const handleAddTodo = () => {
    if (input.trim() === '') return;
    
    const newTodo: TodoItem = {
      id: Date.now(),
      text: input.trim(),
      completed: false
    };
    
    setTodos([...todos, newTodo]);
    setInput('');
  };

  const handleToggleTodo = (id: number) => {
    setTodos(todos.map(todo => 
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  const handleDeleteTodo = (id: number) => {
    setTodos(todos.filter(todo => todo.id !== id));
  };

  return (
    <Card title="Todo List" className="max-w-md w-full mx-auto">
      <div className="flex gap-2 mb-4">
        <input
          type="text"
          value={input}
          onChange={(e) => setInput(e.target.value)}
          onKeyPress={(e) => e.key === 'Enter' && handleAddTodo()}
          placeholder="Add a new task..."
          className="flex-1 p-2 border rounded-md focus:outline-none focus:ring-2 focus:ring-indigo-500 dark:bg-gray-700 dark:border-gray-600 dark:text-white"
        />
        <Button onClick={handleAddTodo}>Add</Button>
      </div>
      
      <ul className="space-y-2">
        {todos.length === 0 ? (
          <p className="text-gray-500 dark:text-gray-400 text-center py-4">No tasks yet. Add one above!</p>
        ) : (
          todos.map(todo => (
            <li key={todo.id} className="flex items-center justify-between p-3 bg-gray-50 dark:bg-gray-700 rounded-md">
              <div className="flex items-center gap-2">
                <input
                  type="checkbox"
                  checked={todo.completed}
                  onChange={() => handleToggleTodo(todo.id)}
                  className="h-5 w-5 text-indigo-600 rounded focus:ring-indigo-500"
                />
                <span className={`${todo.completed ? 'line-through text-gray-500' : 'text-gray-800 dark:text-white'}`}>
                  {todo.text}
                </span>
              </div>
              <Button 
                variant="outline" 
                size="sm" 
                onClick={() => handleDeleteTodo(todo.id)}
                className="text-red-500 hover:bg-red-50 hover:border-red-200"
              >
                Delete
              </Button>
            </li>
          ))
        )}
      </ul>
    </Card>
  );
};

export default Todo;
```

## Step 7: Update App.tsx

```tsx
// src/App.tsx
import { useState } from 'react'
import reactLogo from './assets/react.svg'
import viteLogo from '/vite.svg'
import Todo from './components/Todo'

function App() {
  const [count, setCount] = useState(0)

  return (
    <div className="min-h-screen flex flex-col items-center justify-center p-8 bg-gray-50 dark:bg-gray-900">
      <div className="flex gap-8 mb-8">
        <a href="https://vite.dev" target="_blank" className="hover:scale-110 transition-transform">
          <img src={viteLogo} className="h-24" alt="Vite logo" />
        </a>
        <a href="https://react.dev" target="_blank" className="hover:scale-110 transition-transform">
          <img src={reactLogo} className="h-24 animate-spin-slow" alt="React logo" />
        </a>
      </div>
      <h1 className="text-4xl font-bold mb-8 text-gray-800 dark:text-white">Vite + React</h1>
      
      {/* Todo Component */}
      <div className="w-full max-w-md mb-8">
        <Todo />
      </div>
      
      <div className="bg-white dark:bg-gray-800 p-6 rounded-lg shadow-md mb-8 max-w-md w-full">
        <button 
          onClick={() => setCount((count) => count + 1)}
          className="mb-4 px-4 py-2 bg-indigo-600 text-white rounded-md hover:bg-indigo-700 transition-colors w-full"
        >
          count is {count}
        </button>
        <p className="text-gray-700 dark:text-gray-300">
          Edit <code className="bg-gray-100 dark:bg-gray-700 px-1 py-0.5 rounded">src/App.tsx</code> and save to test HMR
        </p>
      </div>
      <p className="text-gray-500 dark:text-gray-400">
        Click on the Vite and React logos to learn more
      </p>
    </div>
  )
}

export default App
```

## Step 8: Start the Development Server

```bash
npm run dev
```

## Troubleshooting Common Issues

### 1. Module System Mismatch

**Issue**: Error when ES module syntax (`export default`) is used in a CommonJS environment or vice versa.

**Solution**: Check your `package.json` for the presence of `"type": "module"`. If it exists, use ES module syntax; otherwise, use CommonJS syntax (`module.exports`).

### 2. Tailwind CSS PostCSS Plugin Error

**Issue**: Error like `It looks like you're trying to use tailwindcss directly as a PostCSS plugin`.

**Solution**: Either:
   - Install `@tailwindcss/postcss` and update the PostCSS config to use it instead of `tailwindcss`
   - Use a specific older version of Tailwind CSS (e.g., 3.3.5) that is compatible with your setup

### 3. Unknown Utility Class Error

**Issue**: Error like `Cannot apply unknown utility class: px-4`.

**Solution**:
   - Make sure your content paths in `tailwind.config.js` are correct
   - Remove any custom `@apply` directives until the base Tailwind setup is working
   - Simplify your CSS to just the Tailwind directives initially

### 4. Module Not Found Error

**Issue**: Components or modules not being found.

**Solution**: Check your import paths and make sure they match your directory structure.

## Next Steps

Once your base project is working, consider adding:

1. **React Router**: For multi-page navigation
2. **State Management**: Like Redux Toolkit or Zustand for complex state
3. **API Integration**: Using Axios or Fetch for backend communication
4. **Form Handling**: With React Hook Form or Formik
5. **Testing**: Jest and React Testing Library

## Conclusion

This setup gives you a modern React application with TypeScript, Vite for fast development, and Tailwind CSS for styling. The reusable components provide a foundation for building more complex interfaces while maintaining a clean architecture. 

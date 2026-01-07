# Installation and tsconfig

## What it is

TypeScript requires installation as a development tool and uses a configuration file (`tsconfig.json`) to control how the TypeScript compiler transforms `.ts` files into JavaScript. The config file defines compiler options, file inclusion/exclusion rules, and type-checking strictness.

## Installation

TypeScript can be installed globally or locally per project:

```bash
# Global installation (available everywhere)
npm install -g typescript

# Local installation (recommended for projects)
npm install --save-dev typescript

# Verify installation
tsc --version
```

## Before tsconfig (Manual Compilation)

Without a configuration file, you must specify all compiler options via command line:

```bash
# Compile a single file with options
tsc myFile.ts --target ES2020 --module commonjs --strict --outDir ./dist

# Very verbose for every compilation
tsc src/index.ts src/utils.ts --target ES2015 --lib ES2015,DOM --outDir build
```

This becomes unmaintainable as projects grow and options accumulate.

## After tsconfig (Configured Compilation)

Create a `tsconfig.json` file in your project root:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020", "DOM"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "**/*.spec.ts"]
}
```

Now compilation is simple:

```bash
# Compiles all files according to tsconfig.json
tsc

# Watch mode (recompile on changes)
tsc --watch
```

## Essential tsconfig Options Explained

```json
{
  "compilerOptions": {
    // JavaScript version to target (ES5, ES2015, ES2020, ESNext)
    "target": "ES2020",
    
    // Module system (commonjs for Node, ES2015/ES6 for modern)
    "module": "commonjs",
    
    // Type definition libraries to include
    "lib": ["ES2020", "DOM"],
    
    // Where compiled JavaScript goes
    "outDir": "./dist",
    
    // Root of TypeScript source files
    "rootDir": "./src",
    
    // Enable all strict type-checking options (recommended!)
    "strict": true,
    
    // Allow default imports from modules with no default export
    "esModuleInterop": true,
    
    // Skip type checking of declaration files (.d.ts) for faster compilation
    "skipLibCheck": true,
    
    // Ensure consistent file name casing
    "forceConsistentCasingInFileNames": true,
    
    // Generate source maps for debugging
    "sourceMap": true,
    
    // Generate .d.ts declaration files
    "declaration": true,
    
    // Don't emit output if there are errors
    "noEmitOnError": true
  },
  
  // Files to include in compilation
  "include": ["src/**/*"],
  
  // Files to exclude from compilation
  "exclude": ["node_modules", "dist", "**/*.spec.ts"]
}
```

## Why This Is Better

- **Single source of truth**: All compiler settings in one place
- **Consistency**: Same settings used by all team members
- **IDE integration**: Editors use tsconfig for type checking and autocomplete
- **Simpler commands**: Just run `tsc` instead of long commands
- **Project-specific settings**: Different projects can have different configurations
- **Version controlled**: Config is committed to repository

## Common Mistake + Fix

**Mistake**: Not using `"strict": true` and missing important type-checking features

```json
{
  "compilerOptions": {
    "target": "ES2020"
    // Missing "strict" allows unsafe code
  }
}
```

This allows implicit `any` types, null/undefined issues, and other unsafe patterns.

**Fix**: Always start with `"strict": true` for maximum type safety:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "strict": true  // Enables: strictNullChecks, noImplicitAny, strictFunctionTypes, etc.
  }
}
```

If strict is too aggressive initially, enable strict options individually:

```json
{
  "compilerOptions": {
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true
  }
}
```

## Key Notes / Edge Cases

- **Multiple tsconfig files**: You can have different configs for different parts of your project using `extends`:
  ```json
  // tsconfig.prod.json
  {
    "extends": "./tsconfig.json",
    "compilerOptions": {
      "sourceMap": false
    }
  }
  ```

- **Project references**: Large codebases can use project references for better build performance
- **Type-only imports**: Use `"importsNotUsedAsValues": "error"` to enforce type-only imports
- **Module resolution**: `"moduleResolution": "node"` is typically needed for Node.js projects

## Quick Practice

1. **Initialize a project**: Create a new directory and initialize it with:
   - `package.json` (using `npm init`)
   - TypeScript as a dev dependency
   - A basic `tsconfig.json`
   - A simple `src/index.ts` file

2. **Configure for a web app**: What options would you set in tsconfig.json for a browser-based application that needs to support older browsers (ES5) but use modern TypeScript features?

3. **Debug the config**: This tsconfig produces errors. What's wrong?
   ```json
   {
     "compilerOptions": {
       "target": "ES5",
       "lib": ["ES2020"],
       "strict": true
     }
   }
   ```
   Hint: Think about what happens when you try to use ES2020 features but target ES5.

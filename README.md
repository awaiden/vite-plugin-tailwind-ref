# vite-plugin-tailwind-ref

A Vite plugin that automatically adds `@reference` directives to CSS files using Tailwind CSS `@apply`, enabling utility class usage without manual imports.

## Problem

When using Tailwind's `@apply` directive in CSS files (including framework-scoped styles like Svelte or Vue components), you typically need to manually add `@reference` directives:

```css
@reference "tailwindcss";

.my-class {
  @apply flex items-center;
}
```

This is repetitive and easy to forget.

## Solution

This plugin automatically injects the necessary `@reference` directive when it detects `@apply` usage in your CSS files.

## Installation

```bash
npm install -D vite-plugin-tailwind-ref
```

## Usage

Add the plugin to your `vite.config.ts`:

```typescript
import { defineConfig } from "vite";
import tailwindAutoReference from "vite-plugin-tailwind-ref";

export default defineConfig({
  plugins: [tailwindAutoReference()],
});
```

Now you can use `@apply` in your CSS files without manually adding `@reference` directives:

```css
.my-class {
  @apply flex items-center justify-center;
}
```

The plugin will automatically inject:

```css
@reference "tailwindcss";

.my-class {
  @apply flex items-center justify-center;
}
```

### Svelte

For Svelte component style blocks, configure the `include` pattern to match Svelte's style query:

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import { svelte } from "@sveltejs/vite-plugin-svelte";
import tailwindAutoReference from "vite-plugin-tailwind-ref";

export default defineConfig({
  plugins: [
    svelte(),
    tailwindAutoReference("tailwindcss", {
      include: [/\.svelte\?.*svelte&type=style/],
    }),
  ],
});
```

Then use `@apply` freely in your components:

```svelte
<style>
  .my-class {
    @apply flex items-center justify-center;
  }
</style>
```

### Vue

For Vue component style blocks, match Vue's style query:

```typescript
// vite.config.ts
import { defineConfig } from "vite";
import vue from "@vitejs/plugin-vue";
import tailwindAutoReference from "vite-plugin-tailwind-ref";

export default defineConfig({
  plugins: [
    vue(),
    tailwindAutoReference("tailwindcss", {
      include: [/\.vue\?.*type=style/],
    }),
  ],
});
```

Then use `@apply` in your components:

```vue
<style scoped>
.my-class {
  @apply flex items-center justify-center;
}
</style>
```

### Multiple frameworks / CSS files together

You can combine multiple patterns to cover both standalone CSS files and framework style blocks:

```typescript
tailwindAutoReference("tailwindcss", {
  include: [/\.css$/, /\.svelte\?.*svelte&type=style/],
});
```

## Configuration

### Basic Options

```typescript
tailwindAutoReference(cssFile, options);
```

#### `cssFile` (optional)

Type: `string | string[] | (code?: string, id?: string) => string | string[] | Promise<string | string[]>`  
Default: `"tailwindcss"`

The path(s) to your Tailwind CSS file, or a function that returns the path(s).

**Examples:**

```typescript
// String path
tailwindAutoReference("./src/styles/tailwind.css");

// Multiple paths
tailwindAutoReference(["./src/styles/tailwind.css", "./src/styles/custom.css"]);

// Dynamic function
tailwindAutoReference((code, id) => {
  return id.includes("admin") ? "./admin-tailwind.css" : "./tailwind.css";
});

// Async function
tailwindAutoReference(async (code, id) => {
  const config = await loadConfig();
  return config.cssPath;
});
```

#### `options` (optional)

Type: `object`

- **`include`** (optional)  
  Type: `FilterPattern` (string | RegExp | Array<string | RegExp>)  
  Default: `[/\.css$/]`  
  Patterns to match files for transformation.

- **`exclude`** (optional)  
  Type: `FilterPattern`  
  Default: `[]`  
  Patterns to exclude from transformation.

- **`skip`** (optional)  
  Type: `(code?: string, id?: string) => boolean`  
  Default: `() => false`  
  Custom function to skip transformation based on file content or path.

### Advanced Examples

```typescript
// Custom include/exclude patterns
tailwindAutoReference("./src/styles/tailwind.css", {
  include: [/\.css$/, /\.svelte\?.*svelte&type=style/],
  exclude: [/node_modules/],
});

// Skip specific files
tailwindAutoReference("./src/styles/tailwind.css", {
  skip: (code, id) => {
    return code?.includes("@reference") ?? false;
  },
});
```

## How It Works

1. The plugin runs in Vite's `pre` enforce phase
2. It intercepts CSS files (or files matching the configured `include` pattern)
3. When it detects `@apply` usage, it injects the `@reference` directive
4. If `@use` statements exist, the `@reference` is placed after the last `@use`
5. Otherwise, it's placed at the beginning of the file
6. Files that already contain `@reference` or `@import "tailwindcss"` are skipped

## Requirements

- Vite 5.x, 6.x, 7.x, or 8.x

## Credits

This plugin is inspired by [vite-plugin-vue-tailwind-auto-reference](https://github.com/M1CK431/vite-plugin-vue-tailwind-auto-reference) by M1CK431.

## License

MIT

## Contributing

Issues and pull requests are welcome at [https://github.com/awaiden/vite-plugin-tailwind-ref](https://github.com/awaiden/vite-plugin-tailwind-ref)

# Vite Config & Build Optimization

## Table of Contents
1. [vite.config.ts Baseline](#1-viteconfigts-baseline)
2. [Path Aliases](#2-path-aliases)
3. [Dev Proxy](#3-dev-proxy)
4. [Bundle Optimization](#4-bundle-optimization)
5. [Environment Variables](#5-environment-variables)
6. [Common HMR Issues](#6-common-hmr-issues)

---

## 1. vite.config.ts Baseline

```ts
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'
import path from 'path'

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: { '@': path.resolve(__dirname, './src') },
  },
  build: {
    sourcemap: false,          // true for staging
    minify: 'esbuild',
    rollupOptions: {
      output: {
        manualChunks: {
          vendor: ['react', 'react-dom'],
          router: ['react-router-dom'],
          query: ['@tanstack/react-query'],
        },
      },
    },
  },
  server: {
    port: 5173,
    proxy: {
      '/api': { target: 'http://localhost:3000', changeOrigin: true },
    },
  },
})
```

---

## 2. Path Aliases

Must be defined in **both** files — divergence causes TypeScript errors or silent build failures.

**vite.config.ts**:
```ts
resolve: {
  alias: { '@': path.resolve(__dirname, './src') }
}
```

**tsconfig.json**:
```json
{
  "compilerOptions": {
    "baseUrl": ".",
    "paths": { "@/*": ["src/*"] }
  }
}
```

---

## 3. Dev Proxy

Avoid CORS issues in dev by proxying API calls through Vite's dev server.

```ts
server: {
  proxy: {
    '/api': {
      target: 'http://localhost:3000',
      changeOrigin: true,
      rewrite: (path) => path.replace(/^\/api/, ''),
    },
  },
}
```

---

## 4. Bundle Optimization

### Chunk Splitting

Split large vendor libs into separate chunks so users don't re-download unchanged code.

```ts
rollupOptions: {
  output: {
    manualChunks(id) {
      if (id.includes('node_modules')) {
        // Heavy UI libs get their own chunk
        if (id.includes('recharts') || id.includes('@tremor')) return 'charts'
        if (id.includes('react-router')) return 'router'
        return 'vendor'
      }
    },
  },
}
```

### Analyze Bundle Size

Run before every major deploy to catch regressions.

```bash
npx vite-bundle-analyzer
# or
npx rollup-plugin-visualizer
```

Warn threshold: any single chunk > **500 KB** uncompressed.

### Lazy Load Heavy Libs

```ts
// charts, rich-text editors, map libraries
const Chart = React.lazy(() => import('./components/Chart'))
const Editor = React.lazy(() => import('./components/RichEditor'))
```

---

## 5. Environment Variables

### Rules

| Rule | Detail |
|------|--------|
| Client-safe vars | Prefix `VITE_`, access via `import.meta.env.VITE_FOO` |
| Server-only secrets | No `VITE_` prefix — never bundled into client |
| Type declarations | Add to `src/vite-env.d.ts` or `env.d.ts` |

**env.d.ts**:
```ts
/// <reference types="vite/client" />
interface ImportMetaEnv {
  readonly VITE_API_URL: string
  readonly VITE_APP_TITLE: string
}
interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

**Files**:
- `.env` — base defaults (commit only non-secret defaults)
- `.env.local` — local overrides (gitignored)
- `.env.production` — production values (use CI secrets instead when possible)

---

## 6. Common HMR Issues

| Symptom | Cause | Fix |
|---------|-------|-----|
| HMR not updating | Component defined in same file as `createRoot` | Move component to separate file |
| Full page reload instead of HMR | Side effect at module top level (e.g., `console.log` with external dep) | Move to inside function/hook |
| HMR not working in Docker/WSL | `server.host` not set | Add `server: { host: '0.0.0.0' }` |
| `fs.allow` errors | File outside Vite root | Add `server: { fs: { allow: ['..'] } }` |
| TypeScript alias not resolving | `tsconfig.json` paths missing | Sync `paths` with `vite.config.ts` aliases |

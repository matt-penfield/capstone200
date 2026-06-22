## Fast Forward Logistics Dashboard

This project is intentionally set up at the workspace root so all commands run from here.

### First 60 Seconds

1. Open a terminal in this root folder.
2. Run `npm install`.
3. Run `npm run dev`.
4. Open http://localhost:5173/.
5. Start editing `src/App.vue` for the main dashboard.

### Run From Root

```bash
npm install
npm run dev
```

### Root Layout

```text
.
|-- index.html
|-- package.json
|-- vite.config.ts
|-- tsconfig.json
|-- tsconfig.node.json
|-- vercel.json
|-- src/
|   |-- main.ts
|   |-- App.vue
|   |-- env.d.ts
|   `-- data/
|       `-- metrics.json
|-- public/
|-- BRIEF.md
`-- README.md
```

### What Runs From Here

- `npm run dev`: starts Vite dev server from root.
- `npm run build`: builds production assets.
- `npm run preview`: serves the built `dist/` output locally.

### Key Files

- `package.json`: scripts and dependencies for Vuetify, icons, and charts.
- `src/main.ts`: app bootstrap and Vuetify setup.
- `src/App.vue`: full dashboard layout with app bar, KPI cards, and charts.
- `src/data/metrics.json`: 12 months of fake logistics data (Jan–Dec 2025).

### Stack

- Vue 3 + Vite + TypeScript
- Vuetify 3
- Material Design Icons (`@mdi/font`)
- Chart.js + vue-chartjs

# Nuxt.js Project with UI Components

This project is built using [Nuxt.js](https://nuxt.com) and [Nuxt UI](https://ui.nuxt.com), providing a modern and efficient development experience for building user interfaces.

## Features

- **Nuxt.js Framework**: Server-side rendering, static site generation, and more.
- **Nuxt UI**: Pre-built UI components for faster development.
- **Pagination**: Custom pagination with "Previous" and "Next" buttons.
- **Sorting**: Column-based sorting for tables.
- **Dark Mode Support**: Fully responsive with light and dark themes.
- **Dynamic Data**: Fetch and display data from an external API.

## Setup

Make sure to install the dependencies before starting the project:

```bash
# npm
npm install

# pnpm
pnpm install

# yarn
yarn install

# bun
bun install
```

## Development

Start the development server at `http://localhost:3000`:

```bash
# npm
npm run dev

# pnpm
pnpm run dev

# yarn
yarn dev

# bun
bun run dev
```

## Production

Build the application for production:

```bash
# npm
npm run build

# pnpm
pnpm run build

# yarn
yarn build

# bun
bun run build
```

Preview the production build locally:

```bash
# npm
npm run preview

# pnpm
pnpm run preview

# yarn
yarn preview

# bun
bun run preview
```

## Features Overview

### Table with Sorting and Pagination
- **Sorting**: Click on column headers to sort data in ascending or descending order.
- **Pagination**: Navigate through pages using "Previous" and "Next" buttons.

### Dark Mode
- Automatically adapts to the user's system preferences.
- Smooth transitions between light and dark themes.

### Data Fetching
- Fetches product data from an external API (`https://dummyjson.com/products`).
- Displays data in a responsive table with images, descriptions, and ratings.

## Deployment

Check out the [Nuxt deployment documentation](https://nuxt.com/docs/getting-started/deployment) for detailed instructions on deploying your application.
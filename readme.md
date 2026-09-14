# FastForward Logistics Dashboard

FastForward Logistics is a single-page operations dashboard for reviewing monthly logistics performance across revenue, delivery reliability, delays, fuel usage, driver hours, and customer service activity.

The dashboard is designed for an operations lead who needs a quick view of the business for the full year, with the ability to drill into an individual month when investigating trends or issues.

## Purpose

The dashboard provides a compact analytics view of the logistics network:

- Track monthly and annual revenue.
- Monitor the percentage of deliveries completed on time.
- Identify accumulated delivery delay minutes.
- Review customer service volume, sentiment, and resolution outcomes.
- Compare operational activity against seasonal agricultural demand patterns.

## Data Source

All dashboard data is local and fictional. It is stored in [src/data/metrics.json](src/data/metrics.json), with one record for each month from January through December 2025.

Each monthly record includes:

- Revenue in USD.
- On-time delivery percentage.
- Total delay minutes and daily delay values.
- Customer service calls, including positive and negative sentiment and resolution outcomes.
- Fuel cost and gallons consumed.
- Delivery hours, total minutes, and the variance from expected driver hours.

The dataset includes natural month-to-month variation and seasonal volume increases during the agricultural planting and harvest periods. There are no API calls or external data dependencies.

## Dashboard Features

- Dark Vuetify interface with a persistent desktop navigation sidebar.
- Responsive mobile navigation drawer controlled by menu and close buttons.
- Four summary cards for revenue, on-time percentage, delay minutes, and customer service calls.
- Bar chart for monthly revenue.
- Line chart for on-time delivery percentage.
- Full-width bar chart for customer service sentiment and resolution outcomes.
- Chart hover states and tooltips for inspecting individual data points.
- Responsive layout using Vuetify containers, rows, and columns.

## Interactions

The month picker in the top app bar defaults to **All months**:

- **All months** shows annual summary totals and all twelve months in the charts.
- Selecting a specific month updates every summary card and chart to that month.
- On narrow screens, the sidebar is available through the menu icon and can be dismissed from inside the open drawer.
- Chart hover interactions highlight the relevant bar, line point, and tooltip values.

## Technology

- Vue 3 with TypeScript and `<script setup>`.
- Vite for development and production builds.
- Vuetify 3 for the application shell and responsive UI components.
- Material Design Icons for navigation and dashboard controls.
- Chart.js with vue-chartjs for data visualizations.
- Local JSON data with no backend or API layer.

## Development

Install dependencies:

```bash
npm install
```

Start the local development server:

```bash
npm run dev
```

Create a production build:

```bash
npm run build
```

Preview the production build:

```bash
npm run preview
```

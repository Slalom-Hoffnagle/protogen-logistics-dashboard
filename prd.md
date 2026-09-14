# My Dashboard - FastForward Logistics

## What is this?
- A single page analytics dashboard showing monthly business metrics for a logistics company call FastForward Logistics. Think, shopify admin or a simple google analytics view. 

## Data
- Generate a fake data set as a JSON file (src/data/metrics.json) with 12 months of data (Jan-Dec 2025), each month contains 
    - revenue (dollar amount, think about seasonal agricultural trends like planting and harvest with some variance)
    - on-time percentage (across all deliveries and drivers, need to know percentage of runs on-time an late)
    - minutes of delay (cumulative minutes by day of month for late deliveries, how many minutes were they late? this info is critical to identifying issues and improcving performance)
    - customer service calls (how many customer service calls are being received? Are they judge as positive or negative sentiment? How many were/weren't resolved satisfactorily?)
    - fuel costs (dollar amount and gallons, align with volume of deliveries/revenue) 
    - delivery hours (hours and minutes of driver time to fulfill daily deliveries)
        - delta of expected driver hours (over or under)

## Layout
- v-app-bar down the side. A vertical navigation bar down the left side
- Month picker shoudl default ot ALL months (Year) 
- When a specific month is selected, all and charts should filter to reflect the specified month's data
- Page should include 4 summary cards (v-card). Show the key metrics: revenue, on-time percentage, minutes of delay, customer service calls (with positive and negative resolution totals)
- Below the cards, 2 charts
    - Left: Bar chart showing monthly revenue
    - Right: Line chart showing on-time percentage
- Below that: one full width are chart showing customer service calls.
- Use v-container, v-row, v-col for responsive grid layout

## Interactions
- Month picker in the app bar filters EVERYTHING - Summary cards, charts, etc... all reflect the data from the user's chesen month
- When "All" is selected, summary cards show yearly totals and charts show all twelve months
- Hover on charts should visually highlight where the user's cursor is at on the line, bar, etc...

## Style
- Dark theme by default (Vuetify dark theme)
- Clean, minimal, but thoughtfully handle dense data 
- Charts should use a cohesive color palette - not rainbow
- MObile responsive - cards stack on small screens

## Tech
- Vue 3 + Typescript + Vuetify 3
- Chart.js via vue-chartjs for all charts
- Fake data from a local JSON file (no API calls)
- Single page - no routing needed for this app
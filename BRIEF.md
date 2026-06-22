# Fast Forward Logistics Dashboard - Project Brief

## What is this? 

An analytics dashboard consolidating multiple data sets for ease of use.  

## Data

Generate a fake dataset as a JSON file (src/data/metrics.json). 12 months of data (Jan-Dec 2025) each month containing: 
- Shipment volume (trending upwards with some variation)
- on-time delivery rates (some variance, somewhat lower as shipment volume increases)
- regional performance (some variance)
- open exceptions (some variance, trending upward as shipment volume increases)

## Layout 

- v-app-bar at the top with a dashboard title and a month picker. 
- the month picker should default to showing all months
- Performance Analysis button to the left of the All Months button. When clicked, a modal is generate with text explaining possible reasons behind the open exceptions trend data.
- When a specific month is selected, all cards and charts filter to that month. When "All"  is selected, show the full year. 
- Below the app bar: a row of 4 summary cards (v-card) showing the key metrics (shipment volume, on-time delivery rates, regional performance, open exceptions)
- Below the cards: 4 charts 
    - Left: Bar chart showing shipment volume
    - Right: Line chart showing on-time delivery rates over time
- Below that: 
    - Left: area chart showing open exceptions trend
    - Right: Line chart showing regional performance over time
- use v-container, v-row, v-col, for responsive grid layout

## Interactions

- Month picker in the app bar filters everything - summary cards show that month's numbers, charts highlight or filter to that month.
- When "all" is selected, summary cards show yearly totals/averages and charts show all 12 months
- Cards should show a small up/down arrow or color indicating change from previous month

## Style

- light theme by default (Vuetify light theme)
- Clean, minimal, lots of whitespace
- Charts should use a cohesive color palette
- mobile responsive - cards stack on small screens

## Tech

- Vue 3 + Typescript + Vuetify 3 (all from root to avoid problems)
- Chart.js via vuechartjs for all charts (all from root to avoid problems)
- Fake data from a local JSON file (no API calls)
- Single page, no routing needed for this app
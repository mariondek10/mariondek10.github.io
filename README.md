# Xilva Project - Interactive Globe & Analysis Dashboard

An interactive web application combining a 3D globe visualization with a dynamic dashboard to analyze environmental and investment projects (e.g., reforestation, solar grids).

When a user selects a region on the interactive globe, the Vue.js dashboard updates automatically to display complex project data through beautiful D3.js charts.

## ✨ Features

- **Interactive 3D Globe**: Built with standard web technologies (HTML/JS) and embedded within the app.
- **Dynamic Dashboard**: Built with Vue 3 and TypeScript for a smooth, reactive user experience.
- **Advanced Visualizations (D3.js)**:
  - **Risk Flags**: Displays the number of Green, Mitigated (Blue), Yellow, and Red flags.
  - **Pillars Radar Chart**: Evaluates the project across 6 pillars (Design, Governance, Financial, Impact, Safeguards, Growth).
  - **Diverging Bar Chart**: Compares positive vs. negative findings.
  - **Carbon Impact**: Line chart comparing "Business As Usual" (BAU) vs. Project scenarios over time.
  - **Stakeholder Network**: A force-directed graph illustrating the connections between the project, stakeholders, and SDGs.

## 🛠️ Tech Stack

- **Framework**: [Vue 3](https://vuejs.org/) (Composition API, `<script setup>`)
- **Build Tool**: [Vite](https://vitejs.dev/)
- **Language**: TypeScript
- **Visualizations**: [D3.js](https://d3js.org/)
- **3D Globe**: Custom implementation via iframe (`/public/globe.html`)

## 📊 Data Integration (CSV Format)

The dashboard expects data to be formatted in a specific way. Here is the required structure for the project data (1 row = 1 project):

| Project_ID | Project_Name        | Flag_Green | Flag_Mitigated | Flag_Yellow | Flag_Red | Design_Pos | Design_Neg | ... | BAU_2023 | Proj_2023 | ... | Stakeholders | SDGs            |
| :--------- | :------------------ | :--------- | :------------- | :---------- | :------- | :--------- | :--------- | :-- | :------- | :-------- | :-- | :----------- | :-------------- |
| Madagascar | Madagascar Reforest | 12         | 4              | 2           | 0        | 42         | 5          | ... | 1000     | 1000      | ... | WWF - Gov    | SDG 13 - SDG 15 |

_Note: Stakeholders and SDGs must be separated by a dash (`-`) to be properly parsed by the Network chart._

---

## 🚀 Project Setup

Ensure you have [Node.js](https://nodejs.org/) installed, then run the following commands:

### Install dependencies

```sh
npm install
```

### Compile and Hot-Reload for development

```sh
npm run dev
```

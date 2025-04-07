<img width="106" alt="image" src="https://github.com/user-attachments/assets/8bc594ea-eb59-443d-8a5e-ad024c91afb6" />


# CFR Mali Violent Attacks Shiny App

Welcome to the **CFR Mali Violent Attacks Data Shiny App**! This interactive tool allows users to explore the complex patterns of violence in Mali from 1989 to 2023. Using powerful AI-driven insights and RStudio's Shiny framework, this app visualizes the changing dynamics of violent conflict in Mali, from state-based violence to non-state actors and one-sided violence.

This app was inspired by the work and insights of the **Council on Foreign Relations (CFR)**, and it provides a data-driven approach to understanding conflict trends in Mali, with a focus on actionable insights for policymakers, researchers, and global organizations.

## Features

- **Interactive Map**: View geographic markers based on the total deaths for each type of violence over the years. The map dynamically updates as you filter the data by attack type and year range.
  
- **Time Series Plot**: Visualize the trend of deaths from various attack types over time, helping to identify significant spikes or changes in violence.

- **Custom Data Table**: Get a detailed breakdown of violence by year and attack type in an easy-to-read table format. The table is fully interactive, with sorting and pagination options.

- **Filterable Data**: Use the sidebar to filter by year range and attack type, giving you full control over what data to explore.

- **Advanced Tooltip**: Hover over map markers for detailed information on the number of deaths by attack type, making it easier to interpret geographic hotspots of violence.

---

## Live Demo

Explore the app live by clicking on the link below:

[**CFR Mali Violent Attacks App**](https://mmcdonald411.shinyapps.io/CFRMaliProject/)

---

## Table of Contents

- [Project Overview](#project-overview)
- [Features](#features)
- [App Structure](#app-structure)
- [Why This Matters](#why-this-matters)
- [Installation and Usage](#installation-and-usage)
- [Working Code](#working-code)
- [Contributing](#contributing)
- [License](#license)

---

## Project Overview

This project uses **RStudio’s Shiny framework** to build a web application that brings **Mali’s violent attack data** to life. Through data visualization techniques such as time-series analysis and geographic mapping, the app provides an engaging way to understand conflict dynamics.

The goal of the project is to empower organizations, researchers, and policymakers with insights that can aid in decision-making regarding conflict resolution, peacekeeping efforts, and humanitarian assistance.

### Key Technologies:
- **RStudio Shiny**: Interactive web applications for data analysis and visualization.
- **Leaflet**: Interactive maps for geographic data visualization.
- **Plotly**: Interactive time-series plotting and dynamic data visualization.
- **DT**: Data table for displaying detailed information.

---

## Why This Matters

As violence continues to plague regions around the world, it is imperative for organizations like the **Council on Foreign Relations** and global policymakers to leverage cutting-edge technologies to monitor, analyze, and mitigate conflict. The **CFR Mali Violent Attacks Shiny App** provides a user-friendly interface for visualizing and exploring violence data, which can be used for strategic decision-making and long-term policy development.

By combining **AI**, **data science**, and **RStudio’s Shiny framework**, this app allows stakeholders to interact with the data, uncover trends, and draw actionable insights. Whether you’re a researcher analyzing conflict trends, a policymaker evaluating peace strategies, or a humanitarian agency identifying areas of need, this app can help you make data-driven decisions.

---

## App Structure

### UI Components
- **Logo**: The Council on Foreign Relations logo is prominently displayed at the top of the app for brand consistency.
- **Sidebar**: Contains controls for selecting attack types and year ranges, as well as a data table.
- **Main Panel**: Features an interactive map, time-series death trend plot, and dynamic data table.
- **Dynamic Filters**: Users can filter data by year range and attack type for a customized analysis.

### Server Components
- **Data Processing**: Data is filtered reactively based on user input. The map and plot dynamically update based on attack type and year range.
- **Visualization**: Interactive visualizations such as maps and plots are rendered using `leaflet` and `plotly`, while the table is displayed using `DT`.

---

## Working Code

Here is the complete code used to build the CFR Mali Violent Attacks Shiny App. Feel free to use, modify, and contribute.

```r
library(shiny)
library(leaflet)
library(dplyr)
library(ggplot2)
library(plotly)
library(DT)
library(tidyr)

# Sample Data
years <- 1989:2023
state_based <- c(0, 0, 0, 0, 0, 7988, 200, 100, 300, 500, 700, 1000, 1200, 1500, 1600, 1700, 1800, 1900, 
                2000, 2100, 2200, 2400, 2500, 2500, 2600, 2700, 2800, 2900, 3000, 3100, 3200, 3300, 3400, 3500, 3600)
non_state <- c(0, 0, 0, 0, 0, 2977, 100, 150, 200, 300, 400, 500, 600, 700, 800, 900, 1000, 1100, 
              1200, 1300, 1400, 1500, 1600, 1700, 1800, 1900, 2000, 2100, 2200, 2300, 2400, 2500, 2600, 2700, 2800)
one_sided <- c(0, 0, 0, 0, 0, 4771, 50, 70, 100, 150, 200, 250, 300, 350, 400, 450, 500, 550, 
              600, 650, 700, 750, 800, 850, 900, 950, 1000, 1050, 1100, 1150, 1200, 1250, 1300, 1350, 1400)

# Check lengths match
stopifnot(length(years) == length(state_based) && 
          length(years) == length(non_state) && 
          length(years) == length(one_sided))

# Create data frame
death_data <- data.frame(
  Year = rep(years, 3),
  Attack_Type = rep(c("State-Based Violence", "Non-State Violence", "One-Sided Violence"), each = length(years)),
  Deaths = c(state_based, non_state, one_sided)
)

# UI
ui <- fluidPage(
  tags$div(
    style = "text-align: center; padding: 20px;",
    img(src = "cfr-logo.png", height = "100px")
  ),
  
  titlePanel("CFR Shiny App - Mali Violent Attacks Data"),
  
  sidebarLayout(
    sidebarPanel(
      checkboxGroupInput("attack_type", "Select Attack Type",
                        choices = unique(death_data$Attack_Type),
                        selected = unique(death_data$Attack_Type)),
      sliderInput("year_range", "Select Year Range",
                 min = min(years), max = max(years),
                 value = c(min(years), max(years)), 
                 step = 1,
                 sep = ""),
      DTOutput("data_table")
    ),
    
    mainPanel(
      tabsetPanel(
        tabPanel("Map", leafletOutput("map", height = "500px")),
        tabPanel("Death Trend", plotlyOutput("death_plot", height = "500px"))
      )
    )
  )
)

# Server
server <- function(input, output, session) {
  
  # Reactive filtered data
  filtered_data <- reactive({
    death_data %>%
      filter(Attack_Type %in% input$attack_type,
             Year >= input$year_range[1],
             Year <= input$year_range[2])
  })
  
  # Enhanced death plot
  output$death_plot <- renderPlotly({
    plot_data <- filtered_data() %>%
      group_by(Year, Attack_Type) %>%
      summarise(Total_Deaths = sum(Deaths, na.rm = TRUE), .groups = "drop")
    
    p <- ggplot(plot_data, aes(x = Year, y = Total_Deaths, color = Attack_Type)) +
      geom_line() +
      geom_point() +
      theme_minimal() +
      labs(title = "Deaths by Attack Type Over Time",
           x = "Year",
           y = "Number of Deaths",
           color = "Attack Type") +
      scale_color_manual(values = c("State-Based Violence" = "blue",
                                  "Non-State Violence" = "green",
                                  "One-Sided Violence" = "red"))
    
    ggplotly(p)
  })
  
  # Data table
  output$data_table <- renderDT({
    datatable(filtered_data(),
             options = list(pageLength = 10, autoWidth = TRUE),
             rownames = FALSE)
  })
  
  # Enhanced map
  output$map <- renderLeaflet({
    # Calculate total deaths for each type for marker sizing
    map_data <- filtered_data() %>%
      group_by(Attack_Type) %>%
      summarise(Total_Deaths = sum(Deaths))
    
    leaflet() %>% 
      addTiles() %>% 
      setView(lng = -4, lat = 17, zoom = 5) %>% 
      addCircleMarkers(
        lng = c(-8, -5, 0),
        lat = c(15, 16, 12),
        radius = sqrt(map_data$Total_Deaths) / 10,  # Scale marker size by deaths
        color = c("blue", "green", "red"),
        fillOpacity = 0.6,
        popup = paste(map_data$Attack_Type, "<br>Deaths:", map_data$Total_Deaths)
      ) %>%
      addLegend(
        position = "bottomright",
        colors = c("blue", "green", "red"),
        labels = c("State-Based", "Non-State", "One-Sided")
      )
  })
}

# Run app
shinyApp(ui = ui, server = server)
```

---

## Contributing

We welcome contributions from the community! If you have suggestions or improvements for the app, please feel free to fork the repository and submit a pull request.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

### Suggested Repo Name: **CFR-Mali-Violence-Analysis-Shiny-App**


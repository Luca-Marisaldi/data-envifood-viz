```{r}
###------------------Source----------------------


#https://ourworldindata.org/environmental-impacts-of-food


###------------------Load libraries--------------


library("extrafont")
library("ggplot2")
library("readr")
library("cowplot")
library("here")
library("dplyr")

#to load fonts
#loadfonts()


###------------------Data import----------------


water <- read_csv(here("Water usage", "water-withdrawals-per-kg-poore.csv"))
scarcity.water <- read_csv(here("Water usage", "scarcity-water-per-kg-poore.csv"))


###------------------Selection of food----------


items <- c(
  "Beef (dairy herd)", "Beef (beef herd)", "Apples", "Cheese", "Eggs", "Pig Meat", "Rice",
  "Tomatoes", "Peas","Poultry Meat", "Tofu", "Maize", "Potatoes", "Prawns (farmed)", 
  "Dark Chocolate", "Barley", "Nuts"
)


water.filt <- water |>
  filter(Entity %in% items)


###------------------Setting up the theme--------


theme_set(theme_minimal(base_family = "Spline Sans"))
theme_update(
  plot.title = element_text(family = "IBM Plex Serif", size = 11, colour = "grey30", face = "bold"),
  plot.subtitle = element_text(family = "IBM Plex Serif", size = 9, colour = "grey30"),
  panel.grid.minor = element_blank(),
  panel.grid.major = element_blank(),
  axis.line.x = element_line(color = "grey30", linewidth = .4),
  axis.ticks.x = element_line(color = "grey30", linewidth = .4),
  plot.margin = margin(10, 20, 10, 20)
)


###--------Graph for absolute water consumption--------


p1 <- water.filt |>
  rename(water.consumption = `Freshwater withdrawals per kilogram (Poore & Nemecek, 2018)`) |>
  mutate(Entity.ordered = reorder(Entity, water.consumption)) |>
  ggplot(aes(x = Entity.ordered, y = round(water.consumption, 0), fill = water.consumption)) +
  geom_bar(stat = "identity") +
  geom_text(aes(label = paste0(format(round(water.consumption, 0), big.mark = ","), "L")),
    hjust = -0.1, colour = "grey30", size = 3.5
  ) +
  scale_fill_gradient(low = "dodgerblue1", high = "firebrick2") +
  scale_y_continuous(
    expand = expansion(0, 0), limits = c(0, 7000),
    breaks = seq(0, 7000, 1000),
    labels = function(x) format(x, big.mark = ",", scientific = FALSE)
  ) +
  theme(plot.margin = unit(c(1, 1, 1, 1), "lines"),
        legend.position = "none") +
  coord_flip() +
  labs(
    y = "", x = "", title = "Freshwater withdrawals per kilogram of food product",
    subtitle = "Freshwater withdrawals are measured in liters per kilogram \n of food product"
  )

p1


###------Graph for relative (scarcity-weighted) water usage-------


scarcity.water.filt <- scarcity.water |>
  filter(Entity %in% items)
  
p2 <- scarcity.water.filt |>
  rename(water.consumption.corrected = `Scarcity-weighted water use per kilogram (Poore & Nemecek, 2018)`) |>
  mutate(Entity.ordered = reorder(Entity, water.consumption.corrected)) |>
  ggplot(aes(x = Entity.ordered, y = water.consumption.corrected,
             fill = water.consumption.corrected)) +
  geom_bar(stat = "identity") +
  geom_text(aes(label = paste0(format(round(water.consumption.corrected, 0), big.mark = ","), "L")),
    hjust = -0.1, colour = "grey30", size = 3.5
  ) +
  scale_fill_gradient(low = "dodgerblue1", high = "firebrick2") +
  scale_y_continuous(
    expand = expansion(0, 0), limits = c(0, 300000),
    breaks = seq(0, 300000, 50000),
    labels = function(x) format(x, big.mark = ",", scientific = FALSE)
  ) +
  theme(plot.margin = unit(c(1, 1, 1, 1), "lines"),
        legend.position = "none") +
  coord_flip() +
  labs(
    y = "", x = "", title = "Scarcity-weighted water use per kilogram of food product",
    subtitle = "Scarcity-weighted water use represents freshwater use weighted by local \n water scarcity. This is measured in liters per kilogram of food product"
  )

p2


###--------Creating plot------------------


#as a solution to put a little bit of space at the right margin I insert a dummy plot with NULL
#in position 3, so as to add extra dummy space, see also how to adjust it
#with relative width (rel_width)
comparison <- plot_grid(p1, p2, NULL, ncol = 3, rel_widths = c(1, 1, 0.15))


###--------Export-------------

ggsave(filename = here("Water usage", "water_usage.png"), comparison, height = 1000, 
          width = 2000, units = "px", bg = "white", scale = 3.5, dpi = 600)

```


![water_usage](https://github.com/Luca-Marisaldi/data-envifood-viz/assets/60885799/79bcec03-861d-4023-9ea8-2b02e89c38c0)

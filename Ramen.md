Ramen ratings
================

This was a short analysis I did using an old TidyTuesday dataset for
ramen ratings. It was an enjoyable experience, as I figured out how to
graph data on maps\! These were the resulting graphs.

``` r
ramen <- read_csv("https://raw.githubusercontent.com/rfordatascience/tidytuesday/master/data/2019/2019-06-04/ramen_ratings.csv") %>%
  mutate(
    variety = factor(variety),
    style = factor(style)
  ) %>% na.omit()
```

## Top 25 ramen brands by review number:

It looks like Nissin was the winner here.

``` r
ramen %>% 
  group_by(brand) %>% 
  summarise(reviews = sum(review_number))%>% 
  arrange(desc(reviews)) %>% 
  top_n(25) %>% 
  ggplot(aes(brand, reviews, fill = brand))+
  geom_col()+
  theme(axis.text.x = element_text(angle = 270))
```

![](Ramen_files/figure-gfm/unnamed-chunk-3-1.png)<!-- -->

## Top ramen brand for each country by review count

Unfortunately, due to missing data I could only visualize the top brands
for their respective origin countries.

``` r
world_map <- map_data("world")

x <- ramen %>% 
  group_by(country, brand) %>% 
  summarise(reviews = sum(review_number)) %>% 
  arrange(desc(reviews)) %>% 
  top_n(1) 

top_brand <- x[,1:2]

plot1 <- full_join(top_brand, world_map, by = c("country" = "region"))

ggplot(plot1, aes(long, lat, group = group))+
  geom_polygon(aes(fill = brand))+
  labs(x = "Longitude",
       y = "Latitude",
       title = "Top ramen brand in origin country",
       subtitle = " according to review count")
```

![](Ramen_files/figure-gfm/unnamed-chunk-4-1.png)<!-- -->

## Top ramen brands by average rating

MyKuali wins this round. I filtered to only include brands with over 100
reviews to root out brands with lower review counts who had all 5-stars.

``` r
ramen %>% 
  group_by(brand) %>% 
  summarise(average_rating = mean(stars),
            reviews = sum(review_number))%>% 
  filter(reviews > 100) %>% 
  arrange(desc(average_rating))%>% 
  top_n(25) %>% 
  ggplot(aes(brand, average_rating, fill = brand))+
  geom_col()+
  theme(axis.text.x = element_text(angle = 270))+
  labs(x = "Brand",
       y = "Average rating")
```

![](Ramen_files/figure-gfm/unnamed-chunk-5-1.png)<!-- -->

## Top ramen brands in their origin countries according to average rating

There don’t appear to be many differences from the map using brands
sorted by review count.

``` r
top_brand_rating <- ramen %>% 
  group_by(country, brand) %>% 
  summarise(average_rating = mean(stars),
            reviews = sum(review_number))%>% 
  filter(reviews > 100) %>% 
  arrange(desc(average_rating))%>% 
  top_n(1)

top_brand_rating <- top_brand_rating[,1:2]

plot2 <- full_join(top_brand_rating, world_map, by = c("country" = "region"))

ggplot(plot2, aes(long, lat, group = group))+
  geom_polygon(aes(fill = brand))+
  labs(x = "Longitude",
       y = "Latitude",
       title = "Top ramen brand in origin country",
       subtitle = " according to average rating",
       caption = "(taking brands with over 5000 reviews)")
```

![](Ramen_files/figure-gfm/unnamed-chunk-6-1.png)<!-- -->

## Top ramen style by country (according to review count)

Finally, I looked at preferred ramen style.

``` r
top_style <- ramen %>% 
  group_by(country, style) %>% 
  summarise(reviews = sum(review_number))%>%
  arrange(desc(reviews)) %>% 
  top_n(1)

top_style <- top_style[,1:2]  

plot3 <- full_join(top_style, world_map, by = c("country" = "region"))

ggplot(plot3, aes(long, lat, group = group))+
  geom_polygon(aes(fill = style))+
  labs(x = "Longitude",
       y = "Latitude",
       title = "Top ramen style in orgin country",
       subtitle = " according to review count")
```

![](Ramen_files/figure-gfm/unnamed-chunk-7-1.png)<!-- -->

## A couple of linear regressions.

Finally, I ran a couple regressions, which weren’t very accurate, as
indicated by the low adjusted R-squared values.

``` r
topramenbrand <- ramen %>% 
  group_by(brand) %>% 
  summarise(reviews = sum(review_number)) %>% 
  arrange(desc(reviews)) %>% 
  top_n(25) %>% 
  pull(brand)

topramen <- ramen %>% 
  filter(brand %in% topramenbrand)

lm <- lm(review_number ~ style + brand, data = topramen )

topramenbrand2 <- ramen %>% 
  group_by(brand) %>% 
  summarise(average_rating = mean(stars),
            reviews = sum(review_number))%>% 
  filter(reviews > 100) %>% 
  arrange(desc(average_rating))%>% 
  top_n(25) %>% 
  pull(brand)

topramen2 <- ramen %>%
  filter(brand %in% topramenbrand2) %>% 
  mutate(brand = factor(brand))

lm2 <- lm(review_number ~ brand, data = topramen2)

tab_model(lm, lm2, p.style = "stars", show.ci = FALSE)
```

<table style="border-collapse:collapse; border:none;">

<tr>

<th style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm;  text-align:left; ">

 

</th>

<th colspan="1" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">

review\_number

</th>

<th colspan="1" style="border-top: double; text-align:center; font-style:normal; font-weight:bold; padding:0.2cm; ">

review\_number

</th>

</tr>

<tr>

<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  text-align:left; ">

Predictors

</td>

<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">

Estimates

</td>

<td style=" text-align:center; border-bottom:1px solid; font-style:italic; font-weight:normal;  ">

Estimates

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

(Intercept)

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

1709.19 <sup>\*\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

1524.27 <sup>\*\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

style \[Box\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

636.25 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

style \[Cup\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

144.53 <sup>\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

style \[Pack\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-170.71 <sup>\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

style \[Tray\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-293.91 <sup>\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Acecook\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

732.80 <sup>\*\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

935.06 <sup>\*\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Indomie\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-720.81 <sup>\*\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-658.41 <sup>\*\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[JML\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-119.39 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-37.69 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[KOKA\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

878.83 <sup>\*\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

1008.34 <sup>\*\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Maggi\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

51.52 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

98.91 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Mama\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-585.29 <sup>\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-477.10 <sup>\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[MAMA\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

555.51 <sup>\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

614.11 <sup>\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Mamee\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-286.94 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-174.89 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Maruchan\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-44.90 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

80.02 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Master Kong\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

18.20 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

63.49 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[MyKuali\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

443.81 <sup>\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

597.66 <sup>\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Myojo\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

75.76 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

145.23 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Nissin\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

41.79 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

217.53 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Nongshim\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-445.31 <sup>\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-319.07 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Ottogi\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-615.39 <sup>\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-525.16 <sup>\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Paldo\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-589.49 <sup>\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

\-494.24 <sup>\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Samyang Foods\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

548.71 <sup>\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

644.64 <sup>\*\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Sapporo Ichiban\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

142.54 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

285.25 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Sau Tao\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

244.43 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

303.57 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Uni-President\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

688.16 <sup>\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

818.11 <sup>\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Vedan\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

890.36 <sup>\*\*\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

1034.45 <sup>\*\*\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Wei Lih\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

335.24 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

388.85 <sup></sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Wu-Mu\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

538.12 <sup>\*</sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

552.34 <sup>\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; ">

brand \[Yum Yum\]

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

369.71 <sup></sup>

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:center;  ">

471.09 <sup>\*</sup>

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm; border-top:1px solid;">

Observations

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="1">

1532

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left; border-top:1px solid;" colspan="1">

1532

</td>

</tr>

<tr>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; text-align:left; padding-top:0.1cm; padding-bottom:0.1cm;">

R<sup>2</sup> / R<sup>2</sup> adjusted

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="1">

0.227 / 0.213

</td>

<td style=" padding:0.2cm; text-align:left; vertical-align:top; padding-top:0.1cm; padding-bottom:0.1cm; text-align:left;" colspan="1">

0.205 / 0.192

</td>

</tr>

<tr>

<td colspan="3" style="font-style:italic; border-top:double black; text-align:right;">

  - p\<0.05   \*\* p\<0.01   \*\*\* p\<0.001
    </td>
    </tr>

</table>

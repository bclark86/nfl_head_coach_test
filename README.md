<!-- README.md is generated from README.Rmd. Please edit that file -->
Hypothesis Testing NFL Coaches for 2019
---------------------------------------

I am a big fan of football, behavioral economics, and statistics. So naturally, when I read this article from [Profootballtalk.com](https://profootballtalk.nbcsports.com/2019/01/12/nfl-will-have-20-offensive-head-coaches-only-12-from-defense/) about the offensive trend in head coaching hires, I wondered if it was a case of [recency bias](https://www.davemanuel.com/investor-dictionary/recency-bias/).

Per the author:

> In the past, there was always close to an even split between offensive and defensive head coaches. From 2002 to 2016 there were always between 14 and 17 coaches whose background was in offense. In 2017 that number hit 18 for the first time, in 2018 it remained at 18, and in 2019 the number will hit an all-time high of 20 offensive head coaches.

Perfect, this sets up our hypothesis test. Per an article written recently by [Cassie Kozyrkov](https://hackernoon.com/statistical-inference-in-one-sentence-33a4683a6424), statistical inference boils down to one question posed by [R.A. Fisher](https://en.wikipedia.org/wiki/Ronald_Fisher), credited for the creation of modern statistical inference:

> “Does the evidence that we collected make our null hypothesis look ridiculous?”

So now we have the basis for our hypothesis test:

-   H0: Preference for selecting offensive coaches is 50%
-   H1: Perefernce for selecting offensive coaches is not 50%

To test our hypothesis, we are going to build a dataset of offensive and defensive coaches for 2019.

``` r
offensive_coaches <- rep("offensive", 20)
defensive_coaches <- rep("defensive", 12)

coaches <- data_frame(
  specialty = c(offensive_coaches, defensive_coaches)
)
```

Next we will use the `infer` package within the `tidymodels` package to simulate a null universe where offensive and defensive coaches have a 50/50 chance of being assigned to an NFL team. We then compare our observed 2019 offensive coach percentage to what we would see in the null universe across 10,000 simulations.

``` r
p_hat <- coaches %>%
  summarize(mean(specialty == "offensive")) %>%
  pull()

null_distn <- coaches %>%
  specify(response = specialty, success = "offensive") %>%
  hypothesize(null = "point", p = .5) %>%
  generate(reps = 10000, type = "simulate") %>%
  calculate(stat = "prop")

ggplot(null_distn, aes(x = stat)) +
  geom_bar(color = "black", fill = "grey") +
  geom_vline(xintercept = p_hat, color = "red", linetype = "dashed") +
  scale_x_continuous(labels = scales::percent_format(accuracy = 1)) +
  labs(
    title = "Simulation of NFL Head Coach Selection",
    subtitle = "Assumes 50% chance of offensive specialty",
    x = "Offensive Coach Percentage",
    y = "Simuation Trials"
  ) +
  annotate("text", x = .73, y = 1000, label = "2019 offensive\ncoach percentage")
```

<img src="\simulation-1.png" width="70%" style="display: block; margin: auto;" />

A percentage of **62.5%** does seem a little unusal. But just how unusal? We can calculate the percentage of simulations that produced a value as extreme or more than our 2019 observed statistic and that becomes our p-value. We multiple our finding by 2 to test both sides of the tail to confirm that there isn't a preference for hiring defensive coaches either.

``` r
p_val <- null_distn %>%
  summarize(p_value = mean(stat >= p_hat) * 2)

p_val
```

    ## # A tibble: 1 x 1
    ##   p_value
    ##     <dbl>
    ## 1   0.213

For our experiment, we see a value as extreme or more oftern about **21.3%** of the time. Looking at it this way, while NFL head coaches may be trending more offensively, it doesn't seem THAT unusual.

# A/B Testing Project
## Using Home Decor Marketing Data

![image](https://user-images.githubusercontent.com/14934475/224817007-9265d831-fd33-42c1-8c24-dfecd794abee.png)

In this project, I will conduct A/B Testing in R Studio to inform data-driven decisions in a home decor company's marketing strategy.

### What do I mean by A/B testing?

What this means is - I will help a company conduct an experiment and try two different strategies to pushing sales on their app. Instead of releasing it to **all** their user base, they're asking me, the data scientist, to select two smaller samples within their customer base to see how the customers respond to these new marketing notifications. I'm creating a business-safe test run for them to decide how they want to approach their greater user base. If things go well, sales will increase. If things go poorly, users will uninstall the app and they'll lose return business. A mix of both is acceptable as long as it's the correct ratio. I will be the company's guide in picking their strategy.

### The Prompt

I'll use this as my guide throughout the project.

> Decco is an online retailer that sells home decor items. They recently added "Lamps" as a new category of products. In order to generate awareness and boost sales, they want to launch a promotional campaign through their mobile app. Their notifications have had good success in the past and they are considering a 10 euro discount if users click through to the in-app notification. At the same time, they want to be judicious about any features or releases when it comes to their app because they know that the customer lifetime value of people who have installed the app is much higher. Their goal is to drive lamp sales without causing users to uninstall the app.

### Setting hypothesis

Before I do anything I'll want to create a hypothesis, which will be the logical foundation of our testing.

>Because we have seen good success through  in-app notifications in the past, if we send an 
>in-app notification with a promotional offer for Lamps, then the percent of users that purchase 
>from the Lamp Category will increase. 

Our null hypothesis, then, would be:

>If we send an in-app notification with a promotional offer for Lamps, the percent of users that purchase 
>from the Lamp Category will not change.

We'll break this hypothesis down by metrics so it is measurable.

>Primary Metric - Transaction Rate i.e. percent of users that will make a purchase

>Secondary Metric - Purchase Value

>Other metric - Uninstall Rate

### Setting up the R workspace

First I'll import the test data into the R environment. Don't worry about that Texas policing data there - just keeping an eye on law enforcement behaviour in my hometown. ;)

![image](https://user-images.githubusercontent.com/14934475/224819188-1b5f9cf2-c2fc-4dbc-9d5a-1db119398fa3.png)

### Calculating Sample Size

It's necessary to calculate what is the minimum amount of people we need to test on in order for the test group to represent the entire customer base. This is called calculating sample size. There are tests we can run to calculate this. My fellow psychology research graduates are probably familiar with GPower. The concept is the same. although since I already have R Studio downloaded no need to download a new programme.

Let's get started finding the sample size. We'll use the pwr package:

```
install.packages("pwr")
library(pwr)
```

And next we'll tell pwr what our specifications are. I'll break this down for you in a minute.

```
control = 0.101
uplift = .2
variant = (1 + uplift)*control
effect_size <- ES.h(control, variant)
sample_size_output <- pwr.p.test(h = effect_size,
                                 n = ,
                                 sig.level = 0.05,
                                 power = 0.8)
```
The following line of code sets the conversion of control - we know from the stakeholder that it's 10.1%, or .101 here.

![image](https://user-images.githubusercontent.com/14934475/224823333-61e974f1-e745-4123-9121-701a0b9dad24.png)

This next line of code shows the minimum detectable effect we are interested in. We want to see if the treatment will outperform the control by at least 20%, and we're not interested in anything less noticeable than that. So we're telling our model that.

![image](https://user-images.githubusercontent.com/14934475/224823507-a69d131c-a57e-4e8c-8cd4-5d81c9f7a321.png)

Those two prior lines are the building blocks for specifying our variant:

![image](https://user-images.githubusercontent.com/14934475/224824110-a4dff0ef-56f5-4973-ae50-8b91623e9f78.png)

Next I'll specify the effect size I'm interested in, or how much of an impact I need the results to make to be of interest:

![image](https://user-images.githubusercontent.com/14934475/224824256-c0f91915-ec28-48f8-a714-b179c5820a5f.png)

And finally I'll plug it all into a line of code which we'll run to show us our sample size, inclusive of all the above as well as signficance level (just going with a good ol' < .05).

![image](https://user-images.githubusercontent.com/14934475/224824610-b522564c-2da2-4dc0-81ba-720e869f2efb.png)

```
sample_size_output <- ceiling(sample_size_output$n)
sample_size_output 
```

And there's our sample size.

![image](https://user-images.githubusercontent.com/14934475/224824980-0c2e9636-d3aa-41db-8986-25fd66eca26b.png)

What this means is, in order to get a fair representation of Decco's user base without reaching all of their users, we need to reach at least 1896 users. Making good progress!

![image](https://user-images.githubusercontent.com/14934475/224825638-9751b3a4-0534-4471-b97d-0b988af20370.png)

### Understanding the data

```
getwd()
setwd("~/Downloads")

ABdf = read.csv("_AB_Test_Data.csv")

str(ABdf)
```

![image](https://user-images.githubusercontent.com/14934475/224827288-f5f0df53-be81-411d-aa64-2bbe72363289.png)

The str function allows me to quick look over the data and summarise it mentally. A quick data check is good practice, for example I can see that users who have no transactions also have an NA value for purchase value. 

![image](https://user-images.githubusercontent.com/14934475/224827596-d301a3bc-3258-47e3-9df3-a9ae6e295382.png)

I can double check that that is the case with more than 5 instances by opening the dataframe:

![image](https://user-images.githubusercontent.com/14934475/224829036-a0187107-dc11-4c70-8a73-dcaa26eef99e.png)

Seems fine.

I'll examine our variables of interest. 

>Treatment indicator - allocation

(This is whether the user is experiencing the A or B version of the test)

>Response variables - addtocart_flag, transaction_flag, purchase_value

(These are the sales behaviours the stakeholder is so interested in)

>Baseline variables - active_6m, days_since 

(This is whether the user was active on the app in the last 6 months, and how many days since they have been active)

>Other - uninstall_flag

(This is the uninstall behaviour the stakeholder is trying to prevent or reduce while driving sales)

```
summary(ABdf[,c("active_6m", "addtocart_flag","transaction_flag", "uninstall_flag", 
              "purchase_value", "days_since")])
             
```

Here are those summary stats:

![image](https://user-images.githubusercontent.com/14934475/224830068-7b518a83-7a0a-4ba4-857e-114ac148d0aa.png)

Here are some customer behaviour insights we can draw from those stats:

> 75% of users have been active on the app in the last 6 months

> 25.63% of users added an item from the Lamps category to their cart

> 14% of users have purchased from the Lamps category

> There were 4% uninstall rates while the test was running

> Average purchase value was 304.6 euro

### Visualisation via Charts and Plots

I could use some visuals to help myself and the stakeholder understand the distribution of the data. It's easier to make sense of pictures. I'll visualise the continous variables using histograms and scatterplots. I'll need the ggplot2 R package for this.

```
install.packages("ggplot2")
library(ggplot2)
library(dplyr)
```

```
ABdf %>% ggplot(aes(x = purchase_value)) +  
  geom_histogram( color="#e9ecef", fill = "#E69F00", alpha=0.6, position = 'identity') +
  scale_fill_manual(values=c("#69b3a2", "#404080")) +
  labs(fill="")
 ```
 
 ![image](https://user-images.githubusercontent.com/14934475/224831321-fcac3ccc-4f89-4a57-b45e-2895c4b3b400.png)

Above is our purchase value distribution. It looks nice and normal, and the mean is around 300, which is true to our summary stats above.

The next bit of ggplot2 code will find the density plot.

```
ABdf %>% ggplot(aes(x = purchase_value)) +  
  geom_density( color="#E69F00")

```

![image](https://user-images.githubusercontent.com/14934475/224831616-2cba7e5c-6d58-4efb-9c51-72c1ca2fb350.png)

This distribution is not surprising as we saw it in the histogram, but it's good to be thorough.

Let's have a look at the time series app use data.

```
ABdf %>% ggplot(aes(x = days_since)) +  
  geom_histogram( color="#e9ecef", fill = "#56B4E9", alpha=0.6, position = 'identity') +
  scale_fill_manual(values=c("#69b3a2", "#404080")) +
  labs(fill="")
```
![image](https://user-images.githubusercontent.com/14934475/224832050-1b82e990-f4ef-4b5b-9464-929e8ad6488b.png)

It looks like there have been a few peaks in activity. Around 320 days back and 60 days back, it seems that there were peaks in user activity. It's possible that Decco ran promotions during this time. This is worth investigating as part of the data picture.

![image](https://user-images.githubusercontent.com/14934475/224832683-eafa1d6f-7423-4bbf-83a8-19100e2b48c3.png)

And here's the density plot.

```
ABdf %>% ggplot(aes(x = days_since)) +  
  geom_density( color="#56B4E9")
 ```

![image](https://user-images.githubusercontent.com/14934475/224832875-50d23545-8658-4c89-93bc-b82f6b25967d.png)

### Categorical Variable Stats

I'm curious what the categorical variables look like - we only have one, allocation.

```
table(ABdf$allocation)

```

![image](https://user-images.githubusercontent.com/14934475/224833311-bb9ff537-8e30-4c73-b3d3-1e9b736e82f9.png)

That looks about right.

### Verify Randomisation

We'll want to check next if the randomisation was done correctly. Randomisation is used to select the samples for the two variations - not just the "how many", but the "whom". We want a nice variety of participant behaviours. We can check that by taking some baseline variables and seeing how they are distributed between the treatment and control. If the distribution between each group is similar, it's fair to expect that the randomisation was done correctly.

This next line of code will compare means between variables across both groups.

```
ABdf %>% group_by(allocation) %>% summarise(mean(active_6m), mean(days_since))
```

![image](https://user-images.githubusercontent.com/14934475/224834006-d9e0f255-703a-4237-9e1b-4200f2517424.png)

![image](https://user-images.githubusercontent.com/14934475/224834599-3b51e4b8-7400-4135-a5ba-25387b025bd2.png)

The means in control and treatment for both variables are exactly the same; this is good because it shows that the randomisation was likely done correctly, lending validity to our study. We can get a bit more thorough with this check by testing not just for means, but for distributions of the variables across the two groups. We'll use more visualisations to help us make sense of those data.

```
ABdf %>% ggplot(aes(x = days_since, fill = allocation)) +  
  geom_histogram( color="#e9ecef", alpha=0.6, position = 'identity') +
  scale_fill_manual(values=c("#69b3a2", "#404080")) +
  labs(fill="")
```
![image](https://user-images.githubusercontent.com/14934475/224835104-12ae215d-2b36-44f2-8c0a-ab4df1c5f9d9.png)

We can see that the distributions are almost completely identical, which is great. Randomisation has most likely been done correctly.

### Main Analysis

I'm ready for the final analysis to see if the treatment outperforms the control. I'll revisit my hypothesis.

Hypothesis:

>If we send an 
>in-app notification with a promotional offer for Lamps, then the percent of users that purchase 
>from the Lamp Category will increase. 

Null hypothesis:

>If we send an in-app notification with a promotional offer for Lamps, the percent of users that purchase 
>from the Lamp Category will not change.

Like so much of stats, it comes down to a means test.

![image](https://user-images.githubusercontent.com/14934475/224835995-6e51007f-a418-45d0-9898-91a525c4fa44.png)

```
ABdf %>% group_by(allocation) %>% summarise(mean(addtocart_flag), mean(transaction_flag), 
                                              mean(purchase_value, na.rm = TRUE))
```

![image](https://user-images.githubusercontent.com/14934475/224836212-b381c28b-a81f-4414-b823-ff29dc637cfa.png)

![image](https://user-images.githubusercontent.com/14934475/224836603-0c750869-1598-4bf4-8a33-66025ffa48a6.png)

Well, maybe not yet. There are a lot more things to check. But these are early promising signs. It looks like in all variables, the treatment is outperforming the control. This means that where Decco sent notifications offering a 10 euro discount for Lamps category app clickthrough sales. Here are some key trends:

>Cart-adding was around 24% in control and 28% in treatment

>Purchase percentage was around 10% in control and around 18% in treatment

>Purchase values were an average of 272 euro in control and 323 euro in treatment. 

Those are good preliminary results, but I still need to check for statistical significance. We'll run a crosstabulation using prop.test.

```
prop.test(xtabs(~ allocation + transaction_flag, data = ABdf)[,2:1])
```

Reading the results, we can see that the p-value is really small, and the confidence interval does not contain zero. These are good signs and they mean our results are significant.

![image](https://user-images.githubusercontent.com/14934475/224837879-2f5ab17f-5298-4e5f-b6bc-0098b76f1c6c.png)

Those confidence intervals tell me that the treatment group performed between 6.9% and 7.8% better than the control group. Those are sales measurements. Let's examine the Add to Cart variable next.

```
prop.test(xtabs(~ allocation + addtocart_flag, data = df)[,2:1])
```
![image](https://user-images.githubusercontent.com/14934475/224838613-d72cd03c-4c92-40b3-b183-ae91332dff38.png)

Reading results here, I can see once again that the p-value is very small (indicating significance) and the confidence interval does not contain zero. These results tell us that in terms of the Add to Cart behaviour, the treatment had a 3.6% to 4.7% effect.

Next I'll check for differences in the means between the groups for the purchase value variable.

```
t.test(purchase_value ~ allocation, data = ABdf)
```
![image](https://user-images.githubusercontent.com/14934475/224839270-19d55a17-501f-4f00-a610-305d80b0b303.png)

Again we have a small p-value, indicating significance, and a difference between means. Purchase value is also affected by the treatment.


So we saw increases in sales and user interest behaviour; that's wonderful. However, I also want to compare to uninstalls to make sure the business isn't losing too many valuable customers. It's not a win if other unreconcilable losses are occurring. 

## Uninstall Analysis 

I'll check next for uninstall activity associated with treatment and control.

```
ABdf %>% group_by(allocation) %>% summarise(mean(uninstall_flag))
```

![image](https://user-images.githubusercontent.com/14934475/224839766-c0d979b3-7bc0-4c63-b0d3-530df2516e14.png)

At a glance I can see that uninstalls are already quite a bit higher (~5% vs. 3%). I'll run additional tests to see if that number is too high. I'll check the statistical significance of those means differences.

```
prop.test(xtabs(~ allocation + uninstall_flag, data = ABdf)[,2:1])
```

![image](https://user-images.githubusercontent.com/14934475/224840063-d53f5f1e-969f-43fa-b16b-e51dfa55bb0a.png)

That's a low p-value. It's not looking so good for the treatment, all of a sudden. I'll need to adjust my conclusion.

**While the test is driving up percentage of purchases and purchase value, it is also driving up the uninstalls. And based on the guidance at the beginning, we have a constraint that we cannot have the uninstalls going up. Therefore, in this case we will not implement the change, even though the treatment outperformed the control for the respond variables.**

What I have done here as a data analyst is damage control and preventative advice: I have analysed the data to advise on the risk of implementing a proposed business strategy to the entire user base. Minimal setbacks have happened in terms of user retention and new strategies to drive Lamps sales can be formed based on these data insights.

![image](https://user-images.githubusercontent.com/14934475/224841566-f3e89e33-d13c-4393-9d1c-404a8f7f25ba.png)

























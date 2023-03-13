# A/B Testing Project
## Using Lamp Purchase Mobile Application Data

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

Next we'll specify the effect size we're interested in, or how much of an impact we need our results to make to be of interest:

![image](https://user-images.githubusercontent.com/14934475/224824256-c0f91915-ec28-48f8-a714-b179c5820a5f.png)

And finally we'll plug it all into a line of code which we'll run to show us our sample size, inclusive of all the above as well as signficance level (just going with a good ol' < .05).

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

We can double check that that is the case with more than 5 instances by opening the dataframe:

![image](https://user-images.githubusercontent.com/14934475/224829036-a0187107-dc11-4c70-8a73-dcaa26eef99e.png)

Seems fine.

Let's examine our variables of interest. 

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





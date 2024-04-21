# What drives the price of a car?
**Alessandro Morato**

**BH-PCMLAI, January 2024**

## 1. Business Understanding

Scope of this analysis is to identify the features that are most effective in driving the price of a used car.

The provided dataset is made of several potential relevant features. A deep understanding of the hidden relations between said features and the target selling price could improve the seller's business. For instance, are there some models particularly desired by the customers? Is there any technical feature that is considered important by the market?

An analysis like this could provide a significant improvement in the revenues of the car dealer. Examples are:

- The seller could have a better understanding of which cars should have/not-have in his parking lots. This would allow him to minimize the unsold cars.

- The seller can know whith "high" confidence which cars can provide higher revenues. The most desidered cars can be sold with higher prices.

- Being used cars, the negotiation seller-customer is key. The seller could have a better understanding about how much he can drop the price, without losses.

These are just some of the potential applications. In general a picture unveiling the dynamics of the market is something highly desired by any professional in any field.

## 2. Data Analysis

The dataset of used cars is studied and analysed with an EDA (Exploratory Data Analysis).

The dataset appears very incomplete, with multiple features poorly filled. The worst one is 'size' with 72% of values missing, but also other fields are not ideal (e.g. 'condition' and 'cylinders' over 40%).

Checking out some items in the dataset, I observed that some "VIN" are duplicated. The "Vehicle Identification Number" identifies each single vehicle, and two cars cannot share the same code. That is why it looked strange to me and I tried to investigate this.
If I pick one duplicated VIN (e.g. 1GYFZFR46LF088296, but these considerations apply for any other duplicated code as well), I get THE SAME car reported multiple times. The car is the same (same model, characteristics and price), but in different locations. This isn't very clear to me: this could mean that this car was inserted in the database many times (after each resell maybe?) or that the car is currently available for purchase in different locations.
I believe these duplicates are not ideal for the analysis: so many identical lines could add unwanted biases and produce poor predictive results. I eliminate the duplicated rows by keeping only the first occurrence for each VIN. The size of the dataset is drastically downsized to 118247 rows, meaning less than 30% of the original dataset.

Before any further action, I would like to have a very first preliminary visualization of the dataset.

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/7d22105a-01b1-4d91-8c7b-773659b54454)

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/3693a3d8-2f8c-46f4-a599-28afb6dd0e74)

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/6aa861b5-a021-44c5-842f-e676aac30366)

The above visualization gives an indication about a possible approach to manage the columns with missing values.

- `size`: drop column

    Very relevant percentage of missing values (74%). I consider the amount of missing information too big to be reconstructed. Also, I believe that part of this information could be somehow found under the 'type' column. I decide to drop the whole column.

- `condition`: drop NaN

    Relevant percentage of missing values (48%). I don't see a clear trend in the histogram (many 'excellent and 'good') so I am afraid of adding an unwanted bias if I replace the missing values with the most common one ('excellent'). I accept to lose half of the dataset and I drop the rows with missing values. I expect `condition` a key factor in the car price, so I don't want either to drop the whole column or to work with bad reconstructed values.

- `cylinders`: drop NaN

    Relevant percentage of missing values (40%). Same considerations of the previous point: I cannot see a clear predominance (4, 6, 8 cylinders very popular) and I am afraid to add a bias. I accept to lose a significant part of the dataset and I drop rows with missing values.

- `paint_color`: fill NaN

    Relevant percentage of missing values (25%). However I believe the color of the car is not the most important factor when buying a second-hand car. Therefore I would try to save all the rows by replacing the missing values with the label 'Unknown'. It will be likely white, black or silver, but maybe the information is not available yet.

- `drive`: fill NaN

    Relevant percentage of missing values (23%). In this case the visualization helps. The most common cathegory is '4wd' (around 40k vehicles) and looking at the 'type' feature i see around the same value if a sum the main 'potential' 4wd types (SUV, pickup, truck). The idea is to fill the missing value based on the 'type' feature. This would also suggests a strong correlation of this feature...

- `type`: drop NaN

    Relatively limited percentage of missing values (14%). I consider the type of car an important feature so I prefer to remove rows without this feature.

- `fuel`: fill NaN

    Very low percentage of missing values (1.4%), but it can be safely filled by 'gas' value.

- `transmission`: fill NaN

    Very low percentage of missing values (1%), but it can be safely filled by 'automatic' value.

- `title_status`: drop NaN

    Very low percentage of missing values (3%). However almost all the cars are under 'clean' group. I'm very tempted to get rid of this column, but I eventually decide to keep it and drop NaN values.

- `year, manufacturer, model,odometer`: drop NaN

    These features present a very limited number of missing values. Dropping or replacing the incomplete rows shouldn't make a huge difference, so I choose the easiest option and I drop them. Regarding `manufacturer` some values are weird (Harley Davidson is not a car maker) or very limited: this could be solved by filtering brands whith less than 100 units in the dataset.

- `id`, `VIN`: drop column

    Unique values for every row, they don't provide much information.

- `region`, `state`: drop columns

    Said features have lost importance after removing the duplicates of VIN.

### Analysis of Numerical Features and Treatment of Outliers
Numerical features are `price`, `odometer` and `year` and they present a significant presence of outliners. After removing said outliners, the numerical features are visualized in the following.

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/4f7a8bbe-f190-4eef-bbb4-6ed99f01b2b3)

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/c53016c1-870a-4f4b-a32a-360af469f239)

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/10fb7b3c-70ed-49e3-97e4-4ead870ce5c5)

Looking at the `price` graph, I see there are some cars extremely cheap (close to 0 USD). Before further proceeding, I decide to inspect a group of very cheap cars (< 1k$) to see if the items seem reasonable. Some rows are totally unrealistic: e.g. a Mercedes E-class (2017) in excellent condition sold for 1 dollar (according to Carvana, similar models are worth >20k USD). After few iterations, I decide to filter all the rows with a price under 2k USD. The most "cheap" cars now are reasonable: there is still a Mercedes on the lowest side of the spectrum, but this time it's an old model (1999), in fair condition and with more than 230k miles. The updated `price` distribution is in the following.

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/aae73f02-4df4-4499-8be5-e99b1511bab1)

The `price` feature looks very right-skewed (positively-skewed). A 'log' transformation will be applied before the modeling phase, to improve the skewness.

### Visualization of the Dataset
Before encoding the non-numerical features and shifting to the modeling phase, a visualization of the 'cleaned' dataset is provided.

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/3f27b2ac-48f5-4dcd-9407-0a36d9d27d82)

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/49d5087e-10dd-405b-b1ea-75f3fd1b6cc2)

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/b2b6f7f5-b914-4da9-a78a-7afa57fc0f68)

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/bbf99b7a-b553-47f2-8a40-3a1be2e62354)

After cleaning the dataset, the visualization of the features in function of the `price` concludes the EDA. All the following considerations are based on average price values.

- `year` looks a key feature. Price seems to decay esponentially increasing the age of the car.

- `condition` shows a relevant distinction between the 'good' states (new, like new, good and excellent) and 'bad' states.

- `manufacturer` presents several brands, each one even with a significant price variance. However, the most expensive brands agree with other features below. E.g. 'ram' and 'gmc' agree with 'truck' and 'pickup' as the most expensive types of cars, under feature `type`. Same for 'alfa-romeo' and 'porsche' with type 'coupe'.

- `model` allows to identify the most expensive cars. Considering the top 20 models, in terms of availability, the most expensive ones are the 'sierra 1500', 'tacoma', 'silverado 1500' and 'f-150'. All these models are big-sized trucks.

- `cylinders`suggest a division into 2 groups: less than 6 cylinders, and 6 or more cylinders. 

- `fuel` indicates that diesel vehicles are more expensive.

- `title_status` indicates that 'clean' and 'lien' vehicles are more expensive. I think that the 'lien' vehicles are more expensive because they are usually in `condition` 'new' or 'like new'. Almost all the cars in the dataset are under 'clean'condition.

- `transmission` has the vast majority of vehicles under 'automatic' group. The definition of 'other' is not clear: however it is a small group and it is made mainly by expensive cars with automatic transmission.

- `drive` fwd is the cheapest group. This is because it's usually the drive mounted on sedan or small cars. More expensive cars in the dataset, trucks and sport cars, come with 4wd and rwd, respectively.

- `type` suggests that big trucks and pickups are the most expensive cars in the dataset.

- `paint_color` doesn't suggest a clear tendency. The more standard colors (white, black and 'unknown') seems slighly more expensive, because more requested.

## 3. Data Preparation
### Numerical Transformation and Encoding of Categorical Features
As observed in paragraph 2, `price` feature exhibits a relevant positive-skewed shape: a 'log' transformation is applied to improwed the skewness.

Regarding the categorical features `manufacturer`, `model`, `cylinder`, `fuel`, `title_status`, `transmission`, `drive`, `type`, `paint_color`: a numerical encoding (TargetEncoder in function of `price`) is applied.
Regarding `Condition` an ordinal encoding is applied, from 1 ('savage' condition) to 6 ('new' condition).

### Correlation Analysis
The dataset is now fully numerical. This means that relations between variables and the output feature can be investigated by means of a correlation matrix.

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/ecaf51eb-dca8-42ce-bd81-b9da9d92583f)

The correlation matrix (here visualized by means of a heatmap) highlights some relevant correlations and confirm some initial hypothesis.

Regarding the selling `price`:
- The strongest correlations are with `year`, `odometer`, `model` and `type`. There are good correlations even with technical features of the engine (`cylinders`, `transmission`, `drive`). The remaining features are not so correlated and confirm some of the preliminary hypothesis: `condition` (probably because almost all the items are in a good status), `fuel` (almost all vehicles have 'gas'), `title_status` (almost all vehicles are 'clean') and `paint color`.
- Other relevant correlations are between `year`-`odometer` (this definetly makes sense), `model`-`type`, `manufacturer`-`model`, or technical with the type of car (e.g. `cylinders`-`type`, `cylinders`-`drive`, `drive`-`type`...).

## 4. Modeling
The following regressive models are evaluated:
- Linear Regression
- LASSO
- Ridge

### Linear Regression Model
I use MSE and R^2 as parameters to evaluate the quality of the model. Mean Squared Error parameter is a quite common metric to evaluate the predictions of the model, since it strongly penalize any outliners. R^2 measures how much of the variation of a dependent variable `price` is explained in the regression model.

MSE of Linear Regression model is 0.087
R^2 of Linear Regression model is 0.797

### Linear Regression Model with Polynomial Features
The initial Linear Regression model is improved by adding a polynomial transformer. A Simple Cross Validation is implemented to find the polynomial degree of the best linear regression model. A function iterates through complexities from 1-5, builds a model with a degree of that complexity, and returns the model with the lowest test mean squared error.

The linear regression model with degree 3 provides the lowest MSE of 0.072, with a R^2 score of 0.83.

### LASSO Regression Model
After Linear Regression, I evaluate another regression, the LASSO model.

The LASSO model is optimized by means of GridSearchCV. The metric adopted is the minimum MSE (neg_mean_squared_error), with a 3 fold cross-validation. The hyperparameter to be optimized is the alpha coefficient of LASSO regressor.

The LASSO regression model with alpha = 0.0001 has the lowest MSE of 0.082, with a R^2 score of 0.81.

### Ridge Regression Model
The Ridge model is optimized by means of GridSearchCV. The metric adopted is the minimum MSE (neg_mean_squared_error), with a 3 fold cross-validation. The hyperparameter to be optimized is the alpha coefficient of the Ridge regressor.

The Ridge regression model with alpha = 0.0001 has the lowest MSE of 0.075, with a R^2 score of 0.833.

Among the regression models investigated, the Ridge model provides the best overall performance. I try to plot an histogram with the difference between validation test data and the values predicted by the Ridge regression model. The histogram is expected to have a gaussian distribution.

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/1275b69e-5443-4075-a968-ed2bbac59421)

The distributon of validation test data and the values predicted by the Ridge regression model is also visualized by means of a scatter plot.

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/b021533b-c591-44f1-b38c-64af10bc8977)

### Ridge Regression Model with Revisited Encoding (One-Hot)
The previous models have been developed from a numerical encoding. I'd be interested to compare the results obtained by the previous models, with a model developed from a discrete encoding, One-Hot. Plus, I believe One-Hot encoding could provide additional information, in terms of quantifying the importance of specific labels, per each feature.

The regression model comes without any polynomial transformation, to extract coefficients.

The Ridge regression model with One-Hot encoding has MSE of 0.182, with a R^2 score of 0.841.

## 5. Evaluation
### Permutation Importance of Ridge Model (Numerical Encoding)
year    0.187 +/- 0.003
odometer0.078 +/- 0.002
model   0.059 +/- 0.001
cylinders0.038 +/- 0.001
drive   0.017 +/- 0.001
type    0.013 +/- 0.001
fuel    0.009 +/- 0.000
condition0.007 +/- 0.000
manufacturer0.006 +/- 0.000
title_status0.004 +/- 0.000
transmission0.003 +/- 0.000
paint_color0.000 +/- 0.000

### Permutation Importance and Regression Coefficients of Ridge Model (One-Hot Encoding)
year    0.246 +/- 0.003
model   0.157 +/- 0.002
odometer0.083 +/- 0.001
manufacturer0.028 +/- 0.001
type    0.008 +/- 0.000
fuel    0.008 +/- 0.000
cylinders0.005 +/- 0.000
title_status0.004 +/- 0.000
drive   0.003 +/- 0.000
condition0.001 +/- 0.000
transmission0.000 +/- 0.000

10 Highest Regression Coefficients:
![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/ed230b0f-647f-4a8b-aec7-e0b4a973803e)

10 Lowest Regression Coefficients:
![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/f0b01ed9-23cd-40da-b75a-cd5e9df7e6f3)

### Observations
The permutation importance of the features from the two models presented states that the 3 most relevant features are `year`, `model` and `odometer`.

The analysis of the coefficients from the second ridge model (One-Hot encoding) gives interesting information about the most relevant values. The order of the coefficients agrees with the insights from previous paragraphs.

##### 5 Highest Coefficients (driving price high):
- year
- manufacturer_lexus
- model_corvette
- fuel_diesel
- model_silverado 1500

##### 5 Lowest Coefficients (driving price low):
- odometer
- manufacturer_lincoln
- model_focus
- model_focus
- model_cruze

The coefficients confirm the importance of the features from permutation importance.

## 6. Deployment
The quality of the provided dataset was very low: several missing values and a relevant number of duplicates. This could have influenced the accuracy of the final outcome.

The most impostant features driving the price of a used car are `year`, `odometer` and `model`.

`year`: the analysis focuses on the period 1998-2002. The majority of the cars in the dataset are from 2008 to 2015. The price tends to decay esponentially with the age of the car. According to the esponential trend, I would recommend the seller to not keep in his parking lot many cars built before 2008. Cars built after 2015 are the ones with the higher value and potentially with higher reselling margins. As in the provided dataset, all the cars in the parking lot they have to be in clean conditions.

`odometer`: cars with a significant mileage lose value. A more rigorous analysis should be carried out, but I would consider cars below 65k miles (first quartile of odometer distribution) "low-mileage cars", and car over 140k miles (third quartile of odometer distribution) "high-mileage" cars.

`model`: the specific car model hugely impacts the final price. Even the manufacturer could be considered, these two features are correlated. There are some high selling vehicles (e.g. Chevrolet Silverado 1500 or Ford F-150): this is related to the technical value of these vehicles, but I believe with the customers' perception as well. Some models are particularly popular and tend to keep their value higher. I would recommend the seller to keep more of the popular models, taking advantage of the provided "Top 20 Models" chart. Said list can be useful in two ways: top value models (e.g. GMC Sierra 155, Toyota Tacoma and Chevrolet Silverado 1500) are highly requested and can be sold at high prices. However, cheaper models cannot be neglected: e.g. Corolla, Prius and Civic are much cheaper but still they have a large number of loyal customers.

The developed regression model could be also a useful tool to quantify numerically some "price-groups" for the cars to be sold.

### Recommended Next Steps
- Improving the quality of the dataset to get more details (less NaN, duplicated items, outliners). This would also provide more available rows, improving the training of the regression models.
- Improving the accuracy of some values in the dataset: many 'other' labels or wrong parameters assigned.
- Better characterization of a price-penalizing coefficient for the odometer.
- Developing a more rigorous market analysis regarding the most requested models.
- Considering more advanced regression models.

![image](https://github.com/emmetizeta/What-drives-the-price-of-a-car/assets/101027884/77a84bcd-a633-4693-9ee3-4771b5aed7d2)



















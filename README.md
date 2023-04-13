# Life-Expectancy-project

#### All of the data in this dataset is compiled and downloaded from the Global Health Observatory (GHO).
#### Life Expectancy from birth was collected from: https://www.who.int/data/gho/data/indicators/indicator-details/GHO/life-expectancy-at-birth-(years)
#### The rest of the factors was collected from: https://www.who.int/data/gho/data/themes/mortality-and-global-health-estimates/ghe-leading-causes-of-death (BY COUNTRY, Summary tables of mortality estimates by cause, age and sex, by country, 2000–2019, Number of Deaths [2000, 2010, 2015, 2019]).
#### All of the values are crude estimates number of deaths per 1000.

### Transforming the data

#### To make the study easier we'll first filter the data by Spain, Ireland, Ethiopia, Thailand, Haiti.

```python
data_filt=data[data["Country"].isin(["Spain","Ireland","Ethiopia","Thailand","Haiti"])]
```

#### For the same reason we will filter the data by some diseases we find interesting to study.

```python
data_filt2=data_filt.filter(items=["Country","Year","Gender","Life Expectancy at birth","BMI","Alcohol","Liver cancer","Trachea, bronchus, lung cancers","Malignant skin melanoma"])
```

### Cleaning the data

```python
data_filt2 = data_filt2.rename(columns={'Country':'country',
                                        'Alcohol':'alcohol',
                                        'Year':'year',
                                        'Gender':'gender',
                                        'Life Expectancy at birth': 'life_expectancy_at_birth',
                                        'Liver cancer':'liver_cancer',
                                        'Trachea, bronchus, lung cancers':'trachea,_bronchus,_lung_cancers',
                                        'Malignant skin melanoma':'malignant_skin_melanoma'})
```

## Average number of liver cancer cases by country and sex?

### Transforming the data

```python
liver_cancer=data_filt2.pivot_table("liver_cancer", index="country", columns="gender", aggfunc="mean")
```
![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/Capture.JPG)

### Visualizing the data

#### The country with the most cases of death by liver cancer is Thailand followed by Spain.

```python
liver_cancer.plot(kind="barh")
```
![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/Capture2.JPG)

## Country with the highest life expectancy?

### Visualizing the data

```python
life_expectancy.plot(kind="barh")
```

#### The country with higher life expectancy is Spain followed by Ireland.

![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/Capture3.JPG)

## Ireland data study

### Transforming the data

```python
data_Ireland=data_filt2[data_filt2["country"].isin(["Ireland"])]
Ireland_data=data_Ireland.pivot_table(values=['alcohol',
                                            'liver_cancer','trachea,_bronchus,_lung_cancers',
                                            'malignant_skin_melanoma'], index="country", columns="gender", aggfunc="mean")
Ireland_data
```
![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/Capture4.JPG)

```python
Ireland_data= Ireland_data.unstack(-1)
Ireland_diseases = Ireland_data.reset_index()
Ireland_diseases.head(10)
```

### Cleaning the data

```python
Ireland_diseases = Ireland_diseases.rename(columns={'level_0':'diseases',0:'value'})
Ireland_diseases.head(10)
```
![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/Capture5.JPG)

### Transforming the data

```python
def norm_total(group):
    group["normed_value"] = group["value"]/group["value"].sum()
    return group
Ireland_diseases= Ireland_diseases.groupby("diseases").apply(norm_total)
```

### Visualizing the data

#### It seems to be alcoholism the main reasen of death followed by liver cancer.

```python
import seaborn as sns
Ireland_diseases_plot=sns.barplot(x='normed_value',y="diseases",hue="gender", data= Ireland_diseases).set(title='Ireland Diseases')
Ireland_diseases_plot
```

![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/Capture6.JPG)

### Comparing data studies

```python
Ireland_data=data_Ireland.pivot_table(values=['alcohol','life_expectancy_at_birth',
                                            'liver_cancer','trachea,_bronchus,_lung_cancers',
                                            'malignant_skin_melanoma'], index="country", columns="gender", aggfunc="mean")
Ireland_data= Ireland_data.unstack(-1)
Ireland_diseases = Ireland_data.reset_index()
Ireland_diseases = Ireland_diseases.rename(columns={'level_0':'diseases',0:'value'})
Ireland_diseases= Ireland_diseases.groupby("diseases").apply(norm_total)

Spain_data=data_Spain.pivot_table(values=['alcohol','life_expectancy_at_birth',
                                            'liver_cancer','trachea,_bronchus,_lung_cancers',
                                            'malignant_skin_melanoma'], index="country", columns="gender", aggfunc="mean")
Spain_data= Spain_data.unstack(-1)
Spain_diseases = Spain_data.reset_index()
Spain_diseases = Spain_diseases.rename(columns={'level_0':'diseases',0:'value'})
Spain_diseases= Spain_diseases.groupby("diseases").apply(norm_total)

Thailand_data=data_Thailand.pivot_table(values=['alcohol','life_expectancy_at_birth',
                                            'liver_cancer','trachea,_bronchus,_lung_cancers',
                                            'malignant_skin_melanoma'], index="country", columns="gender", aggfunc="mean")
Thailand_data= Thailand_data.unstack(-1)
Thailand_diseases = Thailand_data.reset_index()
Thailand_diseases = Thailand_diseases.rename(columns={'level_0':'diseases',0:'value'})
Thailand_diseases= Thailand_diseases.groupby("diseases").apply(norm_total)
```

### Visualizing the data

```python
import matplotlib.pyplot as plt

# Create a figure with two subplots
fig, axes = plt.subplots(nrows=1, ncols=3, figsize=(20, 30))

# Plot the normalized values for Ireland by gender and disease
Ireland_diseases.pivot_table(index="diseases", columns="gender", values="normed_value").plot(kind="bar", ax=axes[0], rot=20, title="Ireland")

# Plot the normalized values for Thailand by gender and disease
Thailand_diseases.pivot_table(index="diseases", columns="gender", values="normed_value").plot(kind="bar", ax=axes[1], rot=20, title="Thailand")

# Plot the normalized values for Spain by gender and disease
Spain_diseases.pivot_table(index="diseases", columns="gender", values="normed_value").plot(kind="bar", ax=axes[2], rot=20, title="Spain")

# Set common y-axis label for the subplots
fig.text(0, 0.5, "Normalized Value", va="center", rotation="vertical")

# Display the plot
plt.show()
```

![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/Capture7.JPG)

```python
import matplotlib.pyplot as plt

# Create a figure with multiple subplots
fig, axes = plt.subplots(nrows=5, ncols=2, figsize=(15, 20))

# Create a list of diseases to plot
diseases = ["alcohol", "life_expectancy_at_birth", "liver_cancer", "malignant_skin_melanoma", "trachea,_bronchus,_lung_cancers"]

# Plot the normalized values for Ireland by gender and disease
for i, disease in enumerate(diseases):
    Ireland_diseases.loc[Ireland_diseases["diseases"] == disease].pivot_table(index="gender", values="normed_value").plot(kind="bar", ax=axes[i, 0], rot=0, title=disease + " (Ireland)")

# Plot the normalized values for Thailand by gender and disease
for i, disease in enumerate(diseases):
    Thailand_diseases.loc[Thailand_diseases["diseases"] == disease].pivot_table(index="gender", values="normed_value").plot(kind="bar", ax=axes[i, 1], rot=0, title=disease + " (Thailand)")

# Set common y-axis label for all subplots
fig.text(0, 0.5, "Normalized Value", va="center", rotation="vertical")

# Adjust spacing between subplots
plt.subplots_adjust(hspace=0.5)

# Display the plot
plt.show()
```

![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/Capture8.JPG)

```python
import matplotlib.pyplot as plt

# Create a list of the three data frames
dfs = [Ireland_diseases, Thailand_diseases, Spain_diseases]

# Create a list of the countries
countries = ["Ireland", "Thailand", "Spain"]

# Create a list of colors for each country
colors = ["blue", "green", "red"]

# Create a list of the diseases
diseases = ["alcohol", "life_expectancy_at_birth", "liver_cancer", "malignant_skin_melanoma", "trachea,_bronchus,_lung_cancers"]

# Create a figure with subplots for each disease
fig, axs = plt.subplots(nrows=len(diseases), ncols=1, figsize=(10,20))

# Loop through each disease
for i, disease in enumerate(diseases):
    
    # Create a list of the normed values for each country by gender
    normed_values = []
    for df in dfs:
        normed_values.append(df[df["diseases"]==disease][["gender", "normed_value"]])
    
    # Plot the normed values for each country by gender
    ax = axs[i]
    ax.set_title(disease)
    for j, normed_value in enumerate(normed_values):
        ax.plot(normed_value["gender"], normed_value["normed_value"], label=countries[j], color=colors[j])
        ax.set_ylim([0,1])
    
    # Add a legend to the subplot
    ax.legend()

# Add a title to the figure
fig.suptitle("Normed values by gender and disease for Ireland, Thailand, and Spain")

# Show the figure
plt.show()
```

![image](https://github.com/EduardoJMR/Life-Expectancy-project/blob/master/images/output.png)






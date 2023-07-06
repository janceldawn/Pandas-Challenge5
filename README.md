# Pandas-Challenge5

#The following code sourced from Monash Lesson Plans
##code

# Combine the data into a single DataFrame
combined_data_df = pd.merge(study_results, mouse_metadata, how="left", on="Mouse ID")
combined_data_df.head()

# Checking the number of mice.
number_mice = len(combined_data_df["Mouse ID"].unique())
number_mice

# Optional: Get all the data for the duplicate mouse ID. 
duplicate_data_mouse = combined_data_df[combined_data_df["Mouse ID"].isin(duplicate_mouse)]
duplicate_data_mouse

# Create a clean DataFrame by dropping the duplicate mouse by its ID.
cleaned_data_mouse = combined_data_df[combined_data_df["Mouse ID"].isin(duplicate_mouse)==False]
cleaned_data_mouse.head()

##end code

#The following code sourced from Monas Lesson Plans, Stack Overflow, and also with guidance from AskBCs Ryan
##code
duplicate_mouse = combined_data_df.loc[combined_data_df.duplicated(), "Mouse ID"].unique()
duplicate_mouse

##end code

#The following code sourced from Monash Lesson Plans, Summary Statistics
##code

#mean
mean_drug = cleaned_data_mouse.groupby(["Drug Regimen"])["Tumor Volume (mm3)"].mean()
mean_drug

#median
median_drug = cleaned_data_mouse.groupby(["Drug Regimen"])["Tumor Volume (mm3)"].median()
median_drug

#variance
variance_drug = cleaned_data_mouse.groupby(["Drug Regimen"])["Tumor Volume (mm3)"].var()
variance_drug

#standard deviation
std_drug = cleaned_data_mouse.groupby(["Drug Regimen"])["Tumor Volume (mm3)"].std()
std_drug

#sem
sem_drug = cleaned_data_mouse.groupby(["Drug Regimen"])["Tumor Volume (mm3)"].sem()
sem_drug

# Assemble the resulting series into a single summary DataFrame.
summary_statistics = pd.DataFrame ({"Mean Tumor Volume": mean_drug,
                                   "Median Tumor Volume": median_drug,
                                   "Tumor Volume Variance": variance_drug,
                                   "Tumor Volume Std. Dev.": std_drug,
                                   "Tumor Volume Std. Err.": sem_drug})
summary_statistics

##end code

#The following code sourced from Pandas (https://pandas.pydata.org/docs/reference/api/pandas.DataFrame.agg.html)
##code

agg_sum_stat = cleaned_data_mouse.groupby("Drug Regimen")["Tumor Volume (mm3)"].agg({"mean", "median", "var", "std", "sem"})
agg_sum_stat

##end code

#The following code sourced from Monas Lesson Plans, Bar charts, Pybars, and checked with AskBCs Laanu
##code

cleaned_data_bar = cleaned_data_mouse["Drug Regimen"].value_counts()
cleaned_data_bar.plot(kind="bar")
plt.xlabel("Drug Regimen")
plt.xticks(rotation=90)
plt.ylabel("# of Observed Mouse Timepoints")
plt.show()


#pyplot
cleaned_data_bar = cleaned_data_mouse["Drug Regimen"].value_counts()
plt.bar(cleaned_data_bar.index.values,cleaned_data_bar.values)
plt.xlabel("Drug Regimen")
plt.xticks(rotation="vertical")
plt.ylabel("# of Observed Mouse Timepoints")
plt.tight_layout()
plt.show()

##end code

#The following code sourced from Monash Lesson Plans, Pie charts, Pypies
##code

#get sex values
gender_pie = cleaned_data_mouse.groupby("Sex").count()
gender_pie

pie_gender = gender_pie.plot(kind="pie", y="Mouse ID", autopct="%1.1f%%", startangle = 180)
pie_gender.set_ylabel("Sex")

y = np.array([922, 958])
mylabels = ["Female", "Male"]

#chart using pyplot
plt.pie(y, labels = mylabels, autopct="%1.1f%%", startangle = 180)
plt.show() 

##end code

#The following code sourced from Monash Lesson Plans, Quartiles, Outliers and guidance from AskBCs Roberto
##code

#final_tumor_vol 
max_tumor = cleaned_data_mouse.groupby(["Mouse ID"])["Timepoint"].max()
max_tumor = max_tumor.reset_index()

# Merge this group df with the original DataFrame to get the tumor volume at the last timepoint
merged_data = max_tumor.merge(cleaned_data_mouse, how="left", on=["Mouse ID", "Timepoint"])

# Put treatments into a list for for loop (and later for plot labels)
treatment_drugs = ["Capomulin", "Ramicane", "Infubinol", "Ceftamin"]

# Create empty list to fill with tumor vol data (for plotting)
tumor_vol_list = []

# Calculate the IQR and quantitatively determine if there are any potential outliers. 
for drug in treatment_drugs:
    
    # Locate the rows which contain mice on each drug and get the tumor volumes
    final_tumor_vol = merged_data.loc[merged_data["Drug Regimen"] == drug, "Tumor Volume (mm3)"]

    # add subset 
    tumor_vol_list.append(final_tumor_vol)
    
    # Determine outliers using upper and lower bounds
    quartiles = final_tumor_vol.quantile([.25,.5,.75])
    lowerq = quartiles[0.25]
    upperq = quartiles[0.75]
    iqr = upperq-lowerq
    
    lower_bound = lowerq - (1.5*iqr)
    upper_bound = upperq + (1.5*iqr)
    
    outliers = final_tumor_vol.loc[(final_tumor_vol < lower_bound) | (final_tumor_vol > upper_bound)]
    
    print(f"{drug}'s potential outliers: {outliers}")

##end code

#The following code sourced from Matplotlib documentation page (https://matplotlib.org/stable/gallery/statistics/boxplot_demo.html)
##code

fig1, ax1 = plt.subplots()
ax1.set_xlabel (treatment_drugs)
ax1.set_ylabel("Final Tumor Volume (mm3)")
ax1.boxplot(tumor_vol_list, flierprops = dict(marker = "o", markersize = 12.0, markerfacecolor = "red"))
plt.show()

##end code

#The following code sourced from Monash Lesson Plans, .loc, groupby, Correlation and Regression
##code

#filter by mouse id
mouse_id = "l509"
mouse_id_data = cleaned_data_mouse.loc[cleaned_data_mouse["Mouse ID"] == mouse_id]
mouse_id_data

#filter by drug
capomulin_data = mouse_id_data.loc[mouse_id_data["Drug Regimen"] == "Capomulin"]
capomulin_data

#create lineplot
plt.plot(capomulin_data["Timepoint"], capomulin_data["Tumor Volume (mm3)"], linestyle="-")
plt.xlabel("Timepoint (days)")
plt.ylabel("Tumor Volume (mm3)")
plt.title(f"Capomulin Treatment of Mouse {mouse_id}")
plt.show()

#filter capomulin
capomulin_data = cleaned_data_mouse.loc[cleaned_data_mouse["Drug Regimen"] == "Capomulin"]
capomulin_data

#get average tumor
average_tumor_volume = capomulin_data.groupby("Mouse ID")["Tumor Volume (mm3)"].mean()
average_tumor_volume

#get mouse weight
mouse_weight = capomulin_data.groupby("Mouse ID")["Weight (g)"].mean()
mouse_weight

#generate scatter plot
plt.scatter(mouse_weight, average_tumor_volume)
plt.xlabel("Weight (g)")
plt.ylabel("Average Tumor Volume (mm3)")
plt.show()

#calculate correlation
correlation = st.pearsonr(mouse_weight,average_tumor_volume)
print(f"The correlation between mouse weight and average tumor volume is {round(correlation[0],2)}")

# Add the linear regression equation and line to plot
x_values = mouse_weight
y_values = average_tumor_volume
(slope, intercept, rvalue, pvalue, stderr) = linregress(x_values, y_values)
regress_values = x_values * slope + intercept
line_eq = "y = " + str(round(slope,2)) + "x + " + str(round(intercept,2))
plt.scatter(x_values,y_values)
plt.plot(x_values,regress_values,"r-")
plt.annotate(line_eq,(5.8,0.8),fontsize=15,color="red")
plt.xlabel("Weight (g)")
plt.ylabel("Average Tumor Volume (mm3)")
plt.show()

##end code

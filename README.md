# Machine-Learning-Analysis-on-the-Myocardial-infarction-complications-Database
###GOAL
Build a highly-interpretable predictive model for a specific complication to help medical professionals that have available only the data of the patient’s entry at the hospital.
###dataset
The provided dataset has a multitude of medical measurements (columns) for a number of patients (rows) that where taken to hospital with a myocardial disease.  All patients were observed for three days, having measurements at four possible moments: entry to hospital, end of day 1, end of day 2 and end of day 3. The last columns code the appearance or not of a set of possible complications and the lethal outcome for each patient.

We are given a dataset with 124 columns and 1700 rows.
Columns 1 to 112 represent measurements, medical history and drug dosage for each patient
Columns 113 to 123 represent the binary result of whether a patient developed a specific complication by the end of the third day since they were taken to hospital.
Column 124 is the lethal outcome where the value zero implies the patient is alive at the end of the third day while non-zero values portray the cause of death of the respective patient

We choose to only work towards the prediction of one of the possible complications.

###Approaching missing values
After checking the dataset, we see that most columns have missing values, in percentages between 1% and 96%. This obviously makes the analysis more complicated, since we need to find an appropriate strategy to deal with the missingness.

Assumption: Our data is Missing Completely At Random (MCAR). This means that there is no correlation between the missingness of any value and the other values in each row or column. This assumption logically holds since all our feature columns (used for prediction) are medical history, test results or drug dosage, so the value of one should not have any affect on whether another will be missing.

Question: Can we replace the missing values by imputation?
	Answer:  We do not know (not enough information). There are 2 possibilities:
The data is missing due to human error or connection malfunction. So there was a value that was simply not recorded by mistake. In this case, it makes sense to try and impute.
The data is missing because it could not be taken. In this case, the missingness is information we should not lose by imputing (it is wrong to estimate).

As such, we will make a model under each of these missingness assumptions.
1. Replacing missing values:
   Question: How well do the imputed values follow the real densities of each column?

For less than 10% missing, we trust that replacing by the mode/median does not mess with the variance and thus does not mess the original density significantly.

Deleting a column avoids this problem.

For the MICE technic, see graphic.

Conclusion: Densities remain close to original


2. Retaining missing values:
   Going with the second scenario, we believe that the data is missing because it could not be taken. This could be a missing measurement because specific devices cannot give measurements under environmental conditions or a missing drug dosage because the patient is no longer alive. In such cases, the missingness of the data holds important information of its own.

For this reason, we will try to use some special model that can deal with classification while retaining missing data.

###Feature reduction/Model choice:
Problem:  We want to make an interpretable model but we have 100 features available as measurements from the time of entry alone
	Solution: Find an effective way to reduce the features used without sacrificing accuracy.

The best way to accomplice a classification task of this sort is some implementation of a Random Forest (multiple iterations of decision trees on the same data). This means that the computer trains on multiple trees to find the best way to split data and be able to find the most accurate prediction.
𝑎𝑐𝑐𝑢𝑟𝑎𝑐𝑦=(# 𝑜𝑓 𝑐𝑜𝑟𝑟𝑒𝑐𝑡 𝑝𝑟𝑒𝑑𝑖𝑐𝑡𝑖𝑜𝑛𝑠)/(𝑇𝑜𝑡𝑎𝑙 # 𝑜𝑓 𝑖𝑛𝑝𝑢𝑡 𝑠𝑎𝑚𝑝𝑙𝑒𝑠)

This comes with the added bonus of creating ‘importance values’ for all features, based on how many splits happen for each feature. This is a very natural feature selection tool, allowing as to achieve feature reduction as part of our model.

Once again, we have 2 scenarios of models, based on the missing values handling:
With imputed data
With missing values

###WITH imputed data:
We tried four different models:
Decision Tree Classifier
Random Forest Classifier
XGBoost (a special Random Forest algorithm that is Gradient Boosted)
HistGradientBoostingClassifier (a special Random Forest similar to XGBoost, utilizes histogram binning)

We compared the respective accuracies to choose the best one.

However, we first need to find the best subset of features to get the highest accuracy possible. 

Strategy:  After we get the importance values from the initial model with all features, we retrain a new model with the most important features only, adding more in each iterations. We then keep the subset that achieves the highest accuracy as optimal and compare their resulting accuracy with the other models. We lastly do a stratified test to get the average accuracy for the best model.

###WITH missing data:
We tried using the Histogram Gradient Boosting Classifier as a model, similar to the XGBoost but capable of working with missing values.

Once again, we first need to find the best subset of features to get the highest accuracy possible. 

In this case however, we extract the importance by a permutation method:  We mix the values of one column at a time and see the change of accuracy of the model. The more the accuracy drops, the highest the importance of that feature.

Strategy:  After we get the importance values from the initial model with all features, we retrain a new model with the most important features only.  We lastly do a stratified test to get the average accuracy.

###Interpretation:
Other features of importance:
If the ECG rhythm is sinus with heart rate above 90
Appearance of exertional angina pectoris in the anamnesis
Coronary heart disease (CHD) in recent weeks, days before admission to hospital
Presence of a lateral myocardial infarction (left ventricular)

###Conclusions:
Our analysis and results heavily rely on how we deal with missing data.
An accurate medical history is vital for prediction tasks.
In the case of imputed data, lab results seem to be more important predictors for this specific complication, compared to ECG measurements



















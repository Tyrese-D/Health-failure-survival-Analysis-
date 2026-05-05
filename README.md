# Health-failure-survival-Analysis-
An Exploratory Data Analysis (EDA) on clinical records to identify the primary predictors of survival in heart failure patients. By visualizing 13 distinct clinical features, the analysis provides a "Risk Zone" framework to help identify high-mortality segments based on serum biomarkers and patient history.

📊 The Dataset
The analysis is based on a clinical dataset of 299 patients with heart failure.

Clinical Markers: Ejection Fraction, Serum Creatinine, Serum Sodium, Platelets, Creatinine Phosphokinase.

Comorbidities: Anaemia, Diabetes, High Blood Pressure.

Demographics: Age, Sex, Smoking Status.

Outcome Variable: Death Event (Survival vs. Mortality).

Outcome Variable: Death Event (Survival vs. Mortality).
<img width="1920" height="877" alt="Screenshot 2026-04-27 at 11 57 46 AM" src="https://github.com/user-attachments/assets/46b25a9d-d9f7-44f8-89dc-8cc3db400594" />


🛠️ Tech Stack
Analysis: Tableau Desktop / Tableau Public.

Data Prep: Excel (Cleaning and categorical encoding).

Statistical Visualization: Box-and-Whisker Plots, Stacked Histograms, Scatter Plot Matrices.

Live interactive dashboard: https://public.tableau.com/app/profile/tyrese.dieudonne/viz/HeartFailurePatientSurvivalAnalysis/Dashboard1
<img width="1920" height="1080" alt="Screenshot 2026-05-05 at 4 35 15 AM" src="https://github.com/user-attachments/assets/01004c63-d889-44b2-9391-9ed7dae3565e" />



Machine Learning Integration: Predictive Modeling
While the Tableau dashboard provides a visual diagnostic tool, I implemented a Python-based machine learning pipeline to automate risk assessment.

  1. Models Evaluated: Logistic Regression, Random Forest, Support Vector Machine (SVM), and Decision Trees.
 
  2. Top Performer: The Random Forest Classifier achieved the highest accuracy (approx. 85-90%), effectively handling the non-linear relationships between clinical biomarkers.

 3. Feature Engineering: Utilized StandardScaler to normalize continuous variables like Platelets and Creatinine Phosphokinase, ensuring high-variance features didn't bias the model.
4. Evaluation: Leveraged Confusion Matrices and Accuracy scores to validate model reliability against the test dataset.


Evaluation: Leveraged Confusion Matrices and Accuracy scores to validate model reliability against the test dataset.



The Model vs. The Dashboard: I learned that while a model gives a "prediction," the Tableau dashboard provides the "explanation." Combining both allows a doctor to see a high-risk score and the specific biomarkers causing it.

Data Imbalance Challenges: In clinical datasets, "Death Events" are often fewer than "Survival Events." I learned how to evaluate models using more than just accuracy—focusing on Precision and Recall to ensure the model doesn't miss high-risk patients (False Negatives).

Algorithm Selection: I discovered that while Logistic Regression is great for interpretability, Ensemble methods like Random Forest are superior for capturing the complex, interconnected nature of human health data

Predictive Triage System
1. By deploying the Random Forest model into a clinical workflow, hospitals could implement an Automated Triage Alert. When a patient's lab results are entered, the system can instantly flag those with a high "Mortality Probability," allowing staff to prioritize care before symptoms escalate.

2. Consumer & Patient Trust
A major concern in "Black Box" AI is trust.

Recommendation: Use the Tableau visualizations as a "Translation Layer." When the ML model flags a patient, show the patient the Feature Importance (e.g., "Your Serum Creatinine is significantly above the median") to make the AI's decision transparent and actionable.


 Lessons Learned
The Hybrid Approach: I learned that while the Python model provides the "Risk Score," the Tableau dashboard provides the "Explanation." This combination is essential for medical buy-in.

Handling Clinical Imbalance: Real-world medical data is often imbalanced (fewer death events than survival). I learned to use Precision-Recall curves to validate the model's true performance.

Data Synthesis: Managing binary data (Smoker status) alongside continuous data (Platelet counts) required careful axis synchronization in Tableau to avoid misleading correlations.


🛑 Strategic Recommendations
1. Automated Clinical Triage
Implementation: Hospitals can use this Random Forest model as a "second set of eyes" to automatically flag patients with a mortality probability >60%.

Goal: Allow medical staff to prioritize high-risk specialist reviews before symptoms escalate.

2. Transparent AI (XAI)
Trust Factor: To address patient concern over "Black Box" algorithms, use the Tableau visuals to show patients why they were flagged.

Actionable Insights: Turning a "Risk Score" into a visual representation of their biomarkers makes the data a conversation starter for lifestyle changes rather than a scary number.

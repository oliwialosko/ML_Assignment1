# Capstone Report — Machine Learning

# Prioritizing SEO Content Updates Using XGBoost

- **Author:** Oliwia Łośko
- **Lane:** Machine Learning
- **Repo:** https://github.com/oliwialosko/ML_Assignment1
- **Date:** 28.08.2026


## 0. Abstract

**This study investigates whether machine learning can predict short-term traffic decline for high-value web pages before a substantial loss of visibility occurs. A four-month FlyRankAI dataset containing daily search performance metrics and content metadata was analyzed to identify patterns associated with traffic declines. Rolling features describing historical performance and user behavior were engineered, and an XGBoost classifier was trained and optimized using Optuna. The model was evaluated using a strict out-of-distribution validation strategy. The final model achieved a Precision@50 of 0.600, outperforming the static baseline, which achieved 0.540. The results were used to create a prioritized action list, providing a decision-support tool for identifying pages that may require attention before a significant loss of traffic occurs.**

## 1. Problem framing

Imagine an evergreen piece of content that consistently generates valuable organic traffic for a brand. At some point, its click-through rate (CTR) starts to decline slightly, while its average position in search results drops from 4th to 6th. Does this indicate nothing more than normal month-to-month fluctuation, or could it be an early signal of a more significant decline in traffic? This is a common question faced by editorial teams managing large numbers of content pages. Identifying pages that are genuinely at risk of losing visibility is a major challenge. This is particularly important for high-value content, where a delayed response can result in a significant loss of traffic. On the other hand, updating every page that shows a temporary decline is inefficient and leads to unnecessary editorial effort.
This paper presents a machine learning approach aimed at addressing this prioritization problem. The unit of analysis is an individual content page, and the objective is to identify pages at risk of experiencing a significant decline in traffic, defined in this study as a drop of more than 30% over the following 15 days. For each page, the model produces a risk score that allows pages to be ranked according to their predicted probability of experiencing such a decline. This ranking enables editors to focus their attention on pages that are most likely to require intervention, for example by updating outdated information, improving meta descriptions, or expanding the content where necessary.
Machine learning is particularly effective for this task because traffic decline can be influenced by multiple factors that interact in complex, non-linear ways. Traditional SEO processes often rely on editorial judgment or fixed heuristic rules, which can be difficult to apply consistently across a large and diverse set of pages. A machine learning model can instead learn patterns from historical data and combine information about content freshness, user engagement, and search performance to estimate the likelihood of a future traffic decline. The temporal nature of the data also makes it possible to use historical performance to identify patterns preceding such declines, which is a common principle in predictive analyses involving time-dependent data. The proposed approach therefore aims to shift content maintenance from a largely reactive process toward a more systematic and data-driven prioritization of editorial effort.


## 2. Data safety

The analysis is based on a dataset provided by FlyRankAI as part of an international Machine Learning (ML) internship program. The dataset contains more than 73 million rows in total and includes information from both fact and dimension tables. To obtain a sufficiently large training set while keeping the computational requirements manageable, the data were aggregated iteratively over a four-month period, from January to April 2026, rather than using a single-month snapshot. The 15th day of each month was used as the reference point, with a 15-day rolling window applied to capture short-term changes in page performance. The limitations of this approach are discussed in Section 5. Pages with fewer than 100 historical impressions were excluded from the analysis, as they generally provide less reliable performance estimates due to the higher level of noise in low-volume data and are also less likely to be a priority for enterprise editorial teams.
Particular attention was paid to preventing data leakage. The target variable (is_declining_label) was determined exclusively from the future observation window, covering performance after the 15th day of the month. In contrast, all model features were calculated using information available on or before the reference date. This ensured that future information was not included in the feature set.
Pseudonymized identifiers, such as client_hash_id and content_hash_id, were used only to group observations when splitting the data into training and test sets. They were excluded from the model's feature matrix to prevent the model from learning patterns specific to individual pages or clients. Including these identifiers could allow the model to memorize characteristics of specific pages or clients rather than learn patterns that generalize to new ones. No client-identifying information, private URLs, or raw search queries were included in the analysis or its outputs.


## 3. Baseline

Before training the machine learning model, a simple rule was established as a baseline. This provided a reference point for evaluating the model's performance while keeping the comparison transparent and easy to interpret.
The baseline classified a page as "at risk" if it met three conditions: it had more than 500 historical impressions, its average position in search results was between 7 and 10, and its click-through rate (CTR) was below 1.0%. Pages meeting these criteria were then ranked by CTR, with the highest priority given to those with the lowest value.
To ensure a fair comparison, the baseline was evaluated on the same out-of-distribution (OOD) test set used to evaluate the machine learning model. This set contained data from April 2026 and included a group of clients that had not been previously included in the analysis. The base rate for this test set, representing the probability of randomly selecting a page exhibiting a declining trend, was 0.404. The baseline rule achieved a Precision@50 of 0.540. This provides an important reference point for assessing whether the proposed model can identify pages with a declining trend more effectively than a simple rule-based approach.


## 4. Model / analysis

The main model used in the proposed approach was an XGBoost classifier. This model was selected because tree-based ensemble methods perform well on tabular data and can capture complex, non-linear relationships between features. To improve model performance, Optuna was used for hyperparameter optimization. The tuning process included parameters such as the learning rate, tree depth, and regularization weights, with the objective of maximizing the selected evaluation metric, Precision@50.
The feature engineering process was based on both domain knowledge and findings from the FlyRank report State of AI-Driven SEO. The final model included 13 features: *imp_past, ctr_past, avg_pos_past, sessions_past, engagement_rate, scroll_rate, word_count, days_since_update, days_visible_past, content_age, main_intent, competition, and backlinks*. Several additional features were also tested but deliberately excluded from the final model. *age_bucket* was removed after initial experiments showed that it did not provide additional information. *search_volume* and *click_value* were also excluded because including them introduced noise and reduced model performance. This result is consistent with the findings presented in the aforementioned FlyRank report.
The target variable, *is_declining_label*, was defined as a binary indicator of traffic decline. Several thresholds were tested to determine what should constitute a significant decline. A decrease of more than 30% in future impressions compared with the preceding 15-day period produced the most stable and practically useful results and was therefore selected as the final threshold.


## 5. Evaluation

The model was evaluated using an out-of-distribution (OOD) validation strategy based on GroupShuffleSplit. The training data included observations from January to March for 75% of the clients, while evaluation was performed on April data from the remaining 25% of clients, which were not present in the training set. This approach was used to assess whether the model could generalize to previously unseen clients rather than rely on patterns specific to individual clients.
*Precision@50* was selected as the evaluation metric because it reflects the practical use case in which an editorial team has limited time to review pages with the highest priority. A comparison of the results obtained using different methods is shown in Figure 1. On the test set, the base rate, corresponding to the probability of randomly selecting a declining page, was 0.404. The baseline rule defined in Section 3 achieved a *Precision@50* of 0.540, while the final XGBoost model, after hyperparameter optimization, achieved a score of 0.600.

![A comparison of the results obtained using different methods][fig1_baseline_comparison.png]
*Figure 1. A comparison of the results obtained using different methods.*

The final result was achieved through several iterations of feature engineering and model development. An initial model based on a smaller feature set achieved a *Precision@50* of 0.260. Expanding the feature set increased the score to 0.420, representing a substantial improvement over the initial approach, although the result remained below the domain-informed baseline rule. Hyperparameter optimization using Optuna led to a further improvement, increasing *Precision@50* to 0.600.
The use of a 15-day rolling window was a deliberate compromise resulting from the computational constraints of the experimental environment. As a result, the model was designed primarily to identify short-term changes in traffic rather than long-term seasonal patterns. This represents an important limitation, as the model does not explicitly account for broader seasonal effects, such as changes in traffic following holidays. The approach also assumes that the relationship between the selected features and traffic decline remains reasonably stable between the training and evaluation periods.
A longer historical period and additional features capturing long-term trends, such as 90-day performance patterns, could potentially improve the model's ability to account for seasonality. However, incorporating such information would primarily require greater computational resources.


## 6. Interpretation

The behavior of the XGBoost model can be interpreted from two perspectives: the relative importance of individual features and the direction in which their values influence the model's predictions. The feature importance presented in Figure 2 shows that *days_since_update* and *ctr_past* contributed the highest gain to the model. This indicates that these features were particularly useful when constructing the decision trees. However, this analysis alone does not show how higher or lower values of a given variable affect the predicted risk. This relationship can be examined using the SHAP summary plot presented in Figure 3.

![Feature Importance PLot][fig2_feature_importance.png]
*Figure 2. Feature Importance Plot.*

The SHAP results show that the relationship between some features and the predicted probability of decline is not straightforward or linear. For example, the effect of *content_age* varies considerably across observations. Pages with very low content age tend to be associated with lower predicted risk of short-term decline, while high content age also appears to be associated with relatively low risk, as indicated by negative SHAP values. One possible explanation is that older content may have already been updated or may represent evergreen material that remains relevant over time, meaning that content age alone does not necessarily indicate a risk to its search visibility.
Similarly, higher values of *word_count* generally shift the model's predictions towards a lower probability of decline, suggesting that more comprehensive content may be associated with greater short-term stability. Categorical variables such as *competition* and *main_intent* are shown in grey in the SHAP visualization because their values do not correspond to the continuous numerical scale used for color encoding.

![SHAP][fig3_shap_summary.png]
*Figure 3. SHAP Summary Plot.*

Several of the observed relationships can also be considered in the context of search performance. Higher values of *imp_past* generally increase the predicted probability of decline. One possible explanation is that pages with high historical visibility have more traffic to lose, making a substantial percentage decline more likely to occur than for pages with very low initial traffic. Similarly, higher values of *avg_pos_past*, corresponding to worse average positions in search results, are generally associated with an increased risk of future decline.
The model also identified different relationships for the two user behavior metrics. Higher values of *engagement_rate* generally reduce the predicted risk, whereas higher values of *scroll_rate* tend to increase it. This may be related to the fact that extensive scrolling does not necessarily indicate positive user engagement. In some cases, users may scroll through a page without finding the information they are looking for.


## 7. Recommendation

The primary output of the analysis is a prioritized list of pages for which a significant decline in traffic has been identified as a potential risk. Pages are ranked primarily according to the predicted risk probability generated by the proposed XGBoost model. Historical click value is used as an additional sorting criterion, allowing higher priority to be given to pages generating more valuable traffic.
The risk scores generated by the model are combined with predefined SEO rules to assign specific, practical recommendations to individual pages. For example, a page classified as high risk and present on the website for more than six months may be assigned a *"Refresh Mature Content"* recommendation, indicating that its content should be reviewed and updated if necessary. Pages ranking within the top 20 search results but having a low click-through rate (CTR) may be assigned an *"Urgent Snippet Rewrite"* task, prompting a review of their titles or meta descriptions. Similarly, pages with high historical visibility, defined here as more than 1,000 impressions, may receive a *"Boost High-Volume Edge"* recommendation, suggesting that adding new or more comprehensive content could help maintain or improve their visibility.
The resulting prioritization provides a structured workflow for directing editorial attention and resources towards pages that are most likely to require intervention. The system is intended as a decision-support tool rather than an automated content modification process. Human review remains necessary before any changes are introduced. Editors should verify whether a predicted decline actually requires intervention and whether it may be explained by factors outside the scope of the model, such as expected seasonal changes or technical events. In this way, the model supports editorial decision-making while reducing the number of unnecessary content updates.


## 8. Reproducibility

The project architecture and methodology are maintained in the associated GitHub repository to support reproducibility. The required environment can be recreated from the file requirements.txt. The main analysis, including the GroupShuffleSplit procedure used to create the out-of-distribution validation set, is documented in the capstone.ipynb notebook. Model training and Optuna hyperparameter optimization use a fixed random seed of 42 to ensure reproducible results. Running the notebook from start to finish processes the data, evaluates the model on the holdout set, and generates the final recommendations.

## 9. Acknowledgments & data credit

Built on the FlyRank ML Internship dataset (https://flyrank.ai).

---

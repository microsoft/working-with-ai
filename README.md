# "Working with AI" results files
These files contain results from the paper:

> Kiran Tomlinson, Sonia Jaffe, Will Wang, Scott Counts, and Siddharth Suri. Working with AI: Measuring the Applicability of Generative AI to Occupations. arXiv, 2025. https://arxiv.org/abs/2507.07935

 In particular, they include our metrics aggregated at the O*NET intermediate work activity (IWA) and occupation (detailed 2018 SOC code) level. These describe how often Bing Copilot was used for different work activities, how successfully, and what this usage indicates about which occupations AI may be applicable to. The goal of these metrics is to measure how AI may be useful in different occupations. Refer to the paper for a detailed discussion of the metrics and results.

## File descriptions
- `iwa_metrics.csv`: IWA-level metrics, reported separately for user goals and AI actions. 
- `soc_ai_applicability_scores.csv`: AI applicability scores for each SOC code (average of user goal and AI action scores).
- `soc_metrics.csv`: IWA-level metrics aggregated to detailed SOC codes according to the SOC-IWA weights, reported separately for user goals and AI actions.
- `soc_iwa_weights.csv`: IWA weights for each SOC code based on O*NET relevance and importance.
- `soc_to_iwas.csv`: Mapping of SOC codes to associated IWAs.

In brief, the IWA-level metrics are:
- `share_{user,ai}`: the fraction of conversations mapping to each IWA (as a user goal or AI action, reported separately). If a conversation matches multiple IWAs, it is divided equally among them so that the sum of activity shares across IWAs is 1. 
- `completion_{user,ai}`: the fraction of conversations mapping to an IWA (as a user goal or AI action) where our completion classfiier indicates that the LLM completed the user’s goal. 
- `impact_scope_{user,ai}`: the fraction of conversations mapping to an IWA (as a user goal or AI action) where the classified scope of impact is moderate or higher.
- `feedback_positive_fraction_{user,ai}`: of all thumbs reactions in Copilot-Thumbs on conversations mapping to an IWA (as a user goal or AI action), the fraction where the reaction was a thumbs up.
- `completion_x_scope_x_coverage_{user,ai}`: for convenience, the product of completion, impact scope, and the indicator `1[share > 0.0005]`. The weighted sum of these columns using the weights in `soc_iwa_weights` yields the user- and AI-side applicability scores, which are averaged together to compute the final AI applicability scores reported in `soc_ai_applicability_scores.csv`.

The SOC-code-level metrics are:
- `completion_{user,ai}`, `impact_scope_{user,ai}`, `feedback_positive_fraction_{user,ai}`: same as above, aggregated to SOC occupations according to IWA weights
- `coverage_{user,ai}`: the importance- and relevance-weighted fraction of IWAs in an occupation with activity share at least 0.05% (i.e., share > 0.0005)
- `ai_applicability_score_{user,ai}`: AI applicability scores computed only from the user goal and AI action metrics, before being averaged as in `soc_ai_applicability_scores.csv`

### Missing data
O\*NET does not include task data for all occupations. We omit all military occupations (SOC codes 55-xxxx), as they have no task data in O\*NET. We also omit 74 SOC codes mapping to O*NET occupations for which there is no task data.
This leaves us with 785 SOC codes covering 149.8 million workers according to the 2024 BLS Occupational Employment and Wage Statistics (OEWS) data (total US employment in 2024 OEWS data is 154.2 million).

Additionally, some tasks in O\*NET are missing rating data. See the paper's appendix for details about how we handled missing task rating data in computing IWA weights.


## Computing AI applicability score
The following code implements Equation (1) from the paper, averaging user and AI applicability scores. For convenience, AI applicability score is precomputed in `soc_ai_applicability_scores.csv`.

```python
import pandas as pd

iwa_df = pd.read_csv('iwa_metrics.csv').set_index('IWA')
soc_to_iwas = pd.read_csv('soc_to_iwas.csv').groupby('SOC Code')['IWA'].apply(list).to_dict()
iwa_weights = pd.read_csv('soc_iwa_weights.csv').set_index('SOC Code').T.to_dict()

ai_applicability_score = {soc: 0 for soc in soc_to_iwas}
coverage_threshold = 0.0005

for soc, iwas in soc_to_iwas.items():
    total_weight = sum(iwa_weights[soc][iwa] for iwa in iwas)
    for iwa in iwas:
        # average of user goal score and AI action score
        if iwa_df.loc[iwa, 'share_user'] > coverage_threshold:
            ai_applicability_score[soc] += 0.5 * (iwa_weights[soc][iwa] / total_weight) * iwa_df.loc[iwa, 'completion_user'] * iwa_df.loc[iwa, 'impact_scope_user']
        if iwa_df.loc[iwa, 'share_ai'] > coverage_threshold:
            ai_applicability_score[soc] += 0.5 * (iwa_weights[soc][iwa] / total_weight) * iwa_df.loc[iwa, 'completion_ai'] * iwa_df.loc[iwa, 'impact_scope_ai']
```


## Additional public data
These results use the O*NET 29.0 Database, which can be downloaded here: https://www.onetcenter.org/dl_files/database/db_29_0_text.zip.

See https://www.bls.gov/oes/tables.htm for Occupational Employment and Wage Statistics data linked by SOC Code.

Use https://www.bls.gov/emp/classifications-crosswalks/nem-onet-to-soc-crosswalk.xlsx to convert O\*NET-SOC Codes used to identify jobs in O\*NET to SOC Codes. BLS also provides crosswalks for linking SOC codes to other occupational taxonomies, such as the codes used by the Current Population Survey.


## License
This repository is licensed under CC BY 4.0. To view a copy of this license, visit https://creativecommons.org/licenses/by/4.0/. To attribute this repository, use the citation below.


## Citation
```
@misc{tomlinson2025working,
      title={Working with AI: Measuring the Applicability of Generative AI to Occupations}, 
      author={Kiran Tomlinson and Sonia Jaffe and Will Wang and Scott Counts and Siddharth Suri},
      year={2025},
      eprint={2507.07935},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2507.07935}, 
}
```


## Transparency 
### Intended uses 
These results are being shared with the research community to facilitate reproduction of our analyses and foster further research in this area. As we use the O*NET work activity taxonomy and the 2018 Standard Occupational Classification, our results can be linked to other public data sources for further analysis. 

The conversations contributing to these results are from Bing Copilot in the United States from January 1, 2024 to September 30, 2024, so may not represent AI usage in other countries.  

### Out-of-scope uses
Our metrics should not be misconstrued or misrepresented as measuring the ability of AI to replace jobs. There is more to an occupation than the sum of its O*NET tasks, and our measures of success (completion, scope, and thumbs feedback) are proxies that do not capture full workplace capabilities. Rather, our metrics indicate (1) what a consumer LLM is used for, (2) in a relative sense, which of these uses are more or less successful, and (3) which occupations may find AI more useful or applicable as a result.

### Data creation and processing 
These results were computed from Bing Copilot conversation data, which was originally collected by Microsoft and consists of conversations between users and Bing Copilot. 

The classification of conversations into IWAs was performed by an LLM pipeline designed by Kiran Tomlinson, Siddharth Suri, and Scott Counts. The pipeline was programmed and run by Kiran Tomlinson. The analysis was performed by Kiran Tomlinson, Sonia Jaffe, Will Wang, Scott Counts, and Siddharth Suri.  

### People and identifiers 
This repository does not contain any information that could be used to directly or indirectly identify a person. All metrics in this repository are aggregated at the activity and occupation level and do not reveal any information about particular individuals. 

Before we classified the Bing Copilot conversations, potentially identifiable information was automatically scrubbed (e.g., social security numbers, credit card numbers, zip codes).  

### Validation 
A detailed discussion of our classifier validation can be found in our paper.

### Limitations
The Bing Copilot conversations contributing to these results are from the United States during the period from 1/1/2024-9/30/2024. The majority of conversations were in English, but we did not filter by language.  

LLM-based classifiers are imperfect, so all metrics should be interpreted with caution. See the paper for the exact prompts we used and for interrater reliability with human annotators. As these results are based on consumer AI usage, they do not represent the full distribution of AI use in the workplace. For some work tasks, AI tools are directly integrated into specialized software, or specific models are used for their dedicated capability or compliance with data security requirements. For instance, programmers often use AI-enhanced IDEs and legal, medical, and financial work requires compliant AI tools.  

### Trademarks
This project may contain trademarks or logos for projects, products, or services. Authorized use of Microsoft trademarks or logos is subject to and must follow Microsoft's Trademark & Brand Guidelines. Use of Microsoft trademarks or logos in modified versions of this project must not cause confusion or imply Microsoft sponsorship. Any use of third-party trademarks or logos are subject to those third-party's policies. 


### Ethics
Data analysis was conducted with approval from the Microsoft Institutional Review Board. 

Data collection and use for research was performed in accordance with the [Microsoft Privacy Statement](https://go.microsoft.com/fwlink/?LinkId=521839).


### Contact
This research was conducted by members of [Microsoft Research](https://www.microsoft.com/en-us/research/). We welcome feedback. For questions, comments, or concerns about this repository, contact kitomlinson@microsoft.com. For broader inquiries about the paper, contact kitomlinson@microsoft.com, sojaffe@microsoft.com, counts@microsoft.com, suri@microsoft.com.
# "Working with AI" results files
These files contain IWA- and SOC-level results from the paper:

> Kiran Tomlinson, Sonia Jaffe, Will Wang, Scott Counts, and Siddharth Suri. Working with AI: Measuring the Occupational Implications of Generative AI. arXiv, 2025. https://arxiv.org/abs/2507.07935

## File descriptions
- `iwa_completion_ai.csv`: Fractions of conversations with each IWA as an AI action that are labeled as having their task completed.
- `iwa_completion_user.csv`: Fractions of conversations with each IWA as a user goal that are labeled as having their task completed.
- `iwa_impact_scope_ai.csv`: Fraction of AI action IWA matches where the IWA impact scope is labeled moderate or higher. 
- `iwa_impact_scope_user.csv`: Fraction of user goal IWA matches where the IWA impact scope is labeled moderate or higher. 
- `iwa_share_ai.csv`: AI action activity share for each IWA.
- `iwa_share_user.csv`: User goal activity share for each IWA.
- `soc_ai_applicability_score.csv`: AI applicability scores for each SOC (average of user goal and AI action scores).
- `soc_completion_ai.csv`: SOC-level weighted average AI action completion rates.
- `soc_completion_user.csv`: SOC-level weighted average user goal completion rates.
- `soc_coverage_ai.csv`: SOC AI action coverage.
- `soc_coverage_user.csv`: SOC user goal coverage.
- `soc_impact_scope_ai.csv`: SOC-level weighted average impact scope.
- `soc_impact_scope_user.csv`: SOC-level weighted average impact scope.
- `soc_iwa_weights.csv`: IWA weights for each SOC based on O*NET relevance and importance.
- `soc_to_iwas.csv`: Mapping of SOC codes to associated IWAs.

See the paper for definitions of activity share, completion, scope of impact, coverage, and AI applicability score metrics.


## Computing AI applicability score
The following code implements Equation (1) from the paper, averaging user and AI applicability scores. For convenience, AI applicability score is precomputed in `soc_ai_applicability_score.csv`.

```python
soc_ai_applicability_score = {soc : 0 for soc in soc_to_iwas}
coverage_threshold = 0.0005

for soc, iwas in soc_to_iwas.items():
    total_weight = sum(soc_iwa_weights[soc][iwa] for iwa in iwas)
    soc_ai_applicability_score[soc] = 0
    for iwa in iwas:
        # average of user goal score and AI action score
        if iwa_share_user[iwa] > coverage_threshold:
            soc_ai_applicability_score[soc] += 0.5 * (soc_iwa_weights[soc][iwa] / total_weight) * iwa_completion_user[iwa] * iwa_impact_scope_user[iwa]
        if iwa_share_ai[iwa] > coverage_threshold:
            soc_ai_applicability_score[soc] += 0.5 * (soc_iwa_weights[soc][iwa] / total_weight) * iwa_completion_ai[iwa] * iwa_impact_scope_ai[iwa]
```


## Additional public data
These results use the O*NET 29.0 Database, which can be downloaded here: https://www.onetcenter.org/dl_files/database/db_29_0_text.zip. Use the file `IWA Reference.txt` to convert IWA codes into titles.

To convert SOC Codes into job titles, use https://www.bls.gov/soc/2018/soc_2018_definitions.xlsx. See https://www.bls.gov/oes/tables.htm for Occupational Employment and Wage Statistics data linked by SOC Code.

Use https://www.bls.gov/emp/classifications-crosswalks/nem-onet-to-soc-crosswalk.xlsx to convert O\*NET-SOC Codes used to identify jobs in O\*NET to SOC Codes.

## Licence
TODO

## Citation
```
@misc{tomlinson2025working,
      title={Working with AI: Measuring the Occupational Implications of Generative AI}, 
      author={Kiran Tomlinson and Sonia Jaffe and Will Wang and Scott Counts and Siddharth Suri},
      year={2025},
      eprint={2507.07935},
      archivePrefix={arXiv},
      primaryClass={cs.AI},
      url={https://arxiv.org/abs/2507.07935}, 
}
```
# ChEMBL small-molecule dataset
## by Spencer Trinh


## Dataset

> ChEMBL Data is a manually curated database of small molecules used in drug discovery, including information about existing patented drugs

> The `Google BigQuery` query is shown below to obtain the three datasets. The three `csv` files were combined using `pandas`. There are three `csv` files because the free trial version of `Goolge BigQuery` has a limit to the memory usage for querying public datasets, therefore three separate queries were run and combined in the end.

> - First query:
```
SELECT *
  FROM (
  SELECT
  molregno,
  cmpd_rec.doc_id,
  comp.company,
  comp.country,
  prod.dosage_form,
  prod.route,
  prod.trade_name,
  prod.approval_date,
  prod.ad_type,
  prod.oral,
  prod.topical,
  prod.parenteral,
  ROW_NUMBER() OVER(PARTITION BY molregno ORDER BY molregno DESC) rn
  FROM `patents-public-data.ebi_chembl.compound_records_23` AS cmpd_rec
  JOIN `patents-public-data.ebi_chembl.molecule_synonyms_23` AS ms USING (molregno)
  JOIN `patents-public-data.ebi_chembl.research_companies_23` AS comp USING (res_stem_id)
  JOIN `patents-public-data.ebi_chembl.formulations` AS form USING (molregno)
  JOIN `patents-public-data.ebi_chembl.products_23` AS prod USING (product_id)

  ) as subq
 WHERE rn =1
 LIMIT 1000;

```

> - Second query:
```
SELECT *
  FROM (
  SELECT
  molregno,
  cmpd_rec.doc_id,
  cmpd_struc.canonical_smiles,
  cmpd_rec.compound_name,
  cmpd_prop.mw_freebase,
  cmpd_prop.alogp,
  cmpd_prop.hba,
  cmpd_prop.hbd,
  cmpd_prop.psa,
  cmpd_prop.rtb,
  cmpd_prop.ro3_pass,
  cmpd_prop.num_ro5_violations,
  cmpd_prop.acd_most_bpka,
  cmpd_prop.acd_logp,
  cmpd_prop.acd_logd,
  cmpd_prop.molecular_species,
  cmpd_prop.full_mwt,
  cmpd_prop.aromatic_rings,
  cmpd_prop.heavy_atoms,
  cmpd_prop.num_alerts,
  cmpd_prop.qed_weighted,
  cmpd_prop.mw_monoisotopic,
  cmpd_prop.full_molformula,
  cmpd_prop.hba_lipinski,
  cmpd_prop.hbd_lipinski,
  cmpd_prop.num_lipinski_ro5_violations,
  md.max_phase,
  md.therapeutic_flag,
  md.structure_type,
  md.molecule_type,
  md.first_approval,
  md.natural_product,
  md.availability_type,
  md.withdrawn_flag,
  ROW_NUMBER() OVER(PARTITION BY molregno ORDER BY molregno DESC) rn
  FROM `patents-public-data.ebi_chembl.compound_records_23` AS cmpd_rec
  JOIN `patents-public-data.ebi_chembl.compound_properties_23` AS cmpd_prop USING (molregno)
  JOIN `patents-public-data.ebi_chembl.compound_structures_23` AS cmpd_struc USING (molregno)
  JOIN `patents-public-data.ebi_chembl.molecule_dictionary_23` AS md USING (molregno)
  ) as subq
  WHERE rn = 1 AND
  molregno in ({LIST OF MOLREGNO});
```


> - Third query:
```
SELECT *
  FROM (
  SELECT
  cmpd_rec.molregno,
  cmpd_rec.doc_id,
  md.black_box_warning,
  md.inorganic_flag,
  md.indication_class,
  ROW_NUMBER() OVER(PARTITION BY cmpd_rec.molregno ORDER BY cmpd_rec.molregno DESC) rn
  FROM `patents-public-data.ebi_chembl.compound_records_23` AS cmpd_rec
  JOIN `patents-public-data.ebi_chembl.molecule_dictionary_23` AS md USING (molregno)
  ) as subq
  WHERE rn = 1 AND
  molregno in ({LIST OF MOLREGNO});
```

*where the `LIST OF MOLREGNO` is a list that was obtained from the first query*


## Summary of Findings

> The mean numerical features (molecular descriptors) of this small dataset are as follows:

|Feature | Mean Value|
| :---: | :---:|
|mw_freebase | 428.973270|
|alogp | 2.370098|
|psa | 93.431967|
|acd_most_bpka | 6.661588|
|acd_logp | 2.357922|
|acd_logd | 1.053922|
|full_mwt | 463.672533|
|qed_weighted | 0.550195|
|hba | 4.993333|
|hbd | 1.873333|
|rtb | 5.558667|
|num_ro5_violations | 0.360000|
|aromatic_rings | 1.484000|
|heavy_atoms | 25.386667|
|num_alerts | 0.928000|
|hba_lipinski | 5.901333|
|hbd_lipinski | 2.117333|
|num_lipinski_ro5_violations | 0.254667|


> Furthermore, the most common chemical subgroup is benzene; the US has developed the most drug compounds and more specifically Pfizer has the most drugs on the market.

## Key Insights for Presentation

> In this dataset, neutral compounds are the most common at 349 in count. The least are zwitterions which are molecules or ions having separate positively and negatively charged groups. The nitrile group is acidic and seems to have the highest qed scores. Since benzene chemical subgroups are so ubiquitous they have very high density in the inner region of the violin plots. Molecules or compounds that have peptide bonds and are considered a zwitterion have very low qed scores, likewise with terpenes. Nitrile groups seem to have the highes qed scores. This seems to be correct since nitriles are very polar and act as a good hydrogen bond acceptor.

> Benzene is very ubiquitous in all organic compounds, thus it was observed in most of the drug compounds in this dataset. Next in line are the ester groups and then double bond functional groups. Because this dataset is quite small, perhaps we aren't getting the full picture of how these chemical subgroups really count up. The selection of these chemical groups was arbitrary. There may be other chemical groups that could have been better to choose.

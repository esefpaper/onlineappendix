# ESEF-Berichterstattung in Deutschland: Herausforderungen und Chancen

## Der Python-Code zum Replizieren der Tabellen 1–3

Das Jupyter-Notebook befindet sich unter: https://github.com/esefpaper/onlineappendix/blob/main/code/Tabellen.ipynb



Der Inhalt wird auch unten angezeigt (als Markdown).

```bash
!pip install --force-reinstall numpy==2.2.4 pandas==2.2.3
!pip install openpyxl
```



```python
import pandas as pd
import re
```

### Tabelle 1

Tabelle 1 zeigt die ESEF-Konformitätsraten deut-scher kapitalmarktorientierter Unternehmen über den Beobachtungszeitraum von 2020 bis 2023. 


```python
tb1_uregdw_s5 = pd.read_csv("../data/tb1_uregdw_s5_20250616.csv.gz", sep="|", compression="gzip")
```


```python
def create_tb1_panel(df, byvar, ascending =True):
    # Group by the specified variable
    tb1_de_rows = df.groupby([byvar]).gvkey.nunique().reset_index().rename(columns={"gvkey": "gvkey_nunique"})
    
    # Create the different groupings
    tb1c_01 = df.groupby([byvar]).gvkey.count().reset_index()
    tb1c_02 = df.loc[df.sum_filing_score>=8].groupby([byvar]).gvkey.count().reset_index()
    tb1c_03 = df.loc[df.sum_filing_score>=16].groupby([byvar]).gvkey.count().reset_index()
    tb1c_04 = df.loc[df.sum_filing_score>=32768].groupby([byvar]).gvkey.count().reset_index()
    
    # Merge all the data
    result = pd.merge(tb1_de_rows, tb1c_01, on=byvar, how="left", suffixes=("", "")).fillna(0)
    result = pd.merge(result, tb1c_02, on=byvar, how="left", suffixes=("", "_esef")).fillna(0)
    result["anteil_esef"] = result["gvkey_esef"]/result["gvkey"]
    result["anteil_esef"] = result["anteil_esef"].apply(lambda x: round(x*100, 1))
    
    result = pd.merge(result, tb1c_03, on=byvar, how="left", suffixes=("", "_xbrl")).fillna(0)
    result["anteil_xbrl"] = result["gvkey_xbrl"]/result["gvkey"]
    result["anteil_xbrl"] = result["anteil_xbrl"].apply(lambda x: round(x*100, 1))
    
    result = pd.merge(result, tb1c_04, on=byvar, how="left", suffixes=("", "_complete")).fillna(0)
    result["anteil_complete"] = result["gvkey_complete"]/result["gvkey"]
    result["anteil_complete"] = result["anteil_complete"].apply(lambda x: round(x*100, 1))
    
    result.fillna(0, inplace=True)
    result.sort_values(by=byvar, ascending=ascending, inplace=True)
    result.to_excel(f"../results/tb1_by{byvar}.xlsx")
    return result

```


```python
# Panel A
tb1_by_formyear = create_tb1_panel(tb1_uregdw_s5, "formyear")
tb1_by_formyear
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>formyear</th>
      <th>gvkey_nunique</th>
      <th>gvkey</th>
      <th>gvkey_esef</th>
      <th>anteil_esef</th>
      <th>gvkey_xbrl</th>
      <th>anteil_xbrl</th>
      <th>gvkey_complete</th>
      <th>anteil_complete</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020</td>
      <td>596</td>
      <td>596</td>
      <td>354</td>
      <td>59.4</td>
      <td>330</td>
      <td>55.4</td>
      <td>317</td>
      <td>53.2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021</td>
      <td>595</td>
      <td>595</td>
      <td>410</td>
      <td>68.9</td>
      <td>405</td>
      <td>68.1</td>
      <td>362</td>
      <td>60.8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022</td>
      <td>572</td>
      <td>572</td>
      <td>397</td>
      <td>69.4</td>
      <td>392</td>
      <td>68.5</td>
      <td>355</td>
      <td>62.1</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2023</td>
      <td>492</td>
      <td>492</td>
      <td>368</td>
      <td>74.8</td>
      <td>362</td>
      <td>73.6</td>
      <td>330</td>
      <td>67.1</td>
    </tr>
  </tbody>
</table>
</div>




```python
# table 1 2020--2023 ESEF Sample period
tb1_uregdw_s5["2020--2023"] = tb1_uregdw_s5.formyear.apply(lambda x: True if x>=2020 & x<=2024 else False)

byvar = "2020--2023"
tb1_esef = create_tb1_panel(tb1_uregdw_s5, byvar, ascending=False)
tb1_esef
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>2020--2023</th>
      <th>gvkey_nunique</th>
      <th>gvkey</th>
      <th>gvkey_esef</th>
      <th>anteil_esef</th>
      <th>gvkey_xbrl</th>
      <th>anteil_xbrl</th>
      <th>gvkey_complete</th>
      <th>anteil_complete</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>True</td>
      <td>610</td>
      <td>2255</td>
      <td>1529</td>
      <td>67.8</td>
      <td>1489</td>
      <td>66.0</td>
      <td>1364</td>
      <td>60.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
byvar = "FSE_Label"
tb1_by_FSE_label = create_tb1_panel(tb1_uregdw_s5, byvar)
tb1_by_FSE_label
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FSE_Label</th>
      <th>gvkey_nunique</th>
      <th>gvkey</th>
      <th>gvkey_esef</th>
      <th>anteil_esef</th>
      <th>gvkey_xbrl</th>
      <th>anteil_xbrl</th>
      <th>gvkey_complete</th>
      <th>anteil_complete</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1. DAX40</td>
      <td>42</td>
      <td>163</td>
      <td>157</td>
      <td>96.3</td>
      <td>157</td>
      <td>96.3</td>
      <td>155</td>
      <td>95.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2. Prime Standard</td>
      <td>208</td>
      <td>806</td>
      <td>707</td>
      <td>87.7</td>
      <td>701</td>
      <td>87.0</td>
      <td>681</td>
      <td>84.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3. SME</td>
      <td>82</td>
      <td>299</td>
      <td>220</td>
      <td>73.6</td>
      <td>213</td>
      <td>71.2</td>
      <td>186</td>
      <td>62.2</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4. Aktuell nicht als Aktie an FWB</td>
      <td>282</td>
      <td>987</td>
      <td>445</td>
      <td>45.1</td>
      <td>418</td>
      <td>42.4</td>
      <td>342</td>
      <td>34.7</td>
    </tr>
  </tbody>
</table>
</div>




```python
tb1_uregdw_s5["DE_ISIN"] = tb1_uregdw_s5["isin"].str.startswith("DE")
tb1_uregdw_s5["DE_ISIN"] = tb1_uregdw_s5["DE_ISIN"].fillna(True)

byvar = "DE_ISIN"
tb1_de_isin = create_tb1_panel(tb1_uregdw_s5, byvar, ascending=False)
tb1_de_isin
```

    /var/folders/mz/hx4rhsms565c3zkfh7qx8xs00000gn/T/ipykernel_43243/187706699.py:2: FutureWarning: Downcasting object dtype arrays on .fillna, .ffill, .bfill is deprecated and will change in a future version. Call result.infer_objects(copy=False) instead. To opt-in to the future behavior, set `pd.set_option('future.no_silent_downcasting', True)`
      tb1_uregdw_s5["DE_ISIN"] = tb1_uregdw_s5["DE_ISIN"].fillna(True)





<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DE_ISIN</th>
      <th>gvkey_nunique</th>
      <th>gvkey</th>
      <th>gvkey_esef</th>
      <th>anteil_esef</th>
      <th>gvkey_xbrl</th>
      <th>anteil_xbrl</th>
      <th>gvkey_complete</th>
      <th>anteil_complete</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>True</td>
      <td>590</td>
      <td>2189</td>
      <td>1492</td>
      <td>68.2</td>
      <td>1452</td>
      <td>66.3</td>
      <td>1330</td>
      <td>60.8</td>
    </tr>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>20</td>
      <td>66</td>
      <td>37</td>
      <td>56.1</td>
      <td>37</td>
      <td>56.1</td>
      <td>34</td>
      <td>51.5</td>
    </tr>
  </tbody>
</table>
</div>




```python
byvar = "mcap_g4"
tb1_de_mcap_beg = create_tb1_panel(tb1_uregdw_s5, byvar, ascending=False)
tb1_de_mcap_beg
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mcap_g4</th>
      <th>gvkey_nunique</th>
      <th>gvkey</th>
      <th>gvkey_esef</th>
      <th>anteil_esef</th>
      <th>gvkey_xbrl</th>
      <th>anteil_xbrl</th>
      <th>gvkey_complete</th>
      <th>anteil_complete</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>MCap4</td>
      <td>160</td>
      <td>528</td>
      <td>360</td>
      <td>68.2</td>
      <td>353</td>
      <td>66.9</td>
      <td>312</td>
      <td>59.1</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MCap3</td>
      <td>159</td>
      <td>476</td>
      <td>359</td>
      <td>75.4</td>
      <td>357</td>
      <td>75.0</td>
      <td>342</td>
      <td>71.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MCap2</td>
      <td>219</td>
      <td>661</td>
      <td>433</td>
      <td>65.5</td>
      <td>420</td>
      <td>63.5</td>
      <td>380</td>
      <td>57.5</td>
    </tr>
    <tr>
      <th>0</th>
      <td>MCap1</td>
      <td>186</td>
      <td>590</td>
      <td>377</td>
      <td>63.9</td>
      <td>359</td>
      <td>60.8</td>
      <td>330</td>
      <td>55.9</td>
    </tr>
  </tbody>
</table>
</div>




```python
byvar = "gsec_type"
tb1_de_mcap_beg = create_tb1_panel(tb1_uregdw_s5, byvar, ascending=True)
tb1_de_mcap_beg
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gsec_type</th>
      <th>gvkey_nunique</th>
      <th>gvkey</th>
      <th>gvkey_esef</th>
      <th>anteil_esef</th>
      <th>gvkey_xbrl</th>
      <th>anteil_xbrl</th>
      <th>gvkey_complete</th>
      <th>anteil_complete</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10 Energie</td>
      <td>7</td>
      <td>26</td>
      <td>13</td>
      <td>50.0</td>
      <td>13</td>
      <td>50.0</td>
      <td>12</td>
      <td>46.2</td>
    </tr>
    <tr>
      <th>1</th>
      <td>15 Roh- und Grundstoffe</td>
      <td>32</td>
      <td>115</td>
      <td>94</td>
      <td>81.7</td>
      <td>89</td>
      <td>77.4</td>
      <td>77</td>
      <td>67.0</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20 Industrie</td>
      <td>136</td>
      <td>519</td>
      <td>398</td>
      <td>76.7</td>
      <td>395</td>
      <td>76.1</td>
      <td>373</td>
      <td>71.9</td>
    </tr>
    <tr>
      <th>3</th>
      <td>25 Verbraucher Diskretionäre</td>
      <td>90</td>
      <td>329</td>
      <td>223</td>
      <td>67.8</td>
      <td>218</td>
      <td>66.3</td>
      <td>196</td>
      <td>59.6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>30 Verbraucher Staples</td>
      <td>25</td>
      <td>92</td>
      <td>44</td>
      <td>47.8</td>
      <td>44</td>
      <td>47.8</td>
      <td>44</td>
      <td>47.8</td>
    </tr>
    <tr>
      <th>5</th>
      <td>35 Gesundheitswesen</td>
      <td>61</td>
      <td>222</td>
      <td>145</td>
      <td>65.3</td>
      <td>135</td>
      <td>60.8</td>
      <td>124</td>
      <td>55.9</td>
    </tr>
    <tr>
      <th>6</th>
      <td>40 Finanzen</td>
      <td>53</td>
      <td>206</td>
      <td>160</td>
      <td>77.7</td>
      <td>152</td>
      <td>73.8</td>
      <td>126</td>
      <td>61.2</td>
    </tr>
    <tr>
      <th>7</th>
      <td>45 Informationstechnologie</td>
      <td>109</td>
      <td>386</td>
      <td>201</td>
      <td>52.1</td>
      <td>200</td>
      <td>51.8</td>
      <td>194</td>
      <td>50.3</td>
    </tr>
    <tr>
      <th>8</th>
      <td>50 Telekommunikation</td>
      <td>46</td>
      <td>167</td>
      <td>111</td>
      <td>66.5</td>
      <td>108</td>
      <td>64.7</td>
      <td>101</td>
      <td>60.5</td>
    </tr>
    <tr>
      <th>9</th>
      <td>55 Energieversorgung</td>
      <td>16</td>
      <td>61</td>
      <td>42</td>
      <td>68.9</td>
      <td>41</td>
      <td>67.2</td>
      <td>36</td>
      <td>59.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>60 Immobilien</td>
      <td>32</td>
      <td>122</td>
      <td>98</td>
      <td>80.3</td>
      <td>94</td>
      <td>77.0</td>
      <td>81</td>
      <td>66.4</td>
    </tr>
  </tbody>
</table>
</div>



### Tabelle 2

Tabelle 2 analysiert die Verwendung von Standard-IFRS-Taxonomie-Tags und firmenspezifischen Erweiterungstags in den ESEF-Berichten. Es ist zu beachten, dass die Methodik das Zählen der Anzahl der Konzepte unter der XBRL-Taxonomie umfasst, nicht jedoch die Anzahl der Fakten.


```python
data_tb2 = pd.read_csv("../data/data_tb2_20250616.csv.gz", sep="|", compression="gzip")
data_tb2i = pd.read_csv("../data/data_tb2i_20250616.csv.gz", sep="|", compression="gzip")

```


```python
def create_tb2_panel(data_tb2, byvar, ascending=True):
    # Group by the specified variable
    tb2a_rows = data_tb2.groupby([byvar]).gvkey.nunique().reset_index().rename(columns={"gvkey": "gvkey_nunique"})

    tb2a_01 = data_tb2.groupby([byvar, "gvkey"]).concept_name.nunique().reset_index().groupby(byvar).concept_name.mean().apply(lambda x: round(x, 1)).reset_index()
    tb2a_02 = data_tb2.loc[ ~data_tb2.concept_is_extended].groupby([byvar, "gvkey"]).concept_name.nunique().reset_index().groupby(byvar).concept_name.mean().apply(lambda x: round(x, 1)).reset_index()
    tb2a_03 = data_tb2.loc[ data_tb2.concept_is_extended].groupby([byvar, "gvkey"]).concept_name.nunique().reset_index().groupby(byvar).concept_name.mean().apply(lambda x: round(x, 1)).reset_index()

    tb2a = pd.merge(tb2a_01, tb2a_02, on=byvar, how="left", suffixes=("", "_ifrs")).fillna(0)
    tb2a = pd.merge(tb2a, tb2a_03, on=byvar, how="left", suffixes=("", "_erw")).fillna(0)

    tb2a.fillna(0, inplace=True)
    tb2a["concept_name"] = tb2a["concept_name_ifrs"] + tb2a["concept_name_erw"]
    tb2a["anteil_ifrs"] = tb2a["concept_name_ifrs"]/tb2a["concept_name"]
    tb2a["anteil_ifrs"] = tb2a["anteil_ifrs"].apply(lambda x: round(x*100, 1))

    tb2a["anteil_erw"] = tb2a["concept_name_erw"]/tb2a["concept_name"]
    tb2a["anteil_erw"] = tb2a["anteil_erw"].apply(lambda x: round(x*100, 1))


    tb2a.sort_values(by=byvar, ascending=ascending, inplace=True)
    tb2a.to_excel(f"../results/tb2_{byvar}.xlsx")
    tb2a

    return tb2a

```


```python
#formyear
tb2_by_formyear = create_tb2_panel(data_tb2, "formyear")
tb2_by_formyear
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>formyear</th>
      <th>concept_name</th>
      <th>concept_name_ifrs</th>
      <th>concept_name_erw</th>
      <th>anteil_ifrs</th>
      <th>anteil_erw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>2020</td>
      <td>142.8</td>
      <td>127.8</td>
      <td>15.0</td>
      <td>89.5</td>
      <td>10.5</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2021</td>
      <td>143.8</td>
      <td>128.5</td>
      <td>15.3</td>
      <td>89.4</td>
      <td>10.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2022</td>
      <td>266.8</td>
      <td>251.7</td>
      <td>15.1</td>
      <td>94.3</td>
      <td>5.7</td>
    </tr>
    <tr>
      <th>3</th>
      <td>2023</td>
      <td>282.5</td>
      <td>267.8</td>
      <td>14.7</td>
      <td>94.8</td>
      <td>5.2</td>
    </tr>
  </tbody>
</table>
</div>




```python
#statement_type / TagType
tb2_by_statement_type = create_tb2_panel(data_tb2i, "statement_type")
tb2_by_statement_type
```




<div>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>statement_type</th>
      <th>concept_name</th>
      <th>concept_name_ifrs</th>
      <th>concept_name_erw</th>
      <th>anteil_ifrs</th>
      <th>anteil_erw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>0. General</td>
      <td>2.3</td>
      <td>2.3</td>
      <td>0.0</td>
      <td>100.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>1</th>
      <td>1. Bilanz</td>
      <td>42.7</td>
      <td>35.7</td>
      <td>7.0</td>
      <td>83.6</td>
      <td>16.4</td>
    </tr>
    <tr>
      <th>2</th>
      <td>2. GuV</td>
      <td>25.5</td>
      <td>19.9</td>
      <td>5.6</td>
      <td>78.0</td>
      <td>22.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>3. GKV/UKV</td>
      <td>17.7</td>
      <td>12.9</td>
      <td>4.8</td>
      <td>72.9</td>
      <td>27.1</td>
    </tr>
    <tr>
      <th>4</th>
      <td>4. Kapitalflussrechnung</td>
      <td>34.5</td>
      <td>24.0</td>
      <td>10.5</td>
      <td>69.6</td>
      <td>30.4</td>
    </tr>
    <tr>
      <th>5</th>
      <td>5. EK-Veränderungsrechnung</td>
      <td>20.6</td>
      <td>16.1</td>
      <td>4.5</td>
      <td>78.2</td>
      <td>21.8</td>
    </tr>
    <tr>
      <th>6</th>
      <td>6. Nettovermögensänderung</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>100.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>7. Anhang</td>
      <td>164.7</td>
      <td>160.1</td>
      <td>4.6</td>
      <td>97.2</td>
      <td>2.8</td>
    </tr>
  </tbody>
</table>
</div>




```python
#FSE_Label
tb2_by_FSE_Label = create_tb2_panel(data_tb2, "FSE_Label")
tb2_by_FSE_Label
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>FSE_Label</th>
      <th>concept_name</th>
      <th>concept_name_ifrs</th>
      <th>concept_name_erw</th>
      <th>anteil_ifrs</th>
      <th>anteil_erw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>1. DAX40</td>
      <td>364.6</td>
      <td>316.8</td>
      <td>47.8</td>
      <td>86.9</td>
      <td>13.1</td>
    </tr>
    <tr>
      <th>1</th>
      <td>2. Prime Standard</td>
      <td>308.4</td>
      <td>284.9</td>
      <td>23.5</td>
      <td>92.4</td>
      <td>7.6</td>
    </tr>
    <tr>
      <th>2</th>
      <td>3. SME</td>
      <td>263.0</td>
      <td>243.5</td>
      <td>19.5</td>
      <td>92.6</td>
      <td>7.4</td>
    </tr>
    <tr>
      <th>3</th>
      <td>4. Aktuell nicht als Aktie an FWB</td>
      <td>266.9</td>
      <td>242.0</td>
      <td>24.9</td>
      <td>90.7</td>
      <td>9.3</td>
    </tr>
  </tbody>
</table>
</div>




```python
#DE_ISIN
data_tb2["DE_ISIN"] = data_tb2["isin"].str.startswith("DE")
tb2_by_gsec_type = create_tb2_panel(data_tb2, "DE_ISIN", ascending=False)
tb2_by_gsec_type
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>DE_ISIN</th>
      <th>concept_name</th>
      <th>concept_name_ifrs</th>
      <th>concept_name_erw</th>
      <th>anteil_ifrs</th>
      <th>anteil_erw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>1</th>
      <td>True</td>
      <td>309.3</td>
      <td>283.3</td>
      <td>26.0</td>
      <td>91.6</td>
      <td>8.4</td>
    </tr>
    <tr>
      <th>0</th>
      <td>False</td>
      <td>287.4</td>
      <td>258.8</td>
      <td>28.6</td>
      <td>90.0</td>
      <td>10.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
#mcap_g4
tb2_by_mcap_g4 = create_tb2_panel(data_tb2, "mcap_g4", ascending=False)
tb2_by_mcap_g4
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>mcap_g4</th>
      <th>concept_name</th>
      <th>concept_name_ifrs</th>
      <th>concept_name_erw</th>
      <th>anteil_ifrs</th>
      <th>anteil_erw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>3</th>
      <td>MCap4</td>
      <td>302.4</td>
      <td>266.7</td>
      <td>35.7</td>
      <td>88.2</td>
      <td>11.8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>MCap3</td>
      <td>284.7</td>
      <td>262.7</td>
      <td>22.0</td>
      <td>92.3</td>
      <td>7.7</td>
    </tr>
    <tr>
      <th>1</th>
      <td>MCap2</td>
      <td>261.0</td>
      <td>242.3</td>
      <td>18.7</td>
      <td>92.8</td>
      <td>7.2</td>
    </tr>
    <tr>
      <th>0</th>
      <td>MCap1</td>
      <td>261.0</td>
      <td>243.8</td>
      <td>17.2</td>
      <td>93.4</td>
      <td>6.6</td>
    </tr>
  </tbody>
</table>
</div>




```python
#gsec_type
tb2_by_gsec_type = create_tb2_panel(data_tb2, "gsec_type")
tb2_by_gsec_type
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>gsec_type</th>
      <th>concept_name</th>
      <th>concept_name_ifrs</th>
      <th>concept_name_erw</th>
      <th>anteil_ifrs</th>
      <th>anteil_erw</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>10 Energie</td>
      <td>320.3</td>
      <td>282.5</td>
      <td>37.8</td>
      <td>88.2</td>
      <td>11.8</td>
    </tr>
    <tr>
      <th>1</th>
      <td>15 Roh- und Grundstoffe</td>
      <td>325.3</td>
      <td>296.3</td>
      <td>29.0</td>
      <td>91.1</td>
      <td>8.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>20 Industrie</td>
      <td>305.3</td>
      <td>280.8</td>
      <td>24.5</td>
      <td>92.0</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>25 Verbraucher Diskretionäre</td>
      <td>281.3</td>
      <td>258.9</td>
      <td>22.4</td>
      <td>92.0</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>4</th>
      <td>30 Verbraucher Staples</td>
      <td>305.6</td>
      <td>279.7</td>
      <td>25.9</td>
      <td>91.5</td>
      <td>8.5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>35 Gesundheitswesen</td>
      <td>291.2</td>
      <td>267.6</td>
      <td>23.6</td>
      <td>91.9</td>
      <td>8.1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>40 Finanzen</td>
      <td>322.6</td>
      <td>274.2</td>
      <td>48.4</td>
      <td>85.0</td>
      <td>15.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>45 Informationstechnologie</td>
      <td>279.3</td>
      <td>263.6</td>
      <td>15.7</td>
      <td>94.4</td>
      <td>5.6</td>
    </tr>
    <tr>
      <th>8</th>
      <td>50 Telekommunikation</td>
      <td>270.3</td>
      <td>254.1</td>
      <td>16.2</td>
      <td>94.0</td>
      <td>6.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>55 Energieversorgung</td>
      <td>323.1</td>
      <td>280.7</td>
      <td>42.4</td>
      <td>86.9</td>
      <td>13.1</td>
    </tr>
    <tr>
      <th>10</th>
      <td>60 Immobilien</td>
      <td>297.2</td>
      <td>262.2</td>
      <td>35.0</td>
      <td>88.2</td>
      <td>11.8</td>
    </tr>
  </tbody>
</table>
</div>





### Tabelle 3

Tabelle 3 analysiert die Verwendung von XBRL-Markierungen im Anhang zum Jahresabschluss. Sie zeigt die Anzahl der Unternehmen und Berichte, die bestimmte Anhangangaben taggen, sowie die durchschnittliche Anzahl der Tags pro Anhangang-abe und den Anteil der Text- und Zahlentags. 


```python
data_tb2i["reportkey"] = data_tb2i.gvkey.astype(str)+data_tb2i.formyear.astype(str)
data_tb3 = data_tb2i.loc[data_tb2i.statement_type=="7. Anhang"].copy()
```


```python
byvar = "TopConcept"
tb2a_01 = data_tb3.groupby(["TopConcept", "gvkey"]).concept_name.nunique().reset_index().groupby("TopConcept").concept_name.mean().apply(lambda x: round(x, 1)).reset_index()
tb2a_01.head(10)
```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TopConcept</th>
      <th>concept_name</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[800100] Notes - Subclassifications of assets,...</td>
      <td>10.9</td>
    </tr>
    <tr>
      <th>1</th>
      <td>[800200] Notes - Analysis of income and expense</td>
      <td>3.9</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[800300] Notes - Statement of cash flows, addi...</td>
      <td>8.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[800400] Notes - Statement of changes in equit...</td>
      <td>2.6</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[800500] Notes - List of notes</td>
      <td>85.5</td>
    </tr>
    <tr>
      <th>5</th>
      <td>[800610] Notes - List of material accounting p...</td>
      <td>49.1</td>
    </tr>
    <tr>
      <th>6</th>
      <td>[810000] Notes - Corporate information and sta...</td>
      <td>10.7</td>
    </tr>
    <tr>
      <th>7</th>
      <td>[811000] Notes - Accounting policies, changes ...</td>
      <td>1.7</td>
    </tr>
    <tr>
      <th>8</th>
      <td>[813000] Notes - Interim financial reporting</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>9</th>
      <td>[815000] Notes - Events after reporting period</td>
      <td>2.0</td>
    </tr>
  </tbody>
</table>
</div>




```python
data_tb3["type_notetag_text"] = data_tb3.Type_y.astype(str).str.contains("Text|Abstract|Axis|Disclsoure|Lineitems|Textblock")
data_tb3.type_notetag_text = data_tb3.type_notetag_text.fillna(True)
data_tb3.loc[data_tb3.TopType.fillna("").str.contains("800500|800610")].type_notetag_text.value_counts()

tb2a_02 = data_tb3.loc[ ~data_tb3.type_notetag_text].groupby([byvar, "gvkey"]).concept_name.nunique().reset_index().groupby(byvar).concept_name.mean().apply(lambda x: round(x, 1)).reset_index()

```


```python
tb3_firms = data_tb3.groupby(["TopConcept"]).gvkey.nunique().reset_index().rename(columns={"gvkey": "nfirms"})
tb3_reportkeys = data_tb3.groupby(["TopConcept"]).reportkey.nunique().reset_index().rename(columns={"reportkey": "nreports"})
tb3_rows = pd.merge(tb3_firms, tb3_reportkeys, on="TopConcept", how="left", suffixes=("", "_")).fillna(0)

tb3_rows = pd.merge(tb3_rows, tb2a_01, on="TopConcept", how="left", suffixes=("", "")).fillna(0)
tb3_rows = pd.merge(tb3_rows, tb2a_02, on="TopConcept", how="left", suffixes=("", "_num")).fillna(0)
tb3_rows.sort_values(by="nreports", ascending=False, inplace=True)

```


```python
tb2a_03 = data_tb3.loc[ data_tb3.type_notetag_text].groupby([byvar, "gvkey"]).concept_name.nunique().reset_index().groupby(byvar).concept_name.mean().apply(lambda x: round(x, 1)).reset_index()

tb3 = pd.merge(tb3_rows, tb2a_03, on="TopConcept", how="left", suffixes=("", "_text")).fillna(0)

tb3.fillna(0, inplace=True)
tb3.head(10)

```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TopConcept</th>
      <th>nfirms</th>
      <th>nreports</th>
      <th>concept_name</th>
      <th>concept_name_num</th>
      <th>concept_name_text</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>[810000] Notes - Corporate information and sta...</td>
      <td>410</td>
      <td>1357</td>
      <td>10.7</td>
      <td>2.4</td>
      <td>9.4</td>
    </tr>
    <tr>
      <th>1</th>
      <td>[800100] Notes - Subclassifications of assets,...</td>
      <td>410</td>
      <td>1353</td>
      <td>10.9</td>
      <td>10.8</td>
      <td>1.5</td>
    </tr>
    <tr>
      <th>2</th>
      <td>[800300] Notes - Statement of cash flows, addi...</td>
      <td>404</td>
      <td>1320</td>
      <td>8.0</td>
      <td>8.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>[800200] Notes - Analysis of income and expense</td>
      <td>370</td>
      <td>1204</td>
      <td>3.9</td>
      <td>3.9</td>
      <td>1.5</td>
    </tr>
    <tr>
      <th>4</th>
      <td>[800400] Notes - Statement of changes in equit...</td>
      <td>382</td>
      <td>1182</td>
      <td>2.6</td>
      <td>2.6</td>
      <td>1.0</td>
    </tr>
    <tr>
      <th>5</th>
      <td>[800500] Notes - List of notes</td>
      <td>359</td>
      <td>648</td>
      <td>85.5</td>
      <td>0.0</td>
      <td>85.5</td>
    </tr>
    <tr>
      <th>6</th>
      <td>[800610] Notes - List of material accounting p...</td>
      <td>359</td>
      <td>648</td>
      <td>49.1</td>
      <td>0.0</td>
      <td>49.1</td>
    </tr>
    <tr>
      <th>7</th>
      <td>[832610] Notes - Leases</td>
      <td>200</td>
      <td>621</td>
      <td>2.1</td>
      <td>2.1</td>
      <td>1.6</td>
    </tr>
    <tr>
      <th>8</th>
      <td>[851100] Notes - Cash flow statement</td>
      <td>162</td>
      <td>418</td>
      <td>1.6</td>
      <td>1.5</td>
      <td>1.2</td>
    </tr>
    <tr>
      <th>9</th>
      <td>[822390] Notes - Financial instruments</td>
      <td>69</td>
      <td>157</td>
      <td>1.3</td>
      <td>1.2</td>
      <td>1.2</td>
    </tr>
  </tbody>
</table>
</div>




```python

tb3["concept_name"] = tb3["concept_name_text"] + tb3["concept_name_num"]

tb3["anteil_num"] = tb3["concept_name_num"]/tb3["concept_name"]
tb3["anteil_num"] = tb3["anteil_num"].apply(lambda x: round(x*100, 1))
tb3["anteil_text"] = tb3["concept_name_text"]/tb3["concept_name"]
tb3["anteil_text"] = tb3["anteil_text"].apply(lambda x: round(x*100, 1))

tb3["TopConcept"] = tb3["TopConcept"].apply(lambda x: re.sub("^\[\d{6}\]\s?","", str(x)))

tb3.sort_values(by="nreports", ascending=False, inplace=True)
tb3[["TopConcept", "nfirms", "nreports", "concept_name", "concept_name_text", "anteil_text", "concept_name_num",  "anteil_num"]].to_excel(f"../results/tb3_TopConcept.xlsx")
tb3[["TopConcept", "nfirms", "nreports", "concept_name", "concept_name_text", "anteil_text", "concept_name_num",  "anteil_num"]]


```




<div>

<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>TopConcept</th>
      <th>nfirms</th>
      <th>nreports</th>
      <th>concept_name</th>
      <th>concept_name_text</th>
      <th>anteil_text</th>
      <th>concept_name_num</th>
      <th>anteil_num</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>Notes - Corporate information and statement of...</td>
      <td>410</td>
      <td>1357</td>
      <td>11.8</td>
      <td>9.4</td>
      <td>79.7</td>
      <td>2.4</td>
      <td>20.3</td>
    </tr>
    <tr>
      <th>1</th>
      <td>Notes - Subclassifications of assets, liabilit...</td>
      <td>410</td>
      <td>1353</td>
      <td>12.3</td>
      <td>1.5</td>
      <td>12.2</td>
      <td>10.8</td>
      <td>87.8</td>
    </tr>
    <tr>
      <th>2</th>
      <td>Notes - Statement of cash flows, additional di...</td>
      <td>404</td>
      <td>1320</td>
      <td>8.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>8.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>3</th>
      <td>Notes - Analysis of income and expense</td>
      <td>370</td>
      <td>1204</td>
      <td>5.4</td>
      <td>1.5</td>
      <td>27.8</td>
      <td>3.9</td>
      <td>72.2</td>
    </tr>
    <tr>
      <th>4</th>
      <td>Notes - Statement of changes in equity, additi...</td>
      <td>382</td>
      <td>1182</td>
      <td>3.6</td>
      <td>1.0</td>
      <td>27.8</td>
      <td>2.6</td>
      <td>72.2</td>
    </tr>
    <tr>
      <th>5</th>
      <td>Notes - List of notes</td>
      <td>359</td>
      <td>648</td>
      <td>85.5</td>
      <td>85.5</td>
      <td>100.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>6</th>
      <td>Notes - List of material accounting policy inf...</td>
      <td>359</td>
      <td>648</td>
      <td>49.1</td>
      <td>49.1</td>
      <td>100.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>7</th>
      <td>Notes - Leases</td>
      <td>200</td>
      <td>621</td>
      <td>3.7</td>
      <td>1.6</td>
      <td>43.2</td>
      <td>2.1</td>
      <td>56.8</td>
    </tr>
    <tr>
      <th>8</th>
      <td>Notes - Cash flow statement</td>
      <td>162</td>
      <td>418</td>
      <td>2.7</td>
      <td>1.2</td>
      <td>44.4</td>
      <td>1.5</td>
      <td>55.6</td>
    </tr>
    <tr>
      <th>9</th>
      <td>Notes - Financial instruments</td>
      <td>69</td>
      <td>157</td>
      <td>2.4</td>
      <td>1.2</td>
      <td>50.0</td>
      <td>1.2</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>10</th>
      <td>Notes - Non-current asset held for sale and di...</td>
      <td>62</td>
      <td>131</td>
      <td>5.6</td>
      <td>1.2</td>
      <td>21.4</td>
      <td>4.4</td>
      <td>78.6</td>
    </tr>
    <tr>
      <th>11</th>
      <td>Notes - Income taxes</td>
      <td>41</td>
      <td>126</td>
      <td>2.8</td>
      <td>1.0</td>
      <td>35.7</td>
      <td>1.8</td>
      <td>64.3</td>
    </tr>
    <tr>
      <th>12</th>
      <td>Notes - Earnings per share</td>
      <td>51</td>
      <td>121</td>
      <td>2.7</td>
      <td>1.0</td>
      <td>37.0</td>
      <td>1.7</td>
      <td>63.0</td>
    </tr>
    <tr>
      <th>13</th>
      <td>Notes - Effects of changes in foreign exchange...</td>
      <td>35</td>
      <td>110</td>
      <td>2.1</td>
      <td>1.0</td>
      <td>47.6</td>
      <td>1.1</td>
      <td>52.4</td>
    </tr>
    <tr>
      <th>14</th>
      <td>Notes - Revenue from contracts with customers</td>
      <td>29</td>
      <td>72</td>
      <td>2.2</td>
      <td>1.1</td>
      <td>50.0</td>
      <td>1.1</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>15</th>
      <td>Notes - Investment property</td>
      <td>26</td>
      <td>71</td>
      <td>4.4</td>
      <td>1.8</td>
      <td>40.9</td>
      <td>2.6</td>
      <td>59.1</td>
    </tr>
    <tr>
      <th>16</th>
      <td>Notes - Employee benefits</td>
      <td>22</td>
      <td>52</td>
      <td>3.2</td>
      <td>2.2</td>
      <td>68.8</td>
      <td>1.0</td>
      <td>31.2</td>
    </tr>
    <tr>
      <th>17</th>
      <td>Notes - Business combinations</td>
      <td>27</td>
      <td>51</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>50.0</td>
      <td>1.0</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>18</th>
      <td>Notes - Interests in other entities</td>
      <td>25</td>
      <td>50</td>
      <td>2.3</td>
      <td>1.3</td>
      <td>56.5</td>
      <td>1.0</td>
      <td>43.5</td>
    </tr>
    <tr>
      <th>19</th>
      <td>Notes - Share capital, reserves and other equi...</td>
      <td>16</td>
      <td>32</td>
      <td>2.6</td>
      <td>1.2</td>
      <td>46.2</td>
      <td>1.4</td>
      <td>53.8</td>
    </tr>
    <tr>
      <th>20</th>
      <td>Notes - Intangible assets</td>
      <td>15</td>
      <td>31</td>
      <td>2.2</td>
      <td>1.2</td>
      <td>54.5</td>
      <td>1.0</td>
      <td>45.5</td>
    </tr>
    <tr>
      <th>21</th>
      <td>Notes - Accounting policies, changes in accoun...</td>
      <td>21</td>
      <td>28</td>
      <td>2.6</td>
      <td>1.6</td>
      <td>61.5</td>
      <td>1.0</td>
      <td>38.5</td>
    </tr>
    <tr>
      <th>22</th>
      <td>Notes - Operating segments</td>
      <td>11</td>
      <td>22</td>
      <td>2.3</td>
      <td>1.3</td>
      <td>56.5</td>
      <td>1.0</td>
      <td>43.5</td>
    </tr>
    <tr>
      <th>23</th>
      <td>Notes - Impairment of assets</td>
      <td>10</td>
      <td>18</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>50.0</td>
      <td>1.0</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>24</th>
      <td>Notes - Related party</td>
      <td>10</td>
      <td>16</td>
      <td>2.2</td>
      <td>1.2</td>
      <td>54.5</td>
      <td>1.0</td>
      <td>45.5</td>
    </tr>
    <tr>
      <th>25</th>
      <td>Notes - Other provisions, contingent liabiliti...</td>
      <td>8</td>
      <td>15</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>50.0</td>
      <td>1.0</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>26</th>
      <td>Notes - Property, plant and equipment</td>
      <td>8</td>
      <td>13</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>50.0</td>
      <td>1.0</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>27</th>
      <td>Notes - Share-based payment arrangements</td>
      <td>6</td>
      <td>11</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>50.0</td>
      <td>1.0</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>28</th>
      <td>Notes - Additional information</td>
      <td>6</td>
      <td>10</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>50.0</td>
      <td>1.0</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>30</th>
      <td>Notes - Insurance contracts (IFRS 17)</td>
      <td>8</td>
      <td>8</td>
      <td>3.2</td>
      <td>1.0</td>
      <td>31.2</td>
      <td>2.2</td>
      <td>68.8</td>
    </tr>
    <tr>
      <th>29</th>
      <td>Notes - Fair value measurement</td>
      <td>7</td>
      <td>8</td>
      <td>2.5</td>
      <td>1.5</td>
      <td>60.0</td>
      <td>1.0</td>
      <td>40.0</td>
    </tr>
    <tr>
      <th>31</th>
      <td>Notes - Interim financial reporting</td>
      <td>2</td>
      <td>6</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>32</th>
      <td>Notes - Analysis of other comprehensive income...</td>
      <td>1</td>
      <td>4</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>50.0</td>
      <td>1.0</td>
      <td>50.0</td>
    </tr>
    <tr>
      <th>33</th>
      <td>Notes - First time adoption</td>
      <td>2</td>
      <td>4</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>34</th>
      <td>Notes - Government grants</td>
      <td>1</td>
      <td>3</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>35</th>
      <td>Notes - Separate financial statements</td>
      <td>1</td>
      <td>3</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>100.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>36</th>
      <td>Notes - Borrowing costs</td>
      <td>2</td>
      <td>2</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>37</th>
      <td>Notes - Exploration for and evaluation of mine...</td>
      <td>1</td>
      <td>2</td>
      <td>1.0</td>
      <td>0.0</td>
      <td>0.0</td>
      <td>1.0</td>
      <td>100.0</td>
    </tr>
    <tr>
      <th>38</th>
      <td>Notes - Inventories</td>
      <td>1</td>
      <td>2</td>
      <td>1.0</td>
      <td>1.0</td>
      <td>100.0</td>
      <td>0.0</td>
      <td>0.0</td>
    </tr>
    <tr>
      <th>39</th>
      <td>Notes - Events after reporting period</td>
      <td>1</td>
      <td>1</td>
      <td>2.0</td>
      <td>1.0</td>
      <td>50.0</td>
      <td>1.0</td>
      <td>50.0</td>
    </tr>
  </tbody>
</table>
</div>



# ESEF-Berichterstattung in Deutschland: Herausforderungen und Chancen

## Der Python-Code zum Replizieren der Tabellen 1–3


```python
import pandas as pd
```



### Table 1

Tabelle 1 zeigt die ESEF-Konformitätsraten deut-scher kapitalmarktorientierter Unternehmen über den Beobachtungszeitraum von 2020 bis 2023. 


```python
tb1_uregdw_s5 = pd.read_pickle("../data/tb1_uregdw_s5_20250616.p.gz")
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
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
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
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
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
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
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
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
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
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
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
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
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



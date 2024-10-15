# Mini-Project: Economic and Quality of Life Factors Influencing the Life Ladder
_Mini-Project นี้ เป็นส่วนหนึ่งของวิชา DADS5001: Data Analytics and Data Science Tools and Programming_

### จัดทำโดย
1. 6610422034 นายกฤติพงษ์ หมอสุยะ
2. 6620422016 นางสาวธีรตา ศรีเขียวพงษ์
3. 6620422017 นายธณิศร พึ่งเจริญ

### Dataset
1. [HDI Index](https://hdr.undp.org/data-center/documentation-and-downloads) : 
Human Development Report
    * All composite indices and components time series (1990-2022)
2. [Life Ladder](https://worldhappiness.report/ed/2024/#appendices-and-data) : World Happiness Report
    * The survey measure of SWB (Subjective Well-being) is from the Gallup World Poll (GWP) 
3. [WDI](https://datatopics.worldbank.org/world-development-indicators/) : World Development Indicators
    * The database contains 1,400 time series indicators for 217 economies and more than 40 country groups,
      with data for many indicators going back more than 50 years.

## 1. Data Preparation
1.   Import the necessary libraries.
```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from matplotlib.ticker import FuncFormatter
import io
```
2.   Ingestion data by using pandas by using `pd.read_csv()` and `pd.read_excel()`.
    ```python
    #Example code 1: read_csv() with error handling
    with open('/content/HDI_2022.csv', 'r', encoding='utf-8', errors='ignore') as f:
        data = f.read()
    df_hdi = pd.read_csv(io.StringIO(data))
    #Example code 2: read_excel()
    happy_pt = '/content/LIFE_LADDER.xls'
    happy_df = pd.read_excel(happy_pt)
    ```
    - The datasets included in this project are `HDI_2022` and `LIFE_LADDER`, as defined at the beginning of this article.
    
3. Transform the data by unpivoting it from a `wide` format, where the indicators are concatenated with the year, into a `long` format. Extract the indicator name and year into two separate columns, named `Indicator Desc` and `Year`.
    - Original raw data
    ![image](https://imgur.com/lu8h5W9.png)
    - Unpivot data
    ![image](https://imgur.com/jjzzHp7.png)
4. Aggregate the data by averaging all the indicators of interest over the period from 2013 to 2022, or over a 10-year span.
   
```python
contain = ['le', 'eys', 'mys', 'gnipc', 'edu', 'inc', 'ihdi', 'coef_ineq']
pattern = '|'.join(contain)
melt_hdi['Year'] = melt_hdi['Year'].astype(np.int64)
hdi_filter = melt_hdi[(melt_hdi['Indicator Desc'].str.contains(pattern, case=False, regex=True)) & (melt_hdi['Year'] >= 2013) & (melt_hdi['Year'] <= 2022)]
cols = ['Country Code', 'HDI Code', 'Indicator Desc', 'Value']
hdi_agg = hdi_filter[cols].groupby(['Country Code', 'HDI Code', 'Indicator Desc']).mean().reset_index()
```
5. Consolidate the data using `pd.merge()` to join the HDI data with the LIFE_LADDER data, and assign more understandable alias names.
```python
consolidate_all = pd.merge(df_consl_agg,
         hdi_pivot,
         left_on='Country Code',
         right_on='Country Code',
         how='left')
select_col = ['Country Code', 'Country Name', 'HDI Code_x', 'Income Group', 'Region',
       'Life Ladder', 'Social support', 'Freedom to make life choices',
       'Generosity', 'Perceptions of corruption', 'Positive affect',
       'Negative affect', 'Life Ladder Rank', 'eys','gnipc',
              'ineq_edu', 'ineq_inc', 'ineq_le', 'le', 'mys', 'ihdi', 'coef_ineq']
consolidate_all_alias = consolidate_all[select_col]
#assign more understandable alias names
consolidate_all_alias.columns = ['Country Code', 'Country Name', 'HDI Code', 'Income Group', 'Region',
       'Life Ladder', 'Social support', 'Freedom to make life choices',
       'Generosity', 'Perceptions of corruption', 'Positive affect',
       'Negative affect', 'Life Ladder Rank', 'Expected Years of Schooling','GNI per capita',
        'Inequality in education', 'Inequality in income', 'Inequality in life expectancy',
        'life expectancy at birth', 'Mean Years of Schooling', 'Inequality-adjusted Human Development Index', 'Coefficient of human inequality']
```

## 2. Objective


## 3. Exploratory Data Analysis (EDA)
### 3.1 Introduction

จากการวิเคราะห์ข้อมูลตามภูมิภาคที่ครอบคลุมหลากหลายมิติ ทั้ง Life Ladder, Gross National Income per capita, Perceptions of Corruption, Inequality in Income, Mean Years of Schooling และ Inequality in Education ทำให้เห็นความแตกต่างอย่างชัดเจนระหว่างภูมิภาคที่มีการพัฒนาสูงกับภูมิภาคที่ยังคงเผชิญกับความท้าทายทางเศรษฐกิจและสังคม _(ภาพที่ 1)_

![image](https://i.imgur.com/bnKJjgH.png)
_(ภาพที่ 1: กราฟแสดงข้อมูลของปัจจัยที่ส่งผลต่อความพึงพอใจในชีวิตโดยแบ่งตามภูมิภาค)_


* **ภูมิภาคที่มีรายได้เฉลี่ยสูง มักจะมีความพึงพอใจในชีวิตสูง**
   * North America (7.09) ซึ่งมีรายได้เฉลี่ยต่อหัว (GNI per capita) สูงสุดที่ 43,531.76 USD แสดงถึงความพึงพอใจในชีวิตที่สูง การมีรายได้ต่อหัวที่สูงมีส่วนช่วยให้ประชาชนสามารถเข้าถึงโอกาสทางเศรษฐกิจ สังคม และการพัฒนาอื่นๆ ได้อย่างเต็มที่ ซึ่งส่งผลให้พวกเขามีความพึงพอใจในชีวิตมากขึ้น
   * Sub-Saharan Africa (2,315.01 USD) และ South Asia (5,596.83 USD) มี GNI per capita ต่ำที่สุด ส่งผลให้ประชาชนในภูมิภาคเหล่านี้มี Life Ladder ต่ำเช่นกัน (4.27 และ 4.79) สะท้อนถึงข้อจำกัดในด้านรายได้และโอกาสที่ส่งผลต่อคุณภาพชีวิตที่ต่ำ
* **การกระจายรายได้ที่เท่าเทียมมีส่วนในการเพิ่มความพึงพอใจในชีวิต**
   * Sub-Saharan Africa มีความไม่เท่าเทียมในรายได้สูงสุด (44.62%) ซึ่งส่งผลทำให้ประชาชนในภูมิภาคนี้มีความพึงพอใจในชีวิตต่ำที่สุด เนื่องจากการกระจายรายได้ที่ไม่เท่าเทียมทำให้กลุ่มคนจนไม่ได้รับโอกาสเท่าเทียมกันในการพัฒนาคุณภาพชีวิต
   * ในทางกลับกัน Europe & Central Asia มีความไม่เท่าเทียมในรายได้ต่ำที่สุด (16.65%) ส่งผลให้มี Life Ladder สูง (5.98) การกระจายรายได้ที่เท่าเทียมช่วยให้ประชาชนรู้สึกว่าได้รับโอกาสที่เหมาะสมในการพัฒนาคุณภาพชีวิต ซึ่งทำให้พวกเขาพึงพอใจในชีวิตมากขึ้น
* **การศึกษาเป็นปัจจัยสำคัญต่อคุณภาพชีวิต**
   * ภูมิภาคที่มีค่าเฉลี่ยของจำนวนปีที่ประชาชนได้รับการศึกษา (Mean Years of Schooling) สูง เช่น North America (13.60 ปี) และ Europe & Central Asia (11.89 ปี) มีความพึงพอใจในชีวิตที่สูง แสดงให้เห็นว่าการศึกษาช่วยเพิ่มทักษะและโอกาสในการทำงาน ซึ่งส่งผลต่อความเป็นอยู่ที่ดีขึ้นของประชาชน
   * ในขณะเดียวกัน ภูมิภาคที่มีค่าเฉลี่ยการศึกษาต่ำ เช่น South Asia (5.44 ปี) และ Sub-Saharan Africa (5.08 ปี) มีความพึงพอใจในชีวิตต่ำเช่นกัน เนื่องจากการเข้าถึงการศึกษาที่จำกัดทำให้ประชาชนขาดโอกาสในการพัฒนาตนเองและชีวิตในด้านต่างๆ
* **การรับรู้ถึงการคอร์รัปชันมีผลกระทบต่อความเชื่อมั่นและความพึงพอใจในชีวิต**
   * North America มี Perceptions of Corruption ต่ำที่สุด (0.56) ซึ่งแสดงถึงความไว้วางใจในรัฐบาลและระบบการบริหารที่มีความโปร่งใสและเสถียร ความโปร่งใสของรัฐบาลช่วยสร้างความมั่นใจในระบบสาธารณะ ซึ่งส่งผลให้ประชาชนมีความพึงพอใจในชีวิตมากขึ้น
   * ในทางตรงกันข้าม South Asia และ Sub-Saharan Africa ที่มีการรับรู้ถึงคอร์รัปชันสูง (0.79) มีผลต่อความพึงพอใจในชีวิตที่ต่ำ เพราะการรับรู้ว่าระบบการบริหารมีความทุจริตทำให้ประชาชนขาดความเชื่อมั่นในรัฐบาลและระบบสังคม
* **ความไม่เท่าเทียมทางการศึกษาส่งผลต่อความไม่เท่าเทียมทางรายได้**
   * South Asia (40.10%) และ Sub-Saharan Africa (35.09%) มีความไม่เท่าเทียมในการศึกษาในระดับสูง การขาดโอกาสในการเข้าถึงการศึกษาที่เท่าเทียมทำให้ประชากรขาดทักษะและความรู้ ซึ่งส่งผลโดยตรงต่อโอกาสในการทำงานและรายได้ในอนาคต ทำให้ความไม่เท่าเทียมทางรายได้สูงขึ้นตามมา
   * ในทางกลับกัน Europe & Central Asia (3.68%) มีความไม่เท่าเทียมทางการศึกษาต่ำที่สุด ซึ่งสะท้อนให้เห็นว่าประชาชนในภูมิภาคนี้ได้รับโอกาสทางการศึกษาในระดับที่เท่าเทียมกันมากกว่า และส่งผลให้รายได้มีการกระจายอย่างเท่าเทียมมากขึ้นด้วย


**คุณภาพชีวิตที่ดี (HDI) ส่งผลโดยตรงต่อการเพิ่มความพึงพอใจในชีวิต (Life Ladder)**
![image](https://i.imgur.com/cAp7Myh.png)
_(ภาพที่ 2: กราฟแสดงข้อมูลของปัจจัยที่ส่งผลต่อความพึงพอใจในชีวิตโดยแบ่งตาม HDI Group)_

   * การพัฒนาคุณภาพชีวิตที่เพิ่มขึ้นซึ่งวัดจากรายได้ต่อหัว (GNI per capita), การศึกษา (Mean Years of Schooling), และอายุขัยเฉลี่ย แสดงให้เห็นว่าประเทศที่มีการพัฒนามนุษย์ในทุกมิติมีแนวโน้มทีความพึงพอใจในชีวิตจะเพิ่มขึ้นเช่นกัน
   * ประเทศที่อยู่ในกลุ่มคุณภาพชีวิตแบบ Very High จะมี ความพึงพอใจในชีวิตสูงที่สุด สะท้อนถึงคุณภาพชีวิตและความพึงพอใจที่ดีขึ้นเมื่อประชากรมีโอกาสทางการศึกษาและรายได้ที่เพิ่มขึ้น
   * กลุ่มที่มีคุณภาพชีวิตแบบ Low เช่น Sub-Saharan Africa มีความพึงพอใจต่ำ แสดงถึงข้อจำกัดในการเข้าถึงการศึกษาและทรัพยากรที่ทำให้ประชาชนมีความพึงพอใจในชีวิตต่ำลง

แต่การมีคุณภาพชีวิตที่ดีไม่ได้หมายความว่าปัญหาทุจริตและคอร์รัปชันจะหมดไป
   * แม้ประเทศที่มีคุณภาพชีวิตที่สูง เช่น Very High จะมีคุณภาพชีวิตที่ดี แต่การรับรู้ถึงคอร์รัปชัน (Perceptions of Corruption) ยังคงสูงในบางกลุ่ม ซึ่งแสดงถึงความท้าทายในการลดปัญหาการทุจริต แม้คุณภาพชีวิตจะดีขึ้น
การลดความไม่เท่าเทียมในมิติต่างๆ จะช่วยเพิ่มความพึงพอใจในชีวิต
   * การลดความไม่เท่าเทียมทางการศึกษาและรายได้ในประเทศที่มีคุณภาพต่ำ จะช่วยให้ประชาชนมีโอกาสที่เท่าเทียมกันมากขึ้น และส่งผลให้ความพึงพอใจสูงขึ้นตามไปด้วย
   * การพัฒนาคุณภาพชีวิตผ่านการเพิ่มโอกาสทางการศึกษา รายได้ และการลดความไม่เท่าเทียม เป็นปัจจัยสำคัญที่ส่งผลต่อการเพิ่มความพึงพอใจในชีวิต (Life Ladder) อย่างไรก็ตาม การลดปัญหาคอร์รัปชันยังคงเป็นสิ่งสำคัญแม้ว่าคุณภาพชีวิตในด้านอื่นๆ จะดีขึ้นแล้วก็ตาม

### 3.2 Body

จากข้อมูลข้างต้น พบว่าปัจจัยในการวัดความสุข คือ **ความพึงพอใจในชีวิต (Life Ladder)** และ **ดัชนีคุณภาพชีวิตที่ดี (Human Development Index)**
ซึ่งได้ทำการนำตัวชี้วัด (Indicators) ของทั้ง 2 ปัจจัยมาหาความสัมพันธ์ ผ่าน Correlation Matrix _(ภาพที่ 3)_

![image](https://i.imgur.com/U4iIAq7.png)
_(ภาพที่ 3: กราฟแสดงข้อมูลความสัมพันธ์ระหว่างตัวแปรของคุณภาพชีวิต (Human Development Index - HDI) และความสุข (Life Ladder))_

**การตีความ Correlation Matrix:**
   * ค่าสีแดง: แสดงถึงความสัมพันธ์เชิงบวกที่สูง (ค่าความสัมพันธ์ใกล้เคียง 1) หมายความว่าตัวชี้วัดทั้งสองเปลี่ยนแปลงในทิศทางเดียวกัน
   * ค่าสีน้ำเงิน: แสดงถึงความสัมพันธ์เชิงลบ (ค่าความสัมพันธ์ใกล้เคียง -1) หมายถึงตัวชี้วัดทั้งสองเปลี่ยนแปลงในทิศทางตรงข้ามกัน
   * ค่าใกล้ 0: แสดงว่าความสัมพันธ์ระหว่างตัวชี้วัดทั้งสองไม่ชัดเจนหรือไม่มีความสัมพันธ์กัน

จะเห็นได้ว่าตัวชี้วัดของทั้ง 2 ปัจจัย มีทั้งความสัมพันธ์เชิงบวกและเชิงลบ จึงขอหยิบคู่ความสัมพันธ์ที่น่าสนใจมานำเสนอเพิ่มเติม ดังนี้:

**ตัวแปรที่ส่งผลต่อระดับความเครียด (Negative Affect) และแนวทางในการลดปัจจัยเพื่อส่งเสริมสุขภาพจิตและความสุขในสังคม**

  **การศึกษาและทำความเข้าใจเกี่ยวกับตัวแปรที่ส่งผลต่อระดับความเครียดในสังคมถือเป็นก้าวสำคัญในการพัฒนาแนวทางที่ช่วยส่งเสริมสุขภาพจิตและความเป็นอยู่ที่ดี เราสามารถใช้ข้อมูลเหล่านี้ในการออกแบบนโยบายที่ส่งเสริมความสมดุลและความสุขในสังคมได้อย่างยั่งยืน**

  
#### 5.ความไม่เท่าเทียมทางการศึกษา (Inequality in education) VS ระดับความเครียด (Negative affect)

 ![img](https://i.imgur.com/f5slrTN.png)

_( ภาพที่  : กราฟแสดงความสัมพันธ์ระหว่างความไม่เท่าเทียมในด้านการศึกษา(Inequality in education) และ ระดับความเครียด (Negative affect) )_


- ความไม่เท่าเทียมทางการศึกษามีความสัมพันธ์เชิงบวกกับระดับความเครียด ซึ่งหมายความว่าการลดช่องว่างในการเข้าถึงการศึกษาจะช่วยลดความเครียดได้ ข้อมูลนี้ชี้ให้เห็นถึงความสำคัญของการส่งเสริมการศึกษาที่เท่าเทียมสำหรับทุกคน โดยเฉพาะในกลุ่มประชากรที่ขาดโอกาส
  
#### 6.รายได้ประชาชาติต่อหัว (Gross National Income per capita) VS ระดับความเครียด ( Negative affect)**

  ![img](https://i.imgur.com/8mqFNYz.png)

  _( ภาพที่   : กราฟแสดงความสัมพันธ์ระหว่างรายได้ประชาชาติต่อหัว (GNI per capita) และ ระดับความเครียด (Negative affect) )_

  - รายได้ประชาชาติต่อหัวที่สูงขึ้นมีความสัมพันธ์กับระดับความเครียดที่ลดลง แสดงให้เห็นว่าความมั่นคงทางเศรษฐกิจมีบทบาทสำคัญในการลดความเครียด การพัฒนาเศรษฐกิจและสร้างโอกาสทางการเงินที่มั่นคงจึงเป็นแนวทางที่ดีในการสร้างความสุขให้แก่ประชากร
   
#### 7.ความไม่เท่าเทียมในอายุขัย (Inequality in life expectancy) VS ระดับความเครียด ( Negative affect)**

  ![img](https://i.imgur.com/bvP8IrS.png)

  _( ภาพที่   : กราฟแสดงความสัมพันธ์ระหว่างความไม่เท่าเทียมในอายุขัย (Inequality in life expectancy) และ ระดับความเครียด (Negative affect) )_

  - ความไม่เท่าเทียมในอายุขัยสะท้อนถึงการเข้าถึงบริการสุขภาพที่ไม่เท่าเทียม ซึ่งสัมพันธ์กับระดับความเครียดที่สูงขึ้น การพัฒนาระบบสุขภาพที่มีการเข้าถึงอย่างเท่าเทียมสามารถช่วยลดความเครียดและสร้างความพึงพอใจในชีวิตให้กับประชากรได้อย่างยั่งยืน


#### 4.อายุขัยเฉลี่ย (life expectancy at birth) VS ระดับความเครียด ( Negative affect)**

  ![img](https://i.imgur.com/k8syEhr.png)

  _( ภาพที่   : กราฟแสดงความสัมพันธ์ระหว่างอายุขัยเฉลี่ย (life expectancy at birth) และ ระดับความเครียด (Negative affect) )_


  - อายุขัยเฉลี่ยที่สูงขึ้นมีความสัมพันธ์เชิงลบกับระดับความเครียด นั่นคือเมื่อประชากรมีชีวิตยืนยาวขึ้น พวกเขามีแนวโน้มที่จะมีความเครียดน้อยลง การส่งเสริมสุขภาพและยกระดับคุณภาพชีวิตจึงเป็นแนวทางที่ยั่งยืนในการลดปัญหาทางอารมณ์และความเครียด


สรุป:
- การส่งเสริมการศึกษาที่เท่าเทียม การพัฒนาเศรษฐกิจที่มั่นคง และการสร้างระบบสุขภาพที่ครอบคลุม ล้วนเป็นปัจจัยสำคัญที่ช่วยลดความเครียดในสังคมได้อย่างมีประสิทธิภาพ การผสมผสานปัจจัยเหล่านี้ในการพัฒนาสังคมจะช่วยให้เห็นแนวทางที่ครอบคลุมและยั่งยืนมากขึ้น ซึ่งจะนำไปสู่สังคมที่มีความสุขและสมดุลมากขึ้น


### 3.3 Conclusion
  


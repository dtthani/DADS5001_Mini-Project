# Mini-Project: Economic and Quality of Life Factors Influencing the Life Ladder
_Mini-Project นี้เป็นส่วนหนึ่งของวิชา DADS5001: Data Analytics and Data Science Tools and Programming โดยมีจุดมุ่งหมายในการสำรวจความสัมพันธ์ระหว่างปัจจัยทางเศรษฐกิจและคุณภาพชีวิตกับระดับความสุขตามตัวชี้วัด Life Ladder เพื่อทำความเข้าใจว่าปัจจัยเหล่านี้ส่งผลต่อความพึงพอใจในชีวิตของผู้คนอย่างไร ซึ่งจะเป็นประโยชน์ต่อการวางนโยบายและการพัฒนาคุณภาพชีวิตในระดับประเทศ_

**จัดทำโดย**
1. 6610422034 นายกฤติพงษ์ หมอสุยะ
2. 6620422016 นางสาวธีรตา ศรีเขียวพงษ์
3. 6620422017 นายธณิศร พึ่งเจริญ

### Dataset
1. [HDI Index](https://hdr.undp.org/data-center/documentation-and-downloads) : 
Human Development Report
    * รวมดัชนี (Index) และองค์ประกอบต่าง ๆ ในรูปแบบชุดข้อมูลตามช่วงเวลา (1990-2022)
2. [Life Ladder](https://worldhappiness.report/ed/2024/#appendices-and-data) : World Happiness Report
    * การสำรวจเพื่อวัดความเป็นอยู่ที่พึงพอใจ (Subjective Well-being) มาจากการสำรวจ Gallup World Poll (GWP)
3. [WDI](https://datatopics.worldbank.org/world-development-indicators/) : World Development Indicators
    * ข้อมูลประกอบด้วยตัวชี้วัดตามช่วงเวลาจำนวน 1,400 ตัว สำหรับ 217 เศรษฐกิจและกลุ่มประเทศกว่า 40 กลุ่ม โดยข้อมูลของหลายตัวชี้วัดมีความเป็นไปได้ย้อนกลับไปมากกว่า 50 ปี

## 1. Data Preparation
1.   นำเข้า libraries ที่จำเป็น
```python
import pandas as pd
import numpy as np
import seaborn as sns
import matplotlib.pyplot as plt
from matplotlib.ticker import FuncFormatter
import io
```
2.   การนำเข้าข้อมูลโดยใช้ library pandas ผ่านฟังก์ชัน `pd.read_csv()` และ `pd.read_excel()`
```python
    #Example code 1: read_csv() with error handling
    with open('/content/HDI_2022.csv', 'r', encoding='utf-8', errors='ignore') as f:
        data = f.read()
    df_hdi = pd.read_csv(io.StringIO(data))
    #Example code 2: read_excel()
    happy_pt = '/content/LIFE_LADDER.xls'
    happy_df = pd.read_excel(happy_pt)
```
* Dataset ที่รวมอยู่ใน Mini-project นี้คือ "HDI_2022" และ "LIFE_LADDER" ตามที่ระบุไว้
    
3. เปลี่ยนรูปแบบข้อมูลโดยการแปลงรูปแบบ (Unpivot) จาก `wide` ซึ่งข้อมูลตัวชี้วัดถูกจัดเรียงรวมกับปี เป็นรูปแบบ `long`  โดยการแยกชื่อข้อมูลตัวชี้วัดและปี ออกเป็นสองคอลัมน์ โดยตั้งชื่อว่า `Indicator Desc` และ `Year`
   
    - Original raw data
    ![image](https://imgur.com/lu8h5W9.png)
    - Unpivot data
    ![image](https://imgur.com/jjzzHp7.png)

4. ทำการรวมข้อมูลโดยการคำนวณค่าเฉลี่ยของข้อมูลตัวชี้วัดที่สนใจทั้งหมดในช่วงเวลา 10 ปี ตั้งแต่ปี 2013 ถึง 2022
   
```python
contain = ['le', 'eys', 'mys', 'gnipc', 'edu', 'inc', 'ihdi', 'coef_ineq']
pattern = '|'.join(contain)
melt_hdi['Year'] = melt_hdi['Year'].astype(np.int64)
hdi_filter = melt_hdi[(melt_hdi['Indicator Desc'].str.contains(pattern, case=False, regex=True)) & (melt_hdi['Year'] >= 2013) & (melt_hdi['Year'] <= 2022)]
cols = ['Country Code', 'HDI Code', 'Indicator Desc', 'Value']
hdi_agg = hdi_filter[cols].groupby(['Country Code', 'HDI Code', 'Indicator Desc']).mean().reset_index()
```
5. รวมข้อมูลโดยใช้ฟังก์ชัน `pd.merge()` เพื่อเชื่อมต่อข้อมูล HDI กับข้อมูล LIFE_LADDER และตั้งชื่อใหม่ที่เข้าใจง่ายมากขึ้น
   
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
**ปัจจัยทางเศรษฐกิจและคุณภาพชีวิตที่ดีส่งผลต่อของประเทศไทยด้านความพึงพอใจในชีวิตอย่างไร**

## 3. Exploratory Data Analysis (EDA)
### 3.1 Introduction
ในปัจจุบัน ความสุขและคุณภาพชีวิตเป็นประเด็นที่ได้รับความสนใจอย่างมากในการศึกษาทางเศรษฐศาสตร์และสังคมศาสตร์ หนึ่งในตัวชี้วัดที่ได้รับการยอมรับในการวัดความสุขของประชาชน คือ ความพึงพอใจในชีวิต (Life Ladder) ซึ่งสะท้อนถึงความพึงพอใจในชีวิตของผู้คนในแต่ละประเทศ ในขณะเดียวกัน ปัจจัยทางเศรษฐกิจและคุณภาพชีวิต เช่น ผลิตภัณฑ์มวลรวมภายในประเทศ (GDP) รายได้ต่อหัว และดัชนีการพัฒนามนุษย์ (Human Develooment Index: HDI) ก็มีบทบาทสำคัญในการกำหนดความเป็นอยู่ที่ดีของประชากรเช่นกัน


![image](https://i.imgur.com/bnKJjgH.png)
_(ภาพที่ 1: กราฟแสดงการเปรียบเทียบปัจจัยต่างๆ ที่ส่งผลต่อความพึงพอใจในชีวิต (Life Ladder) ของประชากรในภูมิภาคต่างๆ)_

จาก (ภาพที่ 1) สามารถวิเคราะห์ข้อมูลออกมาได้ ดังนี้:

* **ภูมิภาคที่มีรายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita) สูง มักจะมีความพึงพอใจในชีวิตสูง**
   * North America (7.09) ซึ่งมีรายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita) สูงสุดที่ 43,531.76 USD แสดงถึงความพึงพอใจในชีวิตที่สูง การมีรายได้ต่อหัวที่สูงมีส่วนช่วยให้ประชาชนสามารถเข้าถึงโอกาสทางเศรษฐกิจ สังคม และการพัฒนาอื่นๆ ได้อย่างเต็มที่ ซึ่งส่งผลให้พวกเขามีความพึงพอใจในชีวิตมากขึ้น
   * Sub-Saharan Africa (2,315.01 USD) และ South Asia (5,596.83 USD) มีรายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita) ต่ำที่สุด ส่งผลให้ประชาชนในภูมิภาคเหล่านี้มีความพึงพอใจในชีวิตต่ำเช่นกัน (4.27 และ 4.79) สะท้อนถึงข้อจำกัดในด้านรายได้และโอกาสที่ส่งผลต่อคุณภาพชีวิตที่ต่ำ
* **การกระจายรายได้ที่เท่าเทียมมีส่วนในการเพิ่มความพึงพอใจในชีวิต**
   * Sub-Saharan Africa มีความไม่เท่าเทียมในรายได้สูงสุด (44.62%) ซึ่งส่งผลทำให้ประชาชนในภูมิภาคนี้มีความพึงพอใจในชีวิตต่ำที่สุด เนื่องจากการกระจายรายได้ที่ไม่เท่าเทียมทำให้กลุ่มคนจนไม่ได้รับโอกาสเท่าเทียมกันในการพัฒนาคุณภาพชีวิต
   * ในทางกลับกัน Europe & Central Asia มีความไม่เท่าเทียมในรายได้ต่ำที่สุด (16.65%) ส่งผลให้มี Life Ladder สูง (5.98) การกระจายรายได้ที่เท่าเทียมช่วยให้ประชาชนรู้สึกว่าได้รับโอกาสที่เหมาะสมในการพัฒนาคุณภาพชีวิต ซึ่งทำให้พวกเขาพึงพอใจในชีวิตมากขึ้น
* **การศึกษาเป็นปัจจัยสำคัญต่อคุณภาพชีวิต**
   * ภูมิภาคที่มีปีการศึกษาเฉลี่ยของประชากร (Mean Years of Schooling) สูง เช่น North America (13.60 ปี) และ Europe & Central Asia (11.89 ปี) มีความพึงพอใจในชีวิตที่สูง แสดงให้เห็นว่าการศึกษาช่วยเพิ่มทักษะและโอกาสในการทำงาน ซึ่งส่งผลต่อความเป็นอยู่ที่ดีขึ้นของประชาชน
   * ในขณะเดียวกัน ภูมิภาคที่มีค่าเฉลี่ยปีการศึกษาเฉลี่ยของประชากร (Mean Years of Schooling) ต่ำ เช่น South Asia (5.44 ปี) และ Sub-Saharan Africa (5.08 ปี) มีความพึงพอใจในชีวิตต่ำเช่นกัน เนื่องจากการเข้าถึงการศึกษาที่จำกัดทำให้ประชาชนขาดโอกาสในการพัฒนาตนเองและชีวิตในด้านต่างๆ
* **การรับรู้ถึงการคอร์รัปชันมีผลกระทบต่อความเชื่อมั่นและความพึงพอใจในชีวิต**
   * North America มีการรับรู้ถึงการคอร์รัปชัน (Perceptions of corruption) ต่ำที่สุด (0.56) ซึ่งแสดงถึงความไว้วางใจในรัฐบาลและระบบการบริหารที่มีความโปร่งใสและเสถียร ความโปร่งใสของรัฐบาลช่วยสร้างความมั่นใจในระบบสาธารณะ ซึ่งส่งผลให้ประชาชนมีความพึงพอใจในชีวิตมากขึ้น
   * ในทางตรงกันข้าม South Asia และ Sub-Saharan Africa ที่มีการรับรู้ถึงการคอร์รัปชัน (Perceptions of corruption) สูง (0.79) มีผลต่อความพึงพอใจในชีวิตที่ต่ำ เพราะการรับรู้ว่าระบบการบริหารมีความทุจริตทำให้ประชาชนขาดความเชื่อมั่นในรัฐบาลและระบบสังคม
* **ความไม่เท่าเทียมทางการศึกษาส่งผลต่อความไม่เท่าเทียมทางรายได้**
   * South Asia (40.10%) และ Sub-Saharan Africa (35.09%) มีความเหลื่อมล้ำทางการศึกษา (Inequality in Education)ในระดับสูง การขาดโอกาสในการเข้าถึงการศึกษาที่เท่าเทียมทำให้ประชากรขาดทักษะและความรู้ ซึ่งส่งผลโดยตรงต่อโอกาสในการทำงานและรายได้ในอนาคต ทำให้ความไม่เท่าเทียมทางรายได้สูงขึ้นตามมา
   * ในทางกลับกัน Europe & Central Asia (3.68%) มีความเหลื่อมล้ำทางการศึกษา (Inequality in Education)ต่ำที่สุด ซึ่งสะท้อนให้เห็นว่าประชาชนในภูมิภาคนี้ได้รับโอกาสทางการศึกษาในระดับที่เท่าเทียมกันมากกว่า และส่งผลให้รายได้มีการกระจายอย่างเท่าเทียมมากขึ้นด้วย


**คุณภาพชีวิตที่ดี (Human Development Index: HDI) ส่งผลโดยตรงต่อการเพิ่มความพึงพอใจในชีวิต (Life Ladder)**

![image](https://i.imgur.com/cAp7Myh.png)
_(ภาพที่ 2: กราฟแสดงข้อมูลของปัจจัยที่ส่งผลต่อความพึงพอใจในชีวิตโดยแบ่งตาม HDI Group)_

   * จาก (ภาพที่ 2)การพัฒนาคุณภาพชีวิตที่เพิ่มขึ้นซึ่งวัดจากรายได้ต่อหัว (Gross National Income per capita), การศึกษา (Mean Years of Schooling), และอายุขัยเฉลี่ย แสดงให้เห็นว่าประเทศที่มีการพัฒนามนุษย์ในทุกมิติมีแนวโน้มทีความพึงพอใจในชีวิตจะเพิ่มขึ้นเช่นกัน
   * ประเทศที่อยู่ในกลุ่มคุณภาพชีวิตแบบ Very High จะมีความพึงพอใจในชีวิตสูงที่สุด สะท้อนถึงคุณภาพชีวิตและความพึงพอใจที่ดีขึ้นเมื่อประชากรมีโอกาสทางการศึกษาและรายได้ที่เพิ่มขึ้น
   * กลุ่มที่มีคุณภาพชีวิตแบบ Low เช่น ภูมิภาค Sub-Saharan Africa มีความพึงพอใจต่ำ แสดงถึงข้อจำกัดในการเข้าถึงการศึกษาและทรัพยากร ทำให้ประชาชนมีความพึงพอใจในชีวิตต่ำลง

แต่การมีคุณภาพชีวิตที่ดีไม่ได้หมายความว่าปัญหาทุจริตและคอร์รัปชันจะหมดไป
   * แม้ประเทศที่มีคุณภาพชีวิตที่สูง เช่น Very High จะมีคุณภาพชีวิตที่ดี แต่การรับรู้ถึงคอร์รัปชัน (Perceptions of Corruption) ยังคงสูงในบางกลุ่ม ซึ่งแสดงถึงความท้าทายในการลดปัญหาการทุจริต แม้คุณภาพชีวิตจะดีขึ้น
     
การลดความไม่เท่าเทียมในมิติต่างๆ จะช่วยเพิ่มความพึงพอใจในชีวิต
   * การลดความไม่เท่าเทียมทางการศึกษาและรายได้ในประเทศที่มีคุณภาพต่ำ จะช่วยให้ประชาชนมีโอกาสที่เท่าเทียมกันมากขึ้น และส่งผลให้ความพึงพอใจสูงขึ้นตามไปด้วย
   * การพัฒนาคุณภาพชีวิตผ่านการเพิ่มโอกาสทางการศึกษา รายได้ และการลดความไม่เท่าเทียม เป็นปัจจัยสำคัญที่ส่งผลต่อการเพิ่มความพึงพอใจในชีวิต (Life Ladder) อย่างไรก็ตาม การลดปัญหาคอร์รัปชันยังคงเป็นสิ่งสำคัญแม้ว่าคุณภาพชีวิตในด้านอื่นๆ จะดีขึ้นแล้วก็ตาม

### 3.2 Indicators Analysis

* จากข้อมูลข้างต้น พบว่าปัจจัยสำคัญในการวัดความสุขของประชากร คือ **ความพึงพอใจในชีวิต (Life Ladder)** และ **ดัชนีคุณภาพชีวิตที่ดี (Human Development Index: HDI)** ซึ่งทั้งสองปัจจัยนี้เป็นตัวชี้วัดที่ครอบคลุมด้านต่างๆ ของชีวิต โดยแต่ละปัจจัยประกอบไปด้วย ตัวชี้วัด (Indicators) ที่หลากหลาย ตัวอย่างเช่น ความพึงพอใจในชีวิตมักวัดจากระดับความสุขและความพึงพอใจส่วนบุคคล ในขณะที่ดัชนีคุณภาพชีวิตที่ดีวัดจากปัจจัยต่างๆ เช่น การศึกษา สุขภาพ รายได้ และความเท่าเทียม
* เพื่อทำความเข้าใจว่าตัวชี้วัดของทั้ง Life Ladder และ HDI มีความสัมพันธ์กันอย่างไร และส่งผลต่อความสุขของประชากรในแต่ละประเทศมากน้อยเพียงใด จึงมีการนำตัวชี้วัดเหล่านี้มาวิเคราะห์ผ่านเครื่องมือที่เรียกว่า **Correlation Matrix** เครื่องมือนี้จะช่วยให้เห็นความสัมพันธ์ระหว่างตัวแปรต่างๆ ว่ามีทิศทางเป็นบวกหรือลบ และมีความสัมพันธ์กันมากน้อยแค่ไหน _(ภาพที่ 3)_

![image](https://i.imgur.com/U4iIAq7.png)
_(ภาพที่ 3: กราฟแสดงข้อมูลความสัมพันธ์ระหว่างตัวแปรของคุณภาพชีวิต (Human Development Index - HDI) และความสุข (Life Ladder))_

**การตีความ Correlation Matrix:**
   * ค่าสีแดง: แสดงถึงความสัมพันธ์เชิงบวกที่สูง (ค่าความสัมพันธ์ใกล้เคียง 1) หมายความว่าตัวชี้วัดทั้งสองเปลี่ยนแปลงในทิศทางเดียวกัน
   * ค่าสีน้ำเงิน: แสดงถึงความสัมพันธ์เชิงลบ (ค่าความสัมพันธ์ใกล้เคียง -1) หมายถึงตัวชี้วัดทั้งสองเปลี่ยนแปลงในทิศทางตรงข้ามกัน
   * ค่าใกล้ 0: แสดงว่าความสัมพันธ์ระหว่างตัวชี้วัดทั้งสองไม่ชัดเจนหรือไม่มีความสัมพันธ์กัน

จะเห็นได้ว่าตัวชี้วัดของทั้ง 2 ปัจจัย มีทั้งความสัมพันธ์เชิงบวกและเชิงลบ จึงขอหยิบคู่ความสัมพันธ์ที่น่าสนใจมานำเสนอเพิ่มเติม ดังนี้:

#### 1. การสนับสนุนทางสังคม (Social Support) และ รายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita) 
* รายได้ประชาชาติต่อหัวที่สูงขึ้นมีบทบาทสำคัญในการเพิ่มระดับการสนับสนุนทางสังคมและความสุขของประชากร (ภาพที่ 4) เนื่องจากประเทศที่มีรายได้สูงมักมีโครงสร้างสังคมที่ดี ส่งผลให้ประชากรมีความสุขมากขึ้น ขณะที่ประเทศที่มีรายได้ต่ำต้องเผชิญกับปัญหาทางเศรษฐกิจและสังคมซึ่งลดทอนคุณภาพชีวิตและความสุขของประชากร
 ![img](https://i.imgur.com/sqv13x2.png)

_(ภาพที่ 4: กราฟแสดงความสัมพันธ์ระหว่างการสนับสนุนทางสังคม (Social support) และ รายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita))_

#### 2. การสนับสนุนทางสังคม (Social Support) และ อายุขัยเฉลี่ย (Life Expectancy at Birth)
* อายุขัยเฉลี่ยที่สูงสะท้อนถึงคุณภาพชีวิตที่ดีขึ้น และยังส่งผลให้การสนับสนุนทางสังคมแข็งแกร่งขึ้น (ภาพที่ 5) ซึ่งมีผลเชิงบวกต่อความสุขของประชาชน ประเทศที่มีอายุขัยเฉลี่ยสูงมักมีระบบสังคมที่มั่นคงและทำให้ผู้คนมีความสุขมากขึ้น ขณะที่ประเทศที่มีอายุขัยต่ำต้องเผชิญกับปัญหาทางสุขภาพและสังคมที่ลดคุณภาพชีวิตและความสุขโดยรวม
 ![img](https://i.imgur.com/HBrgOAw.png)

_(ภาพที่ 5: กราฟแสดงความสัมพันธ์ระหว่างการสนับสนุนทางสังคม (Social Support) และ อายุขัยเฉลี่ย (Life Expectancy at Birth))_
  
#### 3. การสนับสนุนทางสังคม (Social Support) และ ปีการศึกษาเฉลี่ยของประชากร (Mean Years of Schooling) 
* การศึกษามีบทบาทสำคัญในการส่งเสริมการสนับสนุนทางสังคม การมีปีการศึกษาเฉลี่ยที่สูงช่วยเสริมสร้างความเข้าใจและการสนับสนุนในสังคม (ภาพที่ 6) ซึ่งนำไปสู่ความสุขของประชากรที่มากขึ้น ประเทศที่มีระดับการศึกษาต่ำมักเผชิญกับปัญหาการสนับสนุนทางสังคมและคุณภาพชีวิตที่ลดลง ทำให้ความสุขโดยรวมของประชาชนต่ำกว่าประเทศที่มีการศึกษาดี
![img](https://i.imgur.com/YFfYpVZ.png)

_(ภาพที่ 6: กราฟแสดงความสัมพันธ์ระหว่างการสนับสนุนทางสังคม (Social Support) และ ปีการศึกษาเฉลี่ยของประชากร (Mean Years of Schooling))_

#### 4. เสรีภาพในการเลือกใช้ชีวิต (Freedom to Make Life Choices) และ รายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita) 
* รายได้ประชาชาติต่อหัวที่สูงช่วยเสริมสร้างเสรีภาพในการเลือกใช้ชีวิต (ภาพที่ 7) ทำให้ประชากรมีทางเลือกและโอกาสในการพัฒนาชีวิตอย่างมีความสุข ในทางกลับกัน ประเทศที่มีรายได้ต่ำมักประสบปัญหาด้านเสรีภาพ ซึ่งส่งผลให้คุณภาพชีวิตและความสุขของประชากรต่ำลง
![img](https://i.imgur.com/XTnRztO.png)

_(ภาพที่ 7: กราฟแสดงความสัมพันธ์ระหว่างเสรีภาพในการเลือกใช้ชีวิต (Freedom to Make Life Choices) และ รายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita))_

#### 5. เสรีภาพในการเลือกใช้ชีวิต (Freedom to Make Life Choices) และ อายุขัยเฉลี่ย (Life Expectancy at Birth) 
* อายุขัยเฉลี่ยที่สูงมีผลสำคัญต่อเสรีภาพในการเลือกใช้ชีวิต (ภาพที่ 8) เพราะช่วยให้ประชากรมีโอกาสในการตัดสินใจที่หลากหลายและดีกว่า ในขณะที่ประเทศที่มีอายุขัยเฉลี่ยต่ำจะเผชิญกับความท้าทายด้านสุขภาพและโอกาสที่จำกัด ซึ่งส่งผลให้ความสุขของประชากรลดลงเช่นกัน
 ![img](https://i.imgur.com/9FH9VL3.png)

_(ภาพที่ 8: กราฟแสดงความสัมพันธ์ระหว่างเสรีภาพในการเลือกใช้ชีวิต (Freedom to Make Life Choices) และ อายุขัยเฉลี่ย (Life Expectancy at Birth))_

#### 6. เสรีภาพในการเลือกใช้ชีวิต (Freedom to Make Life Choices) และ ปีการศึกษาเฉลี่ยของประชากร (Mean Years of Schooling) 
* การศึกษามีความสำคัญต่อเสรีภาพในการเลือกใช้ชีวิต (ภาพที่ 9) ปีการศึกษาเฉลี่ยที่สูงช่วยเสริมสร้างความสามารถในการตัดสินใจที่หลากหลายและมีคุณภาพ ทำให้ประชากรมีความสุขมากขึ้น ในขณะที่ประเทศที่มีปีการศึกษาเฉลี่ยต่ำมักต้องเผชิญกับความท้าทายที่ส่งผลต่อโอกาสในการเลือกใช้ชีวิตและความสุขโดยรวม
![img](https://i.imgur.com/WxKVCkG.png)

_(ภาพที่ 9: กราฟแสดงความสัมพันธ์ระหว่างเสรีภาพในการเลือกใช้ชีวิต (Freedom to Make Life Choices) และ ปีการศึกษาเฉลี่ยของประชากร (Mean Years of Schooling) )_

#### 7. ความใจดีหรือการมีน้ำใจ (Generosity) และ ความเหลื่อมล้ำทางการศึกษา (Inequality in Education) 
* ความเหลื่อมล้ำทางการศึกษาส่งผลต่อความใจดีในสังคม (ภาพที่ 10) การเข้าถึงการศึกษาที่ดีช่วยเสริมสร้างความสามารถในการเข้าใจและให้ความช่วยเหลือผู้อื่น ส่งผลให้มีสังคมที่มีความสุขมากขึ้น ขณะที่ความเหลื่อมล้ำทางการศึกษาสูงสามารถจำกัดความใจดีและคุณภาพชีวิตของประชาชนได้
![img](https://i.imgur.com/NRHrK4V.png)

_(ภาพที่ 10: กราฟแสดงความสัมพันธ์ระหว่างความใจดีหรือการมีน้ำใจ (Generosity) และ ความเหลื่อมล้ำทางการศึกษา (Inequality in Education))_

#### 8. ความรู้สึกเชิงลบ ( Negative Affect) และ ความเหลื่อมล้ำทางการศึกษา (Inequality in Education) 
* ความไม่เท่าเทียมทางการศึกษามีความสัมพันธ์เชิงบวกกับความรู้สึกเชิงลบ (ภาพที่ 11) หมายความว่าการลดช่องว่างในการเข้าถึงการศึกษาจะช่วยลดความเครียดได้ ข้อมูลนี้ชี้ให้เห็นถึงความสำคัญของการส่งเสริมการศึกษาที่เท่าเทียมสำหรับประชากรทุกคน โดยเฉพาะในกลุ่มประชากรที่ขาดโอกาส 
 ![img](https://i.imgur.com/f5slrTN.png)

_(ภาพที่ 11: กราฟแสดงความสัมพันธ์ระหว่างความรู้สึกเชิงลบ ( Negative Affect) และ ความไม่เท่าเทียมในด้านการศึกษา (Inequality in education))_
  
#### 9. ความรู้สึกเชิงลบ ( Negative Affect) และ รายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita)
* รายได้ประชาชาติต่อหัวที่สูงขึ้นมีความสัมพันธ์กับความรู้สึกเชิงลบที่ลดลง (ภาพที่ 12) แสดงให้เห็นว่าความมั่นคงทางเศรษฐกิจมีบทบาทสำคัญในการลดความเครียด การพัฒนาเศรษฐกิจและสร้างโอกาสทางการเงินที่มั่นคงจึงเป็นแนวทางที่ดีในการสร้างความสุขให้แก่ประชากร
  ![img](https://i.imgur.com/8mqFNYz.png)
  
  _( ภาพที่ 12: กราฟแสดงความสัมพันธ์ระหว่างความรู้สึกเชิงลบ ( Negative Affect) และ รายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita))_
   
#### 10. ความรู้สึกเชิงลบ ( Negative Affect) และ ความไม่เท่าเทียมในอายุขัย (Inequality in Life Expectancy)
* ความไม่เท่าเทียมในอายุขัยสะท้อนถึงการเข้าถึงบริการสุขภาพที่ไม่เท่าเทียม ซึ่งสัมพันธ์กับความรู้สึกเชิงลบที่สูงขึ้น (ภาพที่ 13) การพัฒนาระบบสุขภาพที่มีการเข้าถึงอย่างเท่าเทียมสามารถช่วยลดความเครียดและสร้างความพึงพอใจในชีวิตให้กับประชากรได้อย่างยั่งยืน
  ![img](https://i.imgur.com/bvP8IrS.png)
  
  _( ภาพที่ 13: กราฟแสดงความสัมพันธ์ระหว่างความรู้สึกเชิงลบ ( Negative Affect) และ ความไม่เท่าเทียมในอายุขัย (Inequality in life expectancy))_

#### 11. ความรู้สึกเชิงลบ ( Negative Affect) และ อายุขัยเฉลี่ย (Life Expectancy at Birth)
* อายุขัยเฉลี่ยที่สูงขึ้นมีความสัมพันธ์เชิงลบกับความรู้สึกเชิงลบ (ภาพที่ 14) นั่นคือเมื่อประชากรมีชีวิตยืนยาวขึ้น พวกเขามีแนวโน้มที่จะมีความเครียดน้อยลง การส่งเสริมสุขภาพและยกระดับคุณภาพชีวิตจึงเป็นแนวทางที่ยั่งยืนในการลดปัญหาทางอารมณ์และความเครียด
  ![img](https://i.imgur.com/k8syEhr.png)
  
  _( ภาพที่ 14: กราฟแสดงความสัมพันธ์ระหว่างความรู้สึกเชิงลบ ( Negative Affect) และ อายุขัยเฉลี่ย (life expectancy at birth))_

### 3.3 Compare to Thailand

เมื่อพิจารณาตัวชี้วัดสำคัญอย่าง **ความพึงพอใจในชีวิต (Life Ladder)** และ **ดัชนีคุณภาพชีวิตที่ดี (Human Development Index)**  พบว่าปัจจัยทั้งสองมีความสัมพันธ์ในลักษณะที่หลากหลาย ทั้งในทิศทางเดียวกันและทิศทางตรงกันข้าม ข้อมูลเหล่านี้สะท้อนให้เห็นถึงความเชื่อมโยงระหว่างคุณภาพชีวิตที่ดีขึ้นและระดับความสุขที่เพิ่มขึ้นในบางภูมิภาค ขณะที่ในบางพื้นที่แม้จะมีการพัฒนาในด้านหนึ่ง แต่กลับไม่ส่งผลให้ความสุขของประชากรเพิ่มขึ้นอย่างชัดเจน

ด้วยการนำข้อมูลเหล่านี้มาเปรียบเทียบกับประเทศไทย จะช่วยให้เรามองเห็นภาพรวมของสถานการณ์ในประเทศและความท้าทายที่อาจมีผลกระทบต่อคุณภาพชีวิตและความสุขของคนไทย การวิเคราะห์เชิงเปรียบเทียบจะช่วยให้เราเข้าใจปัจจัยที่มีบทบาทสำคัญในการพัฒนาคุณภาพชีวิต และสามารถนำไปใช้ในการกำหนดนโยบายหรือมาตรการที่เหมาะสมเพื่อยกระดับความสุขของประชากรไทยในอนาคต โดยพิจารณาจากตัวชี้วัดดังนี้:

* **ความพึงพอใจในชีวิต (Life Ladder)**
  	* จากข้อมูล (ภาพที่ 15) พบว่าประเทศที่อยู่ใน 10 อันดับที่มีความพึงพอใจในชีวิตสูงสุด (Life Ladder) ส่วนใหญ่อยู่ใน**ภูมิภาค Europe & Central Asia**
เมื่อเปรียบเทียบกับประเทศไทย ที่มีค่าความพึงพอใจอยู่ที่ **6.10** ซึ่งต่ำกว่าค่าเฉลี่ยของ 10 อันดับประเทศข้างต้นอย่างชัดเจน _(ทุกประเทศใน 10 อันดับ ล้วนมีค่าความพึงพอใจมากกว่า 7.0)_

  ![img](https://i.imgur.com/7T04z8n.png)
  
_( ภาพที่ 15: กราฟแสดง 10 อันดับประเทศที่มีความพึงพอใจในชีวิต (Life Ladder) สูงที่สุดเปรียบเทียบกับประเทศไทย)_


* **รายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita)**
  	* ประเทศไทยมี รายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita) อยู่ที่ 16,0009.40 USD ซึ่งน้อยกว่าประเทศใน Top 10 อย่างเห็นได้ชัด (ภาพที่ 16)
	* อันดับที่ 1 คือประเทศสวิตเซอร์แลนด์ อยู่ที่ 67,568.25 USD ซึ่งมากกว่าประเทศไทย**ถึง 4 เท่า**

  ![img](https://i.imgur.com/aBOSGGk.png)
  
_( ภาพที่ 16: กราฟแสดงรายได้มวลรวมประชาชาติต่อหัวของ 10 อันดับประเทศที่มีความพึงพอใจในชีวิตสูงที่สุด เทียบกับประเทศไทย)_

*  **การรับรู้ถึงการคอร์รัปชัน (Perceptions of corruption)**
  	* ประเทศสวีเดน, ประเทศฟินแลนด์ และประเทศเดนมาร์ก เป็นประเทศที่มีการรับรู้ถึงการคอร์รัปชัน อยู่ที่ 0.24, 0.22 และ 0.19 ตามลำดับ ซึ่งถือว่าอยู่ในระดับต่ำ (ภาพที่ 17) แต่เป็นประเทศที่มีความพึงพอใจในชีวิต (Life Ladder) สูงมาก ความโปร่งใสและการขาดการคอร์รัปชันในประเทศเหล่านี้ได้สร้างความเชื่อมั่นต่อระบบการปกครองและองค์กรภายในประเทศ ซึ่งส่งผลโดยตรงต่อคุณภาพชีวิตที่ดีขึ้นของประชากรให้ดียิ่งขึ้น
 	* ในขณะที่ประเทศไทยมีการรับรู้ถึงการคอร์รัปชัน สูงถึง 0.90 แสดงให้เห็นว่าการรับรู้ถึงการทุจริตในประเทศไทยยังคงเป็นปัญหาหลักที่อาจส่งผลกระทบต่อความไว้วางใจในรัฐบาลและองค์กรต่างๆ ซึ่งอาจเป็นปัจจัยที่ขัดขวางการพัฒนาคุณภาพชีวิตของประชากร

 ![img](https://i.imgur.com/bGejFDJ.png)
 
_( ภาพที่ 17: กราฟแสดงการรับรู้ถึงการคอร์รัปชันของ 10 อันดับประเทศที่มีความพึงพอใจในชีวิตสูงที่สุด เทียบกับประเทศไทย)_
  

 * **ความเหลื่อมล้ำทางรายได้ (Inequality in Income)**
   	* ประเทศไทยมีความเหลื่อมล้ำทางรายได้สูงถึงร้อยละ 22.40 ซึ่งถือว่าสูง (ภาพที่ 18) เมื่อเปรียบเทียบกับประเทศใน Top 10 โดยเฉพาะเมื่อเทียบกับประเทศไอซ์แลนด์ (10.94%), ประเทศเดนมาร์ก (10.96%) และ ประเทศฟินแลนด์ (11.75%)
   	* ในกรณีของประเทศอิสราเอล แม้ว่าจะมีความเหลื่อมล้ำทางรายได้สูงถึงร้อยละ 24.50 ซึ่งสูงกว่าประเทศอื่นๆ ในกลุ่ม Top 10 แต่ยังคงมีระดับความพึงพอใจในชีวิต (Life Ladder) ที่สูงอยู่ สะท้อนให้เห็นว่าความเหลื่อมล้ำทางรายได้ไม่ได้เป็นปัจจัยเดียวที่ส่งผลต่อความสุขเสมอไป และอาจมีปัจจัยอื่นๆ ที่มีบทบาทสำคัญในด้านนี้

  ![img](https://i.imgur.com/3f4zaus.png)
  
_( ภาพที่ 18: กราฟแสดงความเหลื่อมล้ำทางรายได้ของ 10 อันดับประเทศที่มีความพึงพอใจในชีวิตสูงที่สุด เทียบกับประเทศไทย)_

 
* **ปีการศึกษาเฉลี่ยของประชากร (Mean Years of Schooling)**
  	* ประเทศไทยมีค่าปีการศึกษาเฉลี่ยของประชากร (Mean Years of Schooling) อยู่ที่ 8.44 ปี ซึ่งต่ำกว่าค่าเฉลี่ยของประเทศในกลุ่ม Top 10 ที่ส่วนใหญ่อยู่ที่มากกว่า 12 ปี
   	* ประเทศที่มีค่าเฉลี่ยการศึกษาสูงที่สุดคือ ประเทศสวิตเซอร์แลนด์ อยู่ที่ 13.77 ปี และ ประเทศไอซ์แลนด์ อยู่ที่ 13.55 ปี
    	* ค่าเฉลี่ยการศึกษาที่ต่ำของประเทศไทยเมื่อเทียบกับกลุ่มประเทศที่มีความพึงพอใจในชีวิตสูงบ่งบอกถึงข้อจำกัดในการเข้าถึงการศึกษาอย่างเท่าเทียม และเป็นหนึ่งในปัจจัยที่อาจส่งผลให้ความพึงพอใจในชีวิตยังไม่สูงเทียบเท่ากับประเทศที่พัฒนาแล้ว

  ![img](https://i.imgur.com/IrrN0jd.png)
  
_( ภาพที่ 19: กราฟแสดงปีการศึกษาเฉลี่ยของประชากรของ 10 อันดับประเทศที่มีความพึงพอใจในชีวิตสูงที่สุด เทียบกับประเทศไทย)_


* **ความรู้สึกเชิงบวก (Positive Affect)**
  	* ประเทศไทยมีค่าความรู้สึกเชิงบวก (Positive Affect) ที่ 0.79 ซึ่งถือว่าค่อนข้างสูง เทียบเท่ากับประเทศที่มีความพึงพอใจในชีวิตสูงอย่าง ประเทศเดนมาร์ก และ ประเทศไอซ์แลนด์ ที่มีค่าความรู้สึกเชิงบวกเท่ากัน
    	* ประเทศอิสราเอลมีค่าความรู้สึกเชิงบวกต่ำที่สุดที่ 0.60 เมื่อเทียบกับประเทศอื่นในกลุ่ม Top 10 แม้ว่าจะมี ความพึงพอใจในชีวิตสูง แสดงถึงความไม่สอดคล้องระหว่างความสุขทางอารมณ์
     	* แม้ประเทศไทยจะมีรายได้มวลรวมประชาชาติต่อหัว (Gross National Income per Capita) ที่ต่ำกว่าเมื่อเทียบกับประเทศในกลุ่ม Top 10 แต่ค่าความรู้สึกเชิงบวกของไทยยังคงอยู่ในระดับสูง แสดงให้เห็นว่าความสุขทางอารมณ์ของประชากรไม่ได้ขึ้นอยู่กับปัจจัยทางเศรษฐกิจเสมอไป
  
  ![img](https://i.imgur.com/uCpSuSh.png)
  
_( ภาพที่ 20:กราฟแสดงความรู้สึกเชิงบวกของ 10 อันดับประเทศที่มีความพึงพอใจในชีวิตสูงที่สุด เทียบกับประเทศไทย)_


 * **ความรู้สึกเชิงลบ ( Negative Affect)**
   	* ประเทศไทยมีค่าความรู้สึกเชิงลบ (Negative Affect) อยู่ที่ 0.22 ซึ่งสูงกว่า ประเทศนิวซีแลนด์, ประเทศสวิตเซอร์แลนด์, และประเทศฟินแลนด์ ซึ่งมีค่าอยู่ที่ 0.19 โดยกราฟนี้แสดงถึงความรู้สึกเชิงลบของประชาชน เช่น ความเครียด ความกังวล หรือความเศร้า
	* ประเทศอิสราเอลมีค่าความรู้สึกเชิงลบสูงสุดที่ 0.27 แสดงถึงความเครียดที่มากขึ้น แม้ว่าจะเป็นประเทศที่มีค่าความพึงพอใจในชีวิตสูงก็ตาม แสดงให้เห็นว่าการมีความพึงพอใจในชีวิตที่ดี ไม่ได้หมายความว่าความรู้สึกเชิงลบจะน้อยลงเสมอไป
	* แม้ว่าประเทศไทยมีค่าความรู้สึกเชิงบวกที่สูง แต่การมีค่าความรู้สึกเชิงลบสูงเช่นกัน แสดงให้เห็นถึงความจำเป็นในการจัดการความเครียดหรือปัญหาสังคมที่อาจส่งผลต่อสุขภาพจิตของประชาชน

  ![img](https://i.imgur.com/yMtFE7D.png)
  
_( ภาพที่ 21: กราฟแสดงความรู้สึกเชิงลบของ 10 อันดับประเทศที่มีความพึงพอใจในชีวิตสูงที่สุด เทียบกับประเทศไทย)_



 * **ดัชนีการพัฒนามนุษย์ปรับปรุงด้วยความไม่เท่าเทียม (Inequality-adjusted Human Development Index: IHDI)**
   
   **IHDI สูงสัมพันธ์กับ Life Ladder สูง**
   	* กราฟแสดงให้เห็นถึงความสัมพันธ์เชิงบวกระหว่าง IHDI และ Life Ladder (ภาพที่ 22) ยิ่งค่า IHDI สูง ประเทศนั้นก็จะมีความพึงพอใจในการใช้ชีวิตที่สูงขึ้นเช่นกัน เช่น ประเทศในวงกลมสีน้ำเงินมีค่า IHDI สูงกว่า 0.8 และมีค่า Life Ladder เกิน 7.0 ซึ่งเป็นสัญญาณของคุณภาพชีวิตที่ดีและประชากรมีความพึงพอใจในชีวิตมากขึ้น
   	  
   **IHDI ต่ำสัมพันธ์กับ Life Ladder ต่ำ**
   	* ประเทศที่มีค่า IHDI ต่ำกว่า 0.4 จะเห็นได้ว่าค่า Life Ladder มักจะต่ำกว่า 5.0 ซึ่งแสดงถึงความไม่เท่าเทียมในโอกาสทางการศึกษา สุขภาพ และรายได้ และส่งผลให้ประชาชนมีความพึงพอใจในชีวิตน้อยลง โดยเฉพาะกลุ่มประเทศที่ค่า IHDI ต่ำกว่า 0.3 จะมี Life Ladder อยู่เพียงประมาณ 3-4 เท่านั้น
   	  
   **ประเทศไทยอยู่ในระดับกลาง:**
   	* ประเทศไทย (วงกลมสีส้ม) มีค่า IHDI ประมาณ 0.55 และค่า Life Ladder อยู่ที่ประมาณ 6.0 ซึ่งจัดอยู่ในระดับกลาง เมื่อเปรียบเทียบกับประเทศที่มีค่า IHDI สูง จะเห็นได้ว่า ประเทศไทยยังคงมีช่องว่างในการปรับปรุงเพื่อลดความไม่เท่าเทียมในด้านต่างๆ เช่น การศึกษาและรายได้ ซึ่งจะช่วยยกระดับความพึงพอใจในการใช้ชีวิตของประชาชนได้
   	  
  ![img](https://i.imgur.com/jCb72G7.png)
  
_( ภาพที่ 22: แสดงความสัมพันธ์ระหว่างดัชนีการพัฒนามนุษย์ปรับปรุงด้วยความไม่เท่าเทียม (IHDI) และความพึงพอใจใจชีวิต (Life Ladder))_
  

### 3.4 Conclusion

* การพัฒนาคุณภาพชีวิตของประเทศไทยในระยะยาวจำเป็นต้องมุ่งเน้นไปที่การลดความไม่เท่าเทียมในหลายมิติ โดยเฉพาะใ**นด้านรายได้ การศึกษา และการแก้ไขปัญหาการคอร์รัปชัน** การลดความไม่เท่าเทียมในรายได้จะช่วยกระจายโอกาสทางเศรษฐกิจให้กับประชากรในวงกว้างมากขึ้น ขณะที่การเพิ่มการเข้าถึงการศึกษาที่เท่าเทียมจะส่งเสริมให้ประชาชนมีโอกาสพัฒนาตนเองและเข้าสู่ตลาดแรงงานที่มีคุณภาพ นอกจากนี้ การลดปัญหาคอร์รัปชันจะช่วยสร้างความเชื่อมั่นในระบบการปกครองและองค์กรต่างๆ ส่งผลให้ประชาชนมีความมั่นใจในอนาคตของประเทศ

* เมื่อประเทศไทยสามารถลดความไม่เท่าเทียมในมิติเหล่านี้ได้ ก็จะเป็นการสร้างสังคมที่มีความเท่าเทียมมากขึ้น ทุกคนจะมีโอกาสเข้าถึงทรัพยากรและโอกาสทางเศรษฐกิจอย่างเป็นธรรม ซึ่งจะนำไปสู่การเพิ่มระดับความพึงพอใจในชีวิต (Life Ladder) และยกระดับคุณภาพชีวิตของประชากรให้ใกล้เคียงกับประเทศในกลุ่ม Top 10 ที่มีมาตรฐานการพัฒนาสูงกว่า ความเปลี่ยนแปลงนี้จะช่วยให้ประเทศไทยก้าวไปสู่ความเจริญที่ยั่งยืนและเป็นธรรมมากขึ้นในอนาคต
  


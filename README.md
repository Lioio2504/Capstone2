# Capstone2
# import libraries
  import pandas as pd
  import matplotlib.pyplot as plt
  import seaborn as sns
  import numpy as np

# Import Database and naming it
  df = pd.read_csv(r"C:\Users\Lionie Sibuea\Downloads\Transjakarta data.csv")
  df

  pd.set_option('display.max_columns', None)  # To show all columns
  pd.set_option('display.width', 1000)       # Adjust the display
  
  print(df)
# Data Cleansing
  len(df[df['payAmount'].isna()]) # Finding the qty of **missing amount** in the 'Pay Amount' column

  len(df[df['tapOutStops'].isna()]) # Finding the qty of **missing amount** in the 'TapOutStops' column

  df['payAmount_copy'] = df['tapOutStops'].apply(lambda x: 20000 if x is np.nan else np.nan) #Making sure that the column 'Pay Amount' filled with a def value 
  df['payAmount'] = df['payAmount'].fillna(df['payAmount_copy'])
  df.drop(labels='payAmount_copy', axis=1, inplace=True)

  df['corridorID'] = df['corridorID'].apply(lambda x: None if x is np.nan else x)
  df['corridorID'] = df['corridorID'].apply(lambda x: str(x).replace("JAK.", "") if x is not None else None)

  df_corr_info = df[['corridorID', 'corridorName']].copy()
  df_corr_info.dropna(how="all", inplace=True)
  df_corr_info.head()

  len(df_corr_info)

  df_corr_info.drop_duplicates(inplace=True)

  len(df_corr_info)

  # Filling the blanks (corr_name with known corr_name)
for i in range(0, len(df_corr_info)):
    corr_ids = df_corr_info.iloc[i, 0]
    corr_name = df_corr_info[df_corr_info['corridorID'] == corr_ids]['corridorName'].dropna().values 
    if len(corr_name) > 0:
        df_corr_info.iloc[i, 1] = corr_name[0] 

  # Filling the blanks (corr_ids with known corr_ids)
for i in range(0, len(df_corr_info)):
    corr_name = df_corr_info.iloc[i, 1]
    corr_ids = df_corr_info[df_corr_info['corridorName'] == corr_name]['corridorID'].dropna().values
    if len(corr_ids) > 0:
        df_corr_info.iloc[i, 0] = corr_ids[0]

df_corr_info.drop_duplicates(subset='corridorName', inplace=True)
df_corr_info.drop_duplicates(subset='corridorID', inplace=True)
df_corr_info.dropna(how='any')
df_corr_info.reset_index(inplace=True, drop=True)
df_corr_info.columns = ['corridorID', 'corridorName_TRUE']

df = pd.merge(
    left = df,
    right = df_corr_info,
    on = 'corridorID'
)

df['corridorName'] = df['corridorName'].fillna(df['corridorName_TRUE'])
df.drop('corridorName_TRUE', axis=1, inplace=True)

df['tapOutStops'] = df['tapOutStops'].apply(lambda x: df['tapOutStops'].mode().values[0] if x is np.nan else x)
df['tapInStops'] = df['tapInStops'].apply(lambda x: df['tapInStops'].mode().values[0] if x is np.nan else x)

df=df.drop(labels=['tapInStopsLat', 'tapInStopsLon', 'tapOutStopsLat', 'tapOutStopsLon'], axis=1) #Droping unused column

# changing the birthdate column to string
df['payCardBirthDate']=df['payCardBirthDate'].astype(str)

# Visualization of How Many Male and Female who rode Transjakarta
gender_counts = df['payCardSex'].value_counts()
plt.figure(figsize=(8, 5))
plt.bar(
    x=gender_counts.index,
    height=gender_counts,
    color=['pink', 'blue']
)

plt.title('Gender of Transjakarta User')
plt.ylabel('Count')
plt.show()

# Based on the data there are more females Transjakarta user

import matplotlib.pyplot as plt
bank_type = df['payCardBank'].value_counts()

# Making the pie chart
plt.figure(figsize=(8, 8))
plt.pie(
    bank_type,
    labels = bank_type.index,
    autopct ='%1.1f%%',
    colors =['green', 'lightblue', 'lime', 'darkblue', 'lightgreen', 'yellow'],
    startangle=90
)
plt.title('Distribution of Transjakarta Users preffered bank card')
plt.show()

import pandas as pd
import matplotlib.pyplot as plt

bank_gender_usage = df.groupby(['payCardBank', 'payCardSex']).size().unstack(fill_value=0)

bank_gender_usage.plot(kind='bar', stacked=True, figsize=(8, 4), color=['lightblue', 'lightcoral'])

# Visualization of paycardbank usage bay gender
plt.title('PayCardBank Usage by Gender', fontsize=16)
plt.xlabel('PayCardBank', fontsize=12)
plt.ylabel('Number of Usages', fontsize=12)

plt.legend(title='Gender')
plt.xticks(rotation=45)

plt.tight_layout()
plt.show()

# Saving dataset to excel so that it's easier to open in tableau
df.to_excel('Transjakarta_Cleansed.xlsx',index=False) 

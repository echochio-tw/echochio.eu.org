---
layout: post
title: Python Pandas DataFrame to Google Sheets
date: 2021-06-22
tags: python Pandas Sheets
---

Python Pandas DataFrame to Google Sheets

```
by default it will be called “Master”.
URL = 'https://docs.google.com/spreadsheets/d/1vo9AMK_1BMbB1-YQHysfFqLchj1iUhCzVZT6JSpW111/edit#gid=0'
spreadsheet_key = '1vo9AMK_1BMbB1-YQHysfFqLchj1iUhCzVZT6JSpW111'
```

```
import pandas as pd
import gspread
import df2gspread as d2g

d = {'col1': [1, 2], 'col2': [3, 4]}
df = pd.DataFrame(data=d)

scope = ['https://spreadsheets.google.com/feeds',
         'https://www.googleapis.com/auth/drive']
credentials = ServiceAccountCredentials.from_json_keyfile_name(
    'jsonFileFromGoogle.json', scope)
gc = gspread.authorize(credentials)

spreadsheet_key = '1vo9AMK_1BMbB1-YQHysfFqLchj1iUhCzVZT6JSpW111'
wks_name = 'Master'
d2g.upload(df, spreadsheet_key, wks_name, credentials=credentials, row_names=True)
```

---
layout: post
title: gcp billing bigquery python get
date: 2020-12-11
tags: gcp invoice
---
To enable export or update the export settings, click Edit settings.

From the Project list, select the project that you set up to contain your BigQuery dataset. ...

From the Billing export dataset list, select the dataset that you set up to contain your exported Cloud Billing data. ...

Click Save.


用 python 抓帳單

GCP token service-accounts 建立
```
gcloud iam service-accounts keys create ~/key.json   --iam-account 3XXXXXXXX@cloudservices.gserviceaccount.com
```

<img src="/images/posts/google-doc/a1.png">

帳單要設定 BigQuery

<img src="/images/posts/google-doc/a2.png">

```
bq ls
bq ls data
```

<img src="/images/posts/google-doc/a3.png">

<img src="/images/posts/google-doc/a4.png">

```
import os,requests
os.environ['GOOGLE_APPLICATION_CREDENTIALS'] = '/root/gcp/token.json'
from google.cloud import bigquery

# Construct a BigQuery client object.
client = bigquery.Client()

query = """
SELECT
  invoice.month,
  SUM(cost)
    + SUM(IFNULL((SELECT SUM(c.amount)
                  FROM UNNEST(credits) c), 0))
    AS total,
  (SUM(CAST(cost * 1000000 AS int64))
    + SUM(IFNULL((SELECT SUM(CAST(c.amount * 1000000 as int64))
                  FROM UNNEST(credits) c), 0))) / 1000000
    AS total_exact
FROM `XXXXXX-2XXXXXX.data.gcp_billing_export_v1_01111C_E1111_81111`
GROUP BY 1
ORDER BY 1 ASC
"""
query_job = client.query(query)  # Make an API request.
balance = "GCP google 云(XXXX)\n"
for r in query_job:
    balance = balance + "年月: {} 费用: {}\n".format(r.get('month'), r.get('total_exact'))
data = {'chat_id': '-3XXXXXXXXX', 'text': balance}
url = 'https://api.telegram.org/bot123443323:AXXXXXXXXXXXXXXXXXXXXXXX/sendMessage'
requests.post(url, data, verify=False))

```

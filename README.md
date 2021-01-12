# Text Analysis with an Elasticsearch Index of a MongoDB Collection

Elasticsearch has been the talk of the Data Engineering town for some time now, but only since it's Docker debute has it been accessible to us amateur data scientists. So,I ask, can this hyped up elasticsearch really improve your Kaggle competitions' entries? This article explores how an amateur might go about deploying an Elasticsearch instance and querying the results for data science projects.

## Twitter Dataset from Kaggle
[Sentiment Analysis with 1.6 million Tweets](https://www.kaggle.com/kazanova/sentiment140)

## Things you'll need & I'm assuming.

I'm assuming you know the basics of setting up a docker container. If not, learn about it [here](https://docker-curriculum.com).

You'll need Docker images for 
1. elasticsearch
2. Mongo-Connector
3. MongoDB
[These should pull automatically when you run the script]

You'll need Python Packages for
1. elasticsearch
2. mongo-connector
3. psycopg2 &/ pymongo

```bash
conda install elasticsearch
conda install pymongo
pip install mongo-connector
```

[Mongo-Connector](https://medium.com/@rajkumar012786/how-to-setup-mongodb-sync-with-elastic-search-via-mongo-connector-in-few-minutes-57c39c9f62aa)

### Cleaning the Data ante Upload.

For cleaning and loading the data into a Pandas dataframe, I found that the encoding is different, so I had to add that function to get the proper encoding.

I cleaned the time as a time element, and dropped the 'flag' dimension since it only one value.

```Python

def get_encoding(file):
    import codecs
    encodings = ['UTF_8', 'ASCII','latin_1','iso8859_1','iso8859_15','cp1252','gb2312','euc_kr','cp1251','koi8_r','cp1250','iso8859_2',
 'cp1256','iso8859_6','big5','big5hkscs','cp037','cp424','cp437','cp500','cp737','cp775','cp850',
 'cp852','cp855','cp856','cp857','cp860','cp861','cp862','cp863','cp864','cp865','cp866','cp869',
 'cp874','cp875','cp932','cp949','cp950','cp1006','cp1026','cp1140','cp1250','cp1251','cp1252','cp1253','cp1254','cp1255',
 'cp1256','cp1257','cp1258','euc_jp','euc_jis_2004','euc_jisx0213','euc_kr','gb2312','gbk','gb18030','hz',
'iso2022_jp','iso2022_jp_1','iso2022_jp_2','iso2022_jp_2004','iso2022_jp_3','iso2022_jp_ext','iso2022_kr',
 'iso8859_2','iso8859_3','iso8859_4','iso8859_5','iso8859_6','iso8859_7','iso8859_8','iso8859_9','iso8859_10','iso8859_13','iso8859_14','iso8859_15',
 'johab','koi8_r','koi8_u','mac_cyrillic','mac_greek','mac_iceland','mac_latin2','mac_roman','mac_turkish',
 'shift_jis','shift_jis_2004','shift_jisx0213','utf_16','utf_16_be','utf_16_le','utf_7']
    
    for e in encodings:
        try:
            fh = codecs.open(file, 'r', encoding=e)
            fh.readlines()
            fh.seek(0)
        except UnicodeDecodeError:
            pass
        else:
            return e
            break

def load_transform(file):
    #loads, cleans the time feature, and drops the 'flag' column

    #loading params
    e = get_encoding(file_o)
    headers = ['target', 'id','date','drop_this','user','text']
    
    #create initial df
    df = pd.read_csv(file_o, names=headers, encoding=e)
    
    #clean time
    df.date = pd.to_datetime(df.date, utc=False)
    
    #drop column
    df = df.drop(['drop_this'], axis=1)
    
    return df
```

### Setting up the MongoDB Collection

After cleaning the data in Pandas, I uploaded the DF via Python. 

```Python
#params
db = 'tweetsdb'
coll = 'tweets'
host = "mongodb://localhost:27017/"

#libraries
from pymongo import MongoClient

#functions
def conn_to_mongo(host, db, coll):
    Host =  MongoClient(host)
    DB = myclient[db]
    Collection = DB[coll]
    return Host, DB, Collection
        
def upload_df_to_mongodb(df, host, db, coll,orient='records'):
    # Connect to MongoDB
    Host, DB, Collection = conn_to_mongo(host, db, coll)

    #prep df for transfer
    df.reset_index(inplace=True)
    data_dict = df.to_dict(orient)

    # Insert collection
    Collection.insert_many(data_dict)
```

### Setting up the Elastic-MongoDB connection


### Retrieving Data via a Flask search field


### Retrieving Data via a Tableau workbook


### Retrieving Data via a Jupyter-Notebook
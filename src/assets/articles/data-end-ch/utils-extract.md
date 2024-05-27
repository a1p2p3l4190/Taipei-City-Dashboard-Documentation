---
title: 通用函式-抽取階段

---


包含下載、解壓縮、取得 data.taipei API、取得 data.taipei 檔案更新時間、讀取 geoJOSN 的範例、讀取 SHPfile 的範例、讀取 KML 的範例。你可以在 `/dags/utils/extract_stage.py` 查看相關程式碼。

> **w01**
> 務必確認你已經查看[下載並設定專案](/data-end/project-setup)章節並設置完成。

> **w02**
> 若開發者需要在 Airflow 環境單獨測試以下程式碼，需先執行以下程式碼取得環境設定。
> ``` python
> from airflow import DAG
> ```

> **w03**
> 若開發者需要在非 Airflow 的環境測試以下程式碼，需添加以下幾行程式，以確保將本專案的路徑加入環境變數，從而能找到 `utils` 與 `settings` 等資料夾：
> ``` python
> import os
> import sys
>
> dags_path = os.path.join(os.getcwd(), 'dags')  # Should be looks like './tuic-pipeline-airflow/dags'
> sys.path.append(dags_path)
> ```

## 函式說明

### def download_file(file_name, url, file_folder)

從 `url` 下載檔案儲存於 `./{file_folder}/{file_name}`，並回傳下載檔案的完整路徑。範例如下：

``` python
# Read GeoJSON
# GeoJSON is a special format of JSON that represents geographical data
# The extension of a GeoJSON file can be .geojson or .json.
import geopandas as gpd
from utils.extract_stage import download_file_by_filename

URL = "https://pwdgis.taipei/wg/opendata/I0201-5.geojson"
FILE_NAME = "goose_sanctuary.geojson"
FILE_ENCODING = "UTF-8"

local_file = download_file(FILE_NAME, URL)
gdata = gpd.read_file(local_file, encoding=FILE_ENCODING, driver="GeoJSON")
print(gdata)
```

```
 >>> output:
   Id     名稱            面積    類型  集水區  物理型  水文HY  濱水植  水質WQ  生物BI  MIWC2017                                           geometry
0   3  雁鴨保護區  1.799444e+06  重要濕地  NaN  NaN   NaN  NaN   NaN   NaN       NaN  MULTIPOLYGON (((121.51075 25.02214, 121.51083 ...

```


### def unzip_file_to_target_folder(zip_file, unzip_path)

將 `zip_file` 解壓縮至 `unzip_path` 資料夾。

``` python
# Read Shapefile
# Shapefile is a popular geospatial vector data format for geographic information system software.
# The shapefile is a set of files with the same name but different extensions.
# Usually, theses files been compressed into a zip file.
import geopandas as gpd
from utils.extract_stage import (
    download_file,
    unzip_file_to_target_folder,
)
from settings.global_config import DATA_PATH

URL = r"https://data.moa.gov.tw/OpenData/GetOpenDataFile.aspx?id=I88&FileType=SHP&RID=27237"
FILE_NAME = "debris_area.zip"
unzip_path = f"{DATA_PATH}/debris_area"
FILE_ENCODING = "UTF-8"

zip_file = download_file(FILE_NAME, URL)
unzip_file_to_target_folder(zip_file, unzip_path)
target_shp_file = [f for f in os.listdir(unzip_path) if f.endswith("shp")][0]
gdata = gpd.read_file(f"{unzip_path}/{target_shp_file}", encoding=FILE_ENCODING)
print(gdata)
```

```
>>> print(gdata)
        ID Debrisno  ... Dbno_old                                           geometry
0        1  宜縣DF135  ...   宜蘭A089  LINESTRING (313537.820 2726900.950, 313625.420...
1        2  宜縣DF131  ...   宜蘭A088  LINESTRING (319284.480 2727626.340, 319308.250...
2        3  宜縣DF132  ...   宜蘭A087  LINESTRING (318877.260 2727421.020, 318878.620...
3        4  宜縣DF133  ...   宜蘭A086  MULTILINESTRING ((317842.890 2725794.540, 3178...
4        5  宜縣DF134  ...    宜蘭028  MULTILINESTRING ((315765.720 2726200.720, 3157...
...    ...      ...  ...      ...                                                ...
1727  1728  花縣DF098  ...    花蓮020  LINESTRING (303782.140 2619541.820, 303857.320...
1728  1729  花縣DF103  ...   花蓮A138  LINESTRING (302751.200 2607101.490, 302746.680...
1729  1730  花縣DF104  ...   花蓮A139  LINESTRING (302677.050 2606792.820, 302667.830...
1730  1731  花縣DF105  ...    花蓮025  MULTILINESTRING ((300594.180 2604587.920, 3005...
1731  1732  花縣DF106  ...    花蓮026  MULTILINESTRING ((300470.400 2604218.870, 3004...

[1732 rows x 31 columns]
```

### def get_data_taipei_api(rid)

根據 `rid` 生成 data.taipei API URL，迴圈取得完整資料後回傳。(API 一次最多回傳 1000 筆)

``` python
from utils.extract_stage import get_data_taipei_api

rid = "04a3d195-ee97-467a-b066-e471ff99d15d"
res = get_data_taipei_api(rid)
print(res)
```

```
>>> print(res[0])
{'_id': 1, '_importdate': {'date': '2024-03-01 14:46:51.602832', 'timezone_type': 3, 'timezone': 'Asia/Taipei'}, '機構名稱': '郵政醫院（委託中英醫療社團法人經營）', '地址': '臺北市中正區福州街14號', 'x': '121.5186982', 'y': '25.02874869'}
```

### def read_kml(file)

讀取 `file` 的 kml 檔案成 `gpd.GeoDataFrame`。

``` python
from utils.extract_stage import download_file, read_kml

URL = "https://data.taipei/api/frontstage/tpeod/dataset/resource.download?rid=a69988de-6a49-4956-9220-40ebd7c42800"
FILE_NAME = "urban_bike_path.kml"

res = download_file(FILE_NAME, URL)
df = read_kml(res)
print(df.iloc[0])
```

```
>>> print(df.iloc[0])
Name                                                    三元街(西南側)
Description      編號：TP2329 名稱：三元街(西南側) 縣市別：台北市 起點描述：南海路 迄點描述：泉州街
geometry       LINESTRING Z (121.514241 25.027622 0, 121.5133...
Name: 0, dtype: object
```

> **i02**
> kmz 檔案的讀取方式非常類同，只需將 `.kmz` 副檔名修改為 `.zip`，並將其解壓縮，讀取解壓縮檔案的 `doc.kml`。

### def get_data_taipei_file_last_modified_time(page_id, rank)

取得 data.taipei 指定 `page_id` 頁面的檔案更新時間，通常在頁面下方，下載資料按鈕的左邊。一個頁面可能有多個檔案與更新時間，可使用 `rank` 參數取得指定位置（0 表示最上面）。

``` python
from utils.extract_stage import get_data_taipei_file_last_modified_time

PAGE_ID = "4fefd1b3-58b9-4dab-af00-724c715b0c58"

res = get_data_taipei_file_last_modified_time(PAGE_ID)
print(res)
```

```
>>> print(res)
'2023-06-06 09:53:08'
```

### def get_data_taipei_page_change_time(page_id, rank)

取得 data.taipei 指定 `page_id` 頁面的異動時間，通常在頁面左方頁籤。一個頁面可能有多個異動時間，可使用 `rank` 參數取得指定位置（0 表示最上面）。

``` python
from utils.extract_stage import get_data_taipei_page_change_time

PAGE_ID = "4fefd1b3-58b9-4dab-af00-724c715b0c58"

res = get_data_taipei_page_change_time(PAGE_ID)
print(res)
```

```
>>> print(res)
2023-09-08 10:02:06
```

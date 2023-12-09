# Big_Data_Project
## 제주도 창업을 위한 소비 패턴 분석
### 01. 데이터 준비하기

```python
# 패키지 설치 및 로드
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns
import matplotlib as mpl
import matplotlib.font_manager as fm
from matplotlib.ticker import FuncFormatter
import folium
from folium.plugins import MarkerCluster
```

'''python
# 데이터 불러오기
card1 = pd.read_csv('/content/제주도_카드_17.csv', encoding='cp949')
card2 = pd.read_csv('/content/제주도_카드_18.csv', encoding='cp949')
card3 = pd.read_csv('/content/제주도_카드_19.csv', encoding='cp949')
card4 = pd.read_csv('/content/제주도_카드_20.csv', encoding='cp949')

# 데이터 출력
card1, card2, card3, card4
'''

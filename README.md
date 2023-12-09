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

```python
# 데이터 불러오기
card1 = pd.read_csv('/content/제주도_카드_17.csv', encoding='cp949')
card2 = pd.read_csv('/content/제주도_카드_18.csv', encoding='cp949')
card3 = pd.read_csv('/content/제주도_카드_19.csv', encoding='cp949')
card4 = pd.read_csv('/content/제주도_카드_20.csv', encoding='cp949')

# 데이터 출력
card1, card2, card3, card4
```

```python
# 데이터 병합
card = pd.concat([card1, card2, card3, card4])

# 데이터 크기
card.shape

# 데이터 속성
card.info()
```

```python
# 사용하지 않는 컬럼, 행 삭제
card.drop(columns = ['업종코드', '데이터기준일자'], axis=1, inplace=True)

# 년월 컬럼 분리
card['년월'] = pd.to_datetime(card['년월'], format='%Y-%m')
card['연'] = card['년월'].dt.year.astype(int)
card['월'] = card['년월'].dt.month.astype(int)

# column 인덱스 재정렬
new_card = ['연', '월'] + [i for i in card.columns if i not in ['연', '월', '년월']]
card_data = card[new_card]

# 결측치 알수없음으로 채움
card_data = card_data.copy()
card_data['관광구분'].fillna('알수없음', inplace=True)

# 전처리
card_data['이용자 구분'] = card_data['이용자 구분'].replace(['중국', '일본', '동남아', '기타외국'], '외국인')

# 데이터 출력
card_data
```

```python
# 데이터 속성 출력
card_data.info()
```

```python
# 연도별 총 이용금액
total_amount_2017 = card_data[card_data['연'] == 2017]['이용금액'].sum()
total_amount_2018 = card_data[card_data['연'] == 2018]['이용금액'].sum()
total_amount_2019 = card_data[card_data['연'] == 2019]['이용금액'].sum()
total_amount_2020 = card_data[card_data['연'] == 2020]['이용금액'].sum()

# 새로운 데이터프레임 생성
total_amount = pd.DataFrame({
    '연도': [2017, 2018, 2019, 2020],
    '총 이용금액': [total_amount_2017, total_amount_2018, total_amount_2019, total_amount_2020]
})

# 데이터 출력
total_amount
```

```python
# 폰트 설치
!apt-get -qq -y install fonts-nanum > /dev/null

fe = fm.FontEntry(
    fname=r'/usr/share/fonts/truetype/nanum/NanumBarunGothic.ttf', name='NanumGothic')
fm.fontManager.ttflist.insert(0, fe)
plt.rcParams.update({'font.size': 10, 'font.family': 'NanumGothic'})

# 마이너스 표시 문제
mpl.rcParams['axes.unicode_minus'] = False

# 연도별 총 이용금액 시각화

# y축의 값이 1e12같이 지수로 나오는 것을 숫자로 변경
def currency_formatter(x, pos):
    return '{:,.0f}원'.format(x)
formatter = FuncFormatter(currency_formatter)
plt.gca().yaxis.set_major_formatter(formatter)

green_palette = sns.color_palette('Greens')

# x축에 '연도' 열에 있는 값들만 표시되게끔 변경
plt.xticks(total_amount['연도'])

plt.xlabel('연도')
plt.ylabel('총 이용금액')
plt.title('연도별 총 이용금액')
plt.bar(total_amount['연도'], total_amount['총 이용금액'], color=green_palette)

plt.show()
```

```python
# 새로운 데이터프레임 생성
grouped_data = card_data.groupby(['연', '업종명 대분류'])['이용금액'].sum().reset_index()

# 새로운 데이터프레임 생성
category_amount = pd.DataFrame({
    '연도': grouped_data['연'],
    '업종명 대분류': grouped_data['업종명 대분류'],
    '이용금액': grouped_data['이용금액']
})

# 결과 출력
category_amount
```

```python
# Figure 생성
plt.figure(figsize=(14.5, 4.5))

# seaborn을 사용한 꺾은선 그래프 시각화
plt.subplot(1, 2, 1)
sns.lineplot(x='연도', y='이용금액', hue='업종명 대분류', marker='o', data=category_amount,
             palette=['#4B3B75', '#415D82', '#347681', '#399785', '#59B273', '#9AC64D'])

# y축 레이블 포맷팅
def currency_formatter(x, pos):
    return '{:,.0f}원'.format(x)

formatter = FuncFormatter(currency_formatter)
plt.gca().yaxis.set_major_formatter(formatter)

plt.xticks(category_amount['연도'])

plt.xlabel('연도')
plt.ylabel('총 이용금액')
plt.title('업종별 카드 이용금액')
plt.legend(title='업종명 대분류', bbox_to_anchor=(1, 1.023))

# seaborn을 사용한 막대그래프 시각화
plt.subplot(1, 2, 2)
sns.barplot(x='업종명 대분류', y='이용금액',
            data=card_data.groupby('업종명 대분류')['이용금액'].count().reset_index(), palette='viridis')

# y축 레이블 포맷팅
def count_formatter(x, pos):
    return '{:,.0f}건'.format(x)

count_formatter = FuncFormatter(count_formatter)
plt.gca().yaxis.set_major_formatter(count_formatter)  # 수정된 부분

plt.xlabel('업종명 대분류')
plt.ylabel('이용건수')
plt.title('업종별 카드 이용건수')
plt.xticks(rotation=90)  # x축 레이블 회전

plt.tight_layout()
plt.show()
```

```python
# 업종명 대분류가 소매업인 데이터 추출
retail_data = card_data[card_data['업종명 대분류'] == '소매업']

# 연도별 업종명 이용금액 상위 5개 추출
top_5_categories_by_year = retail_data.groupby(['연', '업종명'])['이용금액'].sum().reset_index()
top_5_categories_by_year = top_5_categories_by_year.sort_values(by=['연', '이용금액'], ascending=[True, False])
top_5_categories_by_year = top_5_categories_by_year.groupby('연').head(5)

# 꺾은선 그래프 시각화
plt.figure(figsize=(8, 4))
sns.lineplot(x='연', y='이용금액', hue='업종명', marker='o', data=top_5_categories_by_year, palette='viridis')

# y축 레이블 포맷팅
def currency_formatter(x, pos):
    return '{:,.0f}원'.format(x)

formatter = FuncFormatter(currency_formatter)
plt.gca().yaxis.set_major_formatter(formatter)

plt.xticks(category_amount['연도'])

plt.xlabel('연도')
plt.ylabel('이용금액')
plt.title('연도별 소매업 중 상위 5개 업종 이용금액')
plt.legend(title='업종명', bbox_to_anchor=(1, 1.015))
plt.show()
```

```python
# 업종명이 체인화 편의점이거나 슈퍼마켓인 행만 추출
month_data = card_data[card_data['업종명'].isin(['체인화 편의점', '슈퍼마켓'])]

# 이용자 구분에 따른 빈도수 계산
user_type_counts = month_data['이용자 구분'].value_counts()

# 파이 그래프 시각화
fig, axes = plt.subplots(2, 2, figsize=(10, 7))

# 첫 번째 subplot: 이용자 구분 파이 그래프
ax1 = axes[0, 0]
ax1.pie(user_type_counts, labels=None, autopct='%1.1f%%', startangle=90, colors=green_palette,
        wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}, pctdistance=0.85)
ax1.legend(user_type_counts.index, title='이용자 구분', loc='center left', bbox_to_anchor=(0.7, 0.85))
ax1.set_title('체인화 편의점 및 슈퍼마켓 이용자 구분')

# 두 번째 subplot: 성별 파이 그래프
user_type_counts = month_data['성별'].value_counts()
ax2 = axes[0, 1]
ax2.pie(user_type_counts, labels=None, autopct='%1.1f%%', startangle=90, colors=green_palette,
        wedgeprops={'width': 0.7, 'edgecolor': 'w', 'linewidth': 5}, pctdistance=0.85)
ax2.legend(user_type_counts.index, title='성별', loc='center left', bbox_to_anchor=(0.7, 0.85))
ax2.set_title('체인화 편의점 및 슈퍼마켓 이용자 성별')

# 세 번째 subplot: 연령대별 이용건수 막대 그래프
user_type_counts = month_data['연령대'].value_counts().sort_index()
ax3 = axes[1, 0]
sns.barplot(x=user_type_counts.index, y=user_type_counts.values, palette='viridis', ax=ax3)
ax3.set_xlabel('연령대')
ax3.set_ylabel('이용건수')
ax3.set_title('연령대별 이용건수')

# 네 번째 subplot: 연령대별 이용금액 막대 그래프
age_grouped = month_data.groupby('연령대')['이용금액'].sum().reset_index()
ax4 = axes[1, 1]
sns.barplot(x='연령대', y='이용금액', data=age_grouped, palette='viridis', ax=ax4)
ax4.set_xlabel('연령대')
ax4.set_ylabel('이용금액')
ax4.set_title('연령대별 이용금액')
ax4.yaxis.set_major_formatter(FuncFormatter(lambda x, _: '{:,.0f}원'.format(x)))

plt.tight_layout()
plt.show()
```

```python
# 데이터 불러오기
fp = pd.read_csv('/content/제주도_유동인구.csv', encoding='cp949')
```

```python
# 사용하지 않는 컬럼 삭제
fp.drop(columns = ['년월', '데이터기준일자', '업종명', '이용자수', '이용금액'], axis=1, inplace=True)

# '읍면동명' 열의 이름을 '읍명'으로 변경
fp = fp.rename(columns={'읍면동명': '읍명'})

# 중복되는 행 제거
fp = fp[fp['읍명'].str.endswith('읍')].drop_duplicates(subset=['읍명']).reset_index(drop=True)
```

```python
# 막대그래프 시각화
plt.figure(figsize=(10, 6))
sns.barplot(x='읍명', y='총 유동인구', data=fp, palette='viridis')

plt.xlabel('읍명')
plt.ylabel('유동인구')
plt.title('읍별 유동인구')

# y축 레이블 포맷팅 (소수점 없애기)
plt.gca().get_yaxis().set_major_formatter(plt.FuncFormatter(lambda x, loc: "{:,}".format(int(x))))

plt.show()
```

```python
# 데이터 불러오기
mart = pd.read_csv('/content/제주관광공사_마을편의시설.csv', encoding='utf-8')

mart
```

```python
# 데이터 속성
mart.info()
```

```python
# 사용하지 않는 컬럼, 행 삭제
mart.drop(columns = ['인허가일자', '상세영업상태코드', '상세영업상태명', '소재지전화번호', '데이터기준일자'], axis=1, inplace=True)

# 데이터 출력
mart
```

```python
# 편의시설유형이 편의점이거나 마트인 행만 추출
mart_data = mart[mart['편의시설유형'].isin(['편의점', '마트'])].reset_index(drop=True)

mart_data
```

```python
# 지도 생성
map = folium.Map(location=[33.3628, 126.5338], zoom_start=11, tiles='CartoDB dark_matter')

# '마트'와 '편의점'을 구분하여 CircleMarker로 표시
for name, lat, lng, facility_type in zip(mart_data.편의시설명, mart_data.위도, mart_data.경도, mart_data.편의시설유형):
    color = 'yellow' if facility_type == '마트' else 'red' if facility_type == '편의점' else None

    folium.CircleMarker(
        location=[lat, lng],
        radius=1,
        color=color,
        fill=True,
        fill_color=color,
        fill_opacity=0.3,
        popup=name
    ).add_to(map)

# 결과 지도 출력
map
```

```python
# 지도 생성
map = folium.Map(location=[33.3628, 126.5338], zoom_start=11)

# MarkerCluster 객체 생성
marker_cluster = MarkerCluster().add_to(map)

# '마트'와 '편의점'을 구분하여 마커를 MarkerCluster에 추가
for name, lat, lng, facility_type in zip(mart_data.편의시설명, mart_data.위도,
                                         mart_data.경도, mart_data.편의시설유형):
    color = 'blue' if facility_type == '마트' else 'orange' if facility_type == '편의점' else 'red'
    folium.Marker(
        location=[lat, lng],
        popup=name,
        icon=folium.Icon(color=color)
    ).add_to(marker_cluster)

# 읍면동 경계 GeoJSON 파일 로드 및 추가
geojson_file = "/content/hangjeongdong_제주특별자치도.geojson"
folium.GeoJson(geojson_file, name='geojson_layer',
               style_function=lambda feature: {'weight': 0.7}).add_to(map)

# 결과 지도 출력
map
```

```python
# 읍별 이용금액 계산
area = month_data.groupby('읍면동명')['이용금액'].sum().reset_index()

# '읍'으로 끝나는 행만 선택
filtered_area = area[area['읍면동명'].str.endswith('읍')]

# 이용금액을 기준으로 내림차순 정렬
filtered_area_sorted = filtered_area.sort_values(by='이용금액', ascending=False)

# 가로 막대그래프 시각화
plt.figure(figsize=(10, 6))
sns.barplot(x='이용금액', y='읍면동명', data=filtered_area_sorted, palette='viridis')

plt.xlabel('이용금액')
plt.ylabel('읍명')
plt.title('읍별 이용금액')

# x축 레이블 포맷팅
def currency_formatter(x, pos):
    return '{:,.0f}원'.format(x)

formatter = FuncFormatter(currency_formatter)
plt.gca().xaxis.set_major_formatter(formatter)

plt.show()
```

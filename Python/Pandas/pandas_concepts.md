# 몰랐던 개념 정리

## 1. Series vs DataFrame

**Series** = 열(컬럼) 하나 = 엑셀의 열 하나

```
0    John
1    Jenny
2    Nate
Name: name
```

**DataFrame** = 열 여러 개를 옆으로 붙여놓은 것 = 엑셀 시트 전체

```
    name  age        job
0   John   20    student
1  Jenny   30  developer
2   Nate   30    teacher
```

> DataFrame은 Series의 결합체다.
> `df.job` 처럼 열 하나를 꺼내면 다시 Series가 된다.

## 2. df.head() / df.tail()

처음 또는 마지막 N개 행을 보여준다. 기본값은 5개.

```python
df.head()    # 처음 5개
df.head(10)  # 처음 10개
df.tail()    # 마지막 5개
df.tail(3)   # 마지막 3개
```

## 3. read_csv()

csv뿐 아니라 txt도 읽어온다. 구분자(sep)를 지정하면 어떤 텍스트 파일이든 가능.

```python
pd.read_csv('file.csv')               # 기본 (쉼표 구분)
pd.read_csv('file.txt', sep='\t')          # 탭 구분 txt
pd.read_csv('file.txt', delimiter='\t')   # sep와 동일, 별칭
pd.read_csv('file.csv', header=None)  # 헤더 없을 때
pd.read_csv('file.csv', index_col=0)  # 첫 번째 열을 인덱스로
pd.read_csv('file.csv', encoding='utf-8')  # 한글 파일은 encoding 필수
```

> 엑셀 파일은 `pd.read_excel('file.xlsx')` 따로 있음.

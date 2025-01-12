# Big Max Index

Nasz projekt skupiał się na przeprowadzeniu pełnego procesu ETL (Extract-Transform-Load), który obejmował pozyskanie danych, ich przekształcenie oraz załadowanie do odpowiednich narzędzi analitycznych. Celem analizy było zbadanie cen Big Maca w różnych krajach, a także identyfikacja wzorców i różnic w tych cenach. Skupiliśmy się na takich aspektach jak siła nabywcza, zmienność kursów walutowych oraz wpływ czynników ekonomicznych na ceny. Nasza analiza pozwoliła na głębsze zrozumienie, jak ceny Big Maca różnią się w zależności od regionu i jak odzwierciedlają one szersze zjawiska gospodarcze w poszczególnych krajach.

## Co to "Big Mac Index"?

**Big Mac Index** to nieformalne narzędzie stworzone przez tygodnik _The Economist_, które służy do porównania siły nabywczej różnych walut na podstawie ceny Big Maca w różnych krajach. Indeks zakłada, że cena tego samego produktu (Big Maca) powinna być podobna na całym świecie, jeśli uwzględni się różnice w kursach walutowych i kosztach życia. Różnice w cenach Big Maca między krajami mogą wskazywać na przewartościowanie lub niedowartościowanie lokalnych walut w stosunku do dolara amerykańskiego. Big Mac Index jest więc prostym, ale popularnym sposobem oceny siły nabywczej oraz kursów walutowych, choć nie jest to narzędzie oparte na skomplikowanych modelach ekonomicznych.

## Przetwarzanie surowych danych
Proces przetwarzania danych w projekcie obejmował następujące kroki:

1. **Wczytanie danych**: Dane o cenach Big Maca zostały zaimportowane z trzech różnych plików CSV, zawierających dane podstawowe, źródłowe oraz pełny indeks Big Maca.   
2. **Standaryzacja danych**: Z wyjściowych plików wyodrębniono kluczowe kolumny, takie jak data, kod kraju, kod waluty, cena lokalna oraz kurs wymiany dolara. Wszystkie dane zostały ujednolicone pod względem formatu i oznaczone źródłem pochodzenia.
3. **Łączenie danych**: Znormalizowane dane z różnych plików zostały scalone w jeden zbiór, eliminując duplikaty.
4. **Przygotowanie danych o PKB per capita**: Na potrzeby analizy, symulowano dane o PKB per capita dla wybranych krajów i lat, które zapisano w osobnym pliku CSV.
5. **Integracja danych**: Połączono dane o cenach Big Maca z danymi o PKB per capita, korzystając z kluczowych atrybutów takich jak kod kraju (ISO A3) i rok. Dodano także nową kolumnę `big_mac_to_gdp_ratio`, wskazującą relację ceny Big Maca do PKB per capita.
6. **Eksport danych**: Finalne zbiory danych zapisano w plikach CSV:
- Oczyszczone dane o Big Macu: `merged_big_mac_cleaned.csv`
- Symulowane dane o PKB per capita: `oecd_gdp_per_capita.csv`
- Zintegrowane dane: `merged_data_with_gdp.csv`

Ten proces pozwolił na utworzenie kompleksowego zestawu danych do dalszej analizy wskaźnika Big Mac Index w kontekście gospodarek krajowych.

## Zapis w relacyjnej bazie danych

W tym etapie projektowania wykonano kroki związane z budową struktury bazy danych oraz przeprowadzaniem analiz na zintegrowanych danych. Przygotowana struktura bazy danych oraz przeprowadzone analizy umożliwiły efektywne badanie zależności między cenami Big Maca a wskaźnikami gospodarczymi, dostarczając wartościowych wniosków do dalszych badań.

#### Utworzenie struktury bazy danych

Stworzono tabelę `big_mac_data` zawierającą kluczowe pola.

| **Nazwa pola**           | **Typ danych**       | **Opis**                                    |
| ------------------------ | -------------------- | ------------------------------------------- |
| `id`                     | `SERIAL PRIMARY KEY` | Unikalny identyfikator rekordu.             |
| `date`                   | `DATE`               | Data dla danego rekordu.                    |
| `iso_a3`                 | `CHAR(3)`            | Kod kraju (3-literowy format ISO).          |
| `currency_code`          | `CHAR(3)`            | Kod waluty (3-literowy format ISO).         |
| `name`                   | `TEXT`               | Nazwa kraju.                                |
| `local_price`            | `NUMERIC(10, 2)`     | Lokalna cena Big Maca (w walucie krajowej). |
| `dollar_ex`              | `NUMERIC(10, 4)`     | Kurs wymiany dolara (lokalna waluta/USD).   |
| `gdp_per_capita`         | `NUMERIC(12, 2)`     | PKB na mieszkańca w lokalnej walucie.       |
| `big_mac_to_gdp_ratio`   | `NUMERIC(12, 6)`     | Stosunek ceny Big Maca do PKB per capita.   |
| `purchasing_power_index` | `NUMERIC(10, 4)`     | Wskaźnik siły nabywczej (dodany później).   |


#### Import danych

Zaimportowano dane z przygotowanego pliku CSV (`merged_data_with_gdp.csv`) do tabeli bazy danych za pomocą polecenia `COPY`.

#### Czyszczenie danych

Usunięto duplikaty na podstawie pól takich jak:
- Data
- Kod kraju
- Kod waluty
- Nazwa kraju
- Rok (`year`).

#### Dodanie wskaźników analitycznych

**Wskaźnik siły nabywczej (PPI)**: Obliczono stosunek lokalnej ceny Big Maca do kursu wymiany dolara.

**Normalizacja PKB per capita**: Przeliczono wartości PKB na dolary amerykańskie dla zapewnienia porównywalności.

#### Usunięcie niekompletnych danych

Wiersze zawierające brakujące wartości w kluczowych kolumnach (np. `local_price`, `dollar_ex`, `gdp_per_capita`) zostały usunięte.

#### Przeprowadzenie analiz

- Wyliczono średni wskaźnik siły nabywczej (`purchasing_power_index`) dla krajów w poszczególnych latach.
- Porównano kraje o najwyższym i najniższym PKB per capita w roku 2020.
- Zbadano zmiany wskaźnika siły nabywczej i stosunku ceny Big Maca do PKB w czasie dla wybranego kraju (np. Australia).
- Zidentyfikowano kraje o wartościach siły nabywczej znacznie odbiegających od średniej (powyżej/poniżej 2 odchyleń standardowych).
- Przeanalizowano korelację między wskaźnikiem siły nabywczej a PKB per capita, wykazując umiarkowaną dodatnią zależność.

## Analiza i Wizualizacja



![[Pasted image 20250112114436.png]]

Wykres słupkowy przedstawia średnie wartości PKB dla każdego kraju. Słupki w tym wykresie ilustrują wartość średniego PKB w badanym okresie. Wartości są porównywalne między krajami, co pozwala na szybkie zauważenie różnic w poziomie PKB.

- **Australia, Kanada i Szwajcaria**: Wykres pokazuje stosunkowo wysokie średnie PKB w tych krajach, co jest efektem silnej gospodarki, stabilności ekonomicznej oraz wysokiej siły nabywczej. PKB w tych krajach jest wyższe niż w innych krajach w zestawieniu.    
- **Brazylia i Argentyna**: Słupki dla Brazylii i Argentyny są zauważalnie niższe, co odzwierciedla ich mniej rozwinięte gospodarki. Szczególnie w Argentynie, średnie PKB może być niższe z powodu niestabilności gospodarczej, wysokiej inflacji i zmniejszonej siły nabywczej.

---

![[Pasted image 20250112114347.png]]

Wykres przedstawia porównanie stosunku ceny Big Maca do PKB dla wybranych krajów w latach 2000 i 2020. Jak możemy zauważyć, stosunek ten zwiększył się dla wszystkich krajów, jednak szczególnym przypadkiem jest Argentyna, gdzie możemy zaobserwować ponad dwudziestokrotny wzrost tego wskaźnika. Może to być spowodowane wieloma czynnikami, w tym wysoką inflacją, deprecjacją lokalnej waluty oraz niestabilnością gospodarczą w Argentynie, co doprowadziło do gwałtownego wzrostu cen towarów, w tym Big Maca, w porównaniu do wzrostu PKB.

Wzrost tego stosunku w Argentynie może również wskazywać na wyższe koszty importu związane z silną zależnością od zagranicznych surowców oraz negatywne skutki kryzysów finansowych i politycznych, które miały miejsce w tym okresie. Tak znaczący wzrost wskazuje również na zmniejszenie siły nabywczej lokalnej waluty, co sprawia, że mieszkańcy Argentyny muszą wydawać znacznie więcej na produkty oparte na imporcie, w tym na popularny Big Mac.

---

![[Pasted image 20250112114358.png]]

Wykres przedstawia zmienność kursu dolara w czasie dla wybranych krajów: Argentyny, Australii, Brazylii, Kanady i Szwajcarii. Zauważalne są różnice w dynamice kursu dolara w poszczególnych krajach, co odzwierciedla ich różne podejścia do polityki monetarnej, inflacji oraz wpływ globalnych czynników gospodarczych.

Dla Argentyny, zmienność kursu dolara jest wyraźnie większa, co może być efektem niestabilnych warunków gospodarczych, wysokiej inflacji i wahań zaufania do lokalnych walut. Częste skoki kursu dolara w tych krajach mogą wskazywać na niestabilność makroekonomiczną, zmiany polityczne oraz reakcje rynków finansowych na lokalne kryzysy.

---

![[Pasted image 20250112114424.png]]

Wykres przedstawia średnią kroczącą wskaźnika siły nabywczej dla wybranych krajów: Kanady, Brazylii, Szwajcarii, Australii oraz Argentyny. Z analizy danych wynika, że wszystkie kraje charakteryzują się ogólnym trendem wzrostowym wskaźnika, jednak ich dynamika oraz amplituda fluktuacji różnią się znacząco.

Dla krajów rozwiniętych, takich jak Szwajcaria, trend wzrostowy jest bardziej stabilny, a okresy wzrostu przeplatają się z momentami stagnacji, co odzwierciedla większą przewidywalność gospodarki oraz stabilność waluty. W przypadku krajów takich jak Argentyna widoczne są znacznie bardziej drastyczne zmiany wskaźnika, co może wynikać z niestabilności gospodarczej, inflacji czy gwałtownych zmian kursu walutowego.

---

![[Pasted image 20250112114449.png]]

Wykres typu boxplot przedstawia rozkład wskaźnika siły nabywczej w wybranych krajach (Argentyna, Australia, Brazylia, Kanada, Szwajcaria). Dzięki niemu można zaobserwować różnice w medianie oraz rozpiętości danych w poszczególnych krajach. Szwajcaria charakteryzuje się najwyższą medianą wskaźnika siły nabywczej, co wskazuje na jej silną walutę i wysoką siłę nabywczą w stosunku do dolara. Argentyna, z kolei, wykazuje największe rozproszenie danych, co sugeruje dużą zmienność siły nabywczej w czasie. Inne kraje, takie jak Kanada czy Australia, mają bardziej stabilne rozkłady z mniejszą rozpiętością wartości.

---

![[Pasted image 20250112114330.png]]

Wykres przedstawia rozkład różnic między siłą nabywczą a lokalną ceną Big Maca dla wybranych krajów: Argentyny, Australii, Brazylii, Kanady i Szwajcarii. Widoczne są wyraźne cechy bimodalności dla wszystkich rozkładów, co oznacza, że w każdym przypadku występują dwa dominujące zakresy wartości. Sugeruje to istnienie dwóch różnych grup obserwacji, które mogą wynikać z czynników ekonomicznych, takich jak zmiany kursów walutowych, różnice w kosztach życia czy różne okresy gospodarcze.

Rozkłady dla Kanady i Szwajcarii charakteryzują się mniejszym skupieniem wartości wokół środka, podczas gdy kraje takie jak Brazylia i Argentyna wykazują mniej rozciągnięte rozkłady, co może wskazywać na większe mniejsze danych. Dodatkowo, różnice są bardziej przesunięte w stronę wartości ujemnych w krajach o relatywnie niższej sile nabywczej, takich jak Argentyna, co obrazuje mniejszą równowagę między lokalną ceną a siłą nabywczą.

Wykres pozwala na analizę różnic w rozkładach dla poszczególnych krajów oraz identyfikację możliwych czynników wpływających na tę zmienność.

---

![[Pasted image 20250112114411.png]]

Heatmapa przedstawia współczynniki korelacji między kluczowymi zmiennymi ekonomicznymi, takimi jak PKB na mieszkańca, lokalna cena Big Maca, kurs wymiany dolara, stosunek ceny Big Maca do PKB oraz wskaźnik siły nabywczej. Wyraźnie widać silną dodatnią korelację między lokalną ceną Big Maca a kursem wymiany dolara (0.970) oraz między ceną Big Maca a stosunkiem ceny do PKB (0.968). Pokazuje to, że cena Big Maca i kurs walutowy mają istotny wpływ na analizowane wskaźniki.

Jednocześnie zauważono słabą korelację wskaźnika siły nabywczej z lokalną ceną Big Maca (0.085) oraz kursem wymiany dolara (0.052), co wskazuje na brak istotnego związku między tymi zmiennymi. Z kolei dodatnia korelacja wskaźnika siły nabywczej z PKB na mieszkańca (0.379) sugeruje, że kraje o wyższym PKB per capita mają tendencję do wyższej siły nabywczej, choć zależność ta nie jest bardzo silna.

Wykres pozwala lepiej zrozumieć zależności między analizowanymi zmiennymi i pomaga wyciągnąć wnioski o wpływie poszczególnych czynników na wskaźniki ekonomiczne.

## Kod

#### Wstępne przetwarzanie danych

```
# Krok 1: Wczytanie danych z plików CSV
import pandas as pd

file_paths = {
    "big_mac": "/mnt/data/big mac.csv",
    "big_mac_source": "/mnt/data/big-mac-source-data.csv",
    "big_mac_full_index": "/mnt/data/big-mac-full-index.csv"
}

big_mac_df = pd.read_csv(file_paths["big_mac"])
big_mac_source_df = pd.read_csv(file_paths["big_mac_source"])
big_mac_full_index_df = pd.read_csv(file_paths["big_mac_full_index"])

# Krok 2: Standaryzacja danych (wyodrębnienie kluczowych kolumn)
key_columns = ['date', 'iso_a3', 'currency_code', 'name', 'local_price', 'dollar_ex']

big_mac_standard = big_mac_df[key_columns]
big_mac_source_standard = big_mac_source_df[key_columns]
big_mac_full_index_standard = big_mac_full_index_df[key_columns]

big_mac_standard['source'] = 'big_mac'
big_mac_source_standard['source'] = 'big_mac_source'
big_mac_full_index_standard['source'] = 'big_mac_full_index'

# Krok 3: Połączenie danych
merged_data = pd.concat([big_mac_standard, big_mac_source_standard, big_mac_full_index_standard], ignore_index=True)

# Krok 4: Czyszczenie danych
merged_data_cleaned = merged_data.drop_duplicates()

# Krok 5: Eksport oczyszczonych danych
merged_data_cleaned.to_csv("/mnt/data/merged_big_mac_cleaned.csv", index=False)

# Krok 6: Wczytanie i przygotowanie danych o PKB per capita
import numpy as np

countries = ["ARG", "AUS", "BRA", "CAN", "CHE"]
years = range(2000, 2025)
data = []

for year in years:
    for country in countries:
        data.append({
            "country": country,
            "year": year,
            "gdp_per_capita": np.random.uniform(5000, 70000)  # Symulowane dane
        })

gdp_per_capita_df = pd.DataFrame(data)
gdp_per_capita_df.to_csv("/mnt/data/oecd_gdp_per_capita.csv", index=False)

# Krok 7: Integracja danych o Big Macu i PKB
big_mac_cleaned_df = pd.read_csv("/mnt/data/merged_big_mac_cleaned.csv")
gdp_per_capita_df = pd.read_csv("/mnt/data/oecd_gdp_per_capita.csv")

big_mac_cleaned_df['year'] = pd.to_datetime(big_mac_cleaned_df['date']).dt.year

merged_data_with_gdp = pd.merge(
    big_mac_cleaned_df,
    gdp_per_capita_df.rename(columns={"country": "iso_a3"}),
    how="inner",
    on=["iso_a3", "year"]
)

merged_data_with_gdp['big_mac_to_gdp_ratio'] = (
    merged_data_with_gdp['local_price'] / merged_data_with_gdp['gdp_per_capita']
)

# Krok 8: Eksport zintegrowanych danych
merged_data_with_gdp.to_csv("/mnt/data/merged_data_with_gdp.csv", index=False)
```

#### Implementacja Struktury Danych i ETL
```
---- POSTGRES ----
---- UTWORZENIE STRUKTURY BAZY DANYCH I PROCESÓW ETL ----

- 1: Utworzenie głównej tabeli do przechowywania danych

DROP TABLE IF EXISTS big_mac_data;
CREATE TABLE big_mac_data (
    id SERIAL PRIMARY KEY,
    date DATE NOT NULL,
    iso_a3 CHAR(3) NOT NULL,
    currency_code CHAR(3) NOT NULL,
    name TEXT NOT NULL,
    local_price NUMERIC(10, 2) NOT NULL,
    dollar_ex NUMERIC(10, 4) NOT NULL,
    source TEXT NOT NULL,
    year INTEGER NOT NULL,
    gdp_per_capita NUMERIC(12, 2) NOT NULL,
    big_mac_to_gdp_ratio NUMERIC(12, 6) NOT NULL
);

- 2: Wczytanie danych do tabeli(UŻYJ WŁASNEJ ŚCIEŻKI)

COPY big_mac_data(date, iso_a3, currency_code, name, local_price, dollar_ex, source, year, gdp_per_capita, big_mac_to_gdp_ratio)
FROM 'C:\Users\Adrian\Desktop\merged_data_with_gdp.csv'
DELIMITER ','
CSV HEADER;

- 3: Oczyszczenie danych 

DELETE FROM big_mac_data
WHERE id NOT IN (
    SELECT MIN(id) FROM big_mac_data
    GROUP BY date, iso_a3, currency_code, name, year
);

- 4: Dodanie kolumny do przechowywania wskaźnika siły nabywczej (PPI)

ALTER TABLE big_mac_data
ADD COLUMN purchasing_power_index NUMERIC(10, 4);

- 5: Obliczenie wskaźnika siły nabywczej 

UPDATE big_mac_data
SET purchasing_power_index = local_price / dollar_ex;

- 6: Normalizacja danych 

UPDATE big_mac_data
SET gdp_per_capita = gdp_per_capita / dollar_ex;

- 7: Usunięcie wierszy z brakującymi lub nieprawidłowymi danymi

DELETE FROM big_mac_data
WHERE local_price IS NULL OR dollar_ex IS NULL OR gdp_per_capita IS NULL;

---- ANALIZY NA TABELI ----

-- Wyświetlenie wierszy tabeli

SELECT * FROM big_mac_data;


-- Analiza siły nabywczej między krajami

SELECT name, iso_a3, year, AVG(purchasing_power_index) AS avg_ppi
FROM big_mac_data
GROUP BY name, iso_a3, year
ORDER BY avg_ppi DESC;


-- Porównanie krajów o najwyższym i najniższym PKB na mieszkańca w 2020

SELECT name, iso_a3, year, gdp_per_capita, purchasing_power_index
FROM big_mac_data
WHERE year = 2020
ORDER BY gdp_per_capita DESC

-- Analiza zmian wskaźnika siły nabywczej lub stosunku ceny Big Maca do PKB na mieszkańca w czasie dla konkretnego kraju (Australia)

SELECT date, purchasing_power_index, big_mac_to_gdp_ratio
FROM big_mac_data
WHERE iso_a3 = 'AUS'
ORDER BY date;



-- Identyfikacja krajów, które mają dużo wyższe lub dużo niższe wartości wskaźnika siły nabywczej w porównaniu do średniej

SELECT name, iso_a3, year, purchasing_power_index
FROM big_mac_data
WHERE purchasing_power_index > (
    SELECT AVG(purchasing_power_index) + 2 * STDDEV(purchasing_power_index) FROM big_mac_data
)
OR purchasing_power_index < (
    SELECT AVG(purchasing_power_index) - 2 * STDDEV(purchasing_power_index) FROM big_mac_data
);


-- Zbadanie, czy istnieje korelacja między wskaźnikiem siły nabywczej a PKB na mieszkańca (wynik to 0.378, więc jest dodatnia korelacja. Wzrost jednej zmiennej wiąże się z pewnym wzrostem drugiej, jednak zależność ta nie jest silna. Chociaż istnieje trend, nie jest on wystarczająco wyraźny, aby mówić o silnym związku.)

SELECT CORR(purchasing_power_index, gdp_per_capita) AS correlation
FROM big_mac_data;
```

#### Wizualizacje

```
import os
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

file_path = 'data\merged_data_with_gdp_ready_for_analysis.csv'
data = pd.read_csv(file_path)

output_dir = "visualizations"
os.makedirs(output_dir, exist_ok=True)
data = data.sort_values(by=['name', 'year'])

# 1. Wykres pudełkowy PKB na mieszkańca w krajach z dedykowanymi kolorami

plt.figure(figsize=(12, 6))

country_colors = {
    'Canada': '#8B0000',
    'Brazil': '#228B22',
    'Switzerland': '#FF0000',
    'Australia': '#0000FF',
    'Argentina': '#87CEEB'
}

sns.boxplot(
    data=data,
    x='name',
    y='gdp_per_capita',
    palette=country_colors
)

plt.title('Rozkład PKB na mieszkańca w krajach (w dolarach)', fontsize=16)
plt.xlabel('Kraj', fontsize=12)
plt.ylabel('PKB na mieszkańca (USD)', fontsize=12)
plt.xticks(rotation=45)
plt.grid(True)

plt.savefig(os.path.join(output_dir, 'boxplot_gdp_per_capita_by_country_colored.png'))
plt.close()

# 2. Słupkowy wykres średniego PKB na mieszkańca w krajach

avg_gdp_by_country = data.groupby('name')['gdp_per_capita'].mean().sort_values()
plt.figure(figsize=(10, 8))
avg_gdp_by_country.plot(kind='barh', color='skyblue')
plt.title('Średnie PKB na mieszkańca w krajach (w dolarach)', fontsize=16)
plt.xlabel('PKB na mieszkańca (USD)', fontsize=12)
plt.ylabel('Kraj', fontsize=12)
plt.grid(axis='x')
plt.savefig(os.path.join(output_dir, 'bar_avg_gdp_per_capita.png'))
plt.close()

# 3. Słupkowy wykres średniej ceny Big Maca w dolarach

data['price_in_usd'] = data['local_price'] / data['dollar_ex']
avg_price_by_country = data.groupby('name')['price_in_usd'].mean().sort_values()
plt.figure(figsize=(10, 8))
avg_price_by_country.plot(kind='barh', color='orange')
plt.title('Średnia cena Big Maca w dolarach w różnych krajach', fontsize=16)
plt.xlabel('Średnia cena (USD)', fontsize=12)
plt.ylabel('Kraj', fontsize=12)
plt.grid(axis='x')
plt.savefig(os.path.join(output_dir, 'bar_avg_local_price_in_usd.png'))
plt.close()

# 4. Wykres pudełkowy wskaźnika siły nabywczej w różnych krajach

plt.figure(figsize=(12, 6))
sns.boxplot(data=data, x='name', y='purchasing_power_index')
plt.title('Rozkład wskaźnika siły nabywczej w różnych krajach', fontsize=16)
plt.xlabel('Kraj', fontsize=12)
plt.ylabel('Wskaźnik siły nabywczej', fontsize=12)
plt.xticks(rotation=45)
plt.grid(True)
plt.savefig(os.path.join(output_dir, 'boxplot_purchasing_power_index_by_country.png'))
plt.close()

# 5. Heatmapa korelacji między zmiennymi

correlation_matrix = data[['gdp_per_capita', 'local_price', 'dollar_ex', 'big_mac_to_gdp_ratio', 'purchasing_power_index']].corr()
renamed_columns = {
    'gdp_per_capita': 'PKB na mieszkańca (USD)',
    'local_price': 'Lokalna cena Big Maca',
    'dollar_ex': 'Kurs wymiany dolara',
    'big_mac_to_gdp_ratio': 'Stosunek ceny Big Maca do PKB',
    'purchasing_power_index': 'Wskaźnik siły nabywczej'
}
correlation_matrix.rename(columns=renamed_columns, index=renamed_columns, inplace=True)

plt.figure(figsize=(12, 10))
sns.heatmap(
    correlation_matrix,
    annot=True,
    cmap='cividis',
    fmt=".3f",
    linewidths=0.5,
    cbar_kws={'label': 'Współczynnik korelacji'},
    annot_kws={'size': 10, 'weight': 'bold'}
)

plt.title('Heatmapa korelacji między zmiennymi', fontsize=18, weight='bold', pad=20)
plt.xticks(fontsize=12, rotation=45, ha='right')
plt.yticks(fontsize=12, rotation=0)
plt.xlabel('Zmienna', fontsize=14, weight='bold')
plt.ylabel('Zmienna', fontsize=14, weight='bold')

plt.tight_layout()
plt.savefig(os.path.join(output_dir, 'heatmap_correlation_between_variables.png'))
plt.close()

# 6. Liniowy wykres porównawczy wskaźnika siły nabywczej dla wybranych krajów w czasie

selected_countries = ['Canada', 'Brazil', 'Switzerland', 'Australia', 'Argentina']
plt.figure(figsize=(14, 8))

for country in selected_countries:
    country_data = data[data['name'] == country]
    moving_avg = country_data['purchasing_power_index'].rolling(window=3, min_periods=1).mean()
    plt.plot(
        country_data['year'],
        moving_avg,
        linestyle='--',
        label=f'{country} (średnia krocząca)'
    )

plt.title('Wskaźnik siły nabywczej w czasie dla wybranych krajów (średnia krocząca)', fontsize=18, weight='bold')
plt.xlabel('Rok', fontsize=14, weight='bold')
plt.ylabel('Wskaźnik siły nabywczej', fontsize=14, weight='bold')
plt.legend(title='Kraj', fontsize=10, title_fontsize=12)
plt.grid(True, linestyle='--', alpha=0.6)
plt.tight_layout()

plt.savefig(os.path.join(output_dir, 'lineplot_purchasing_power_index_moving_average.png'))
plt.close()


# 7. Porównanie stosunku ceny Big Maca do PKB w latach 2000 i 2020

selected_years = [2000, 2020]
selected_data = data[data['year'].isin(selected_years)]
selected_data = selected_data[selected_data['name'].isin(selected_countries)]
plt.figure(figsize=(14, 8))
selected_data_2020 = selected_data[selected_data['year'] == 2020]
selected_data_2020 = selected_data_2020.sort_values(by='big_mac_to_gdp_ratio', ascending=False)
sorted_countries = selected_data_2020['name'].unique()
selected_data['name'] = pd.Categorical(selected_data['name'], categories=sorted_countries, ordered=True)
selected_data = selected_data.sort_values(by=['name', 'year'])

sns.barplot(
    data=selected_data,
    x='name',
    y='big_mac_to_gdp_ratio',
    hue='year',
    palette='viridis'
)

for index, row in selected_data.iterrows():
    plt.text(
        x=row['name'],
        y=row['big_mac_to_gdp_ratio'] + 0.01,
        s=f"{row['big_mac_to_gdp_ratio']:.2f}",
        ha='center', fontsize=10
    )

plt.title('Porównanie stosunku ceny Big Maca do PKB w latach 2000 i 2020', fontsize=18, weight='bold')
plt.xlabel('Kraj', fontsize=14, weight='bold')
plt.ylabel('Stosunek ceny Big Maca do PKB', fontsize=14, weight='bold')
plt.legend(title='Rok', fontsize=10, title_fontsize=12)
plt.grid(axis='y', linestyle='--', alpha=0.6)
plt.tight_layout()

plt.savefig(os.path.join(output_dir, 'barplot_big_mac_to_gdp_ratio_comparison_2000_2020.png'))
plt.close()


# 8. Wykres słupkowy zmienności kursu dolara w czasie dla wybranych krajów

plt.figure(figsize=(16, 10))

for country in selected_countries:
    country_data = data[data['name'] == country]
    plt.plot(
        country_data['year'],
        country_data['dollar_ex'],
        marker='o',
        linestyle='-',
        label=country
    )

for country in selected_countries:
    country_data = data[data['name'] == country]
    max_year = country_data.loc[country_data['dollar_ex'].idxmax()]['year']
    max_value = country_data['dollar_ex'].max()
    plt.annotate(
        f'Max: {max_value:.2f}',
        xy=(max_year, max_value),
        xytext=(max_year, max_value + 0.5),
        arrowprops=dict(facecolor='black', arrowstyle='->'),
        fontsize=10, color='red'
    )

plt.title('Zmienność kursu dolara w czasie dla wybranych krajów', fontsize=20, weight='bold')
plt.xlabel('Rok', fontsize=14, weight='bold')
plt.ylabel('Kurs dolara (USD)', fontsize=14, weight='bold')
plt.legend(title='Kraj', fontsize=12, title_fontsize=14)
plt.grid(True, linestyle='--', alpha=0.6)
plt.tight_layout()

plt.savefig(os.path.join(output_dir, 'lineplot_dollar_exchange_selected_countries_enhanced.png'))
plt.close()

# 9. Rozkład różnic między siłą nabywczą a lokalną ceną Big Maca

data['price_diff'] = data['purchasing_power_index'] - data['price_in_usd']
plt.figure(figsize=(16, 8))
sns.kdeplot(
    data=data,
    x='price_diff',
    hue='name',
    palette='tab10',
    fill=True,
    alpha=0.6
)

plt.title('Rozkład różnic między siłą nabywczą a lokalną ceną Big Maca', fontsize=20, weight='bold')
plt.xlabel('Różnica siły nabywczej a ceny (USD)', fontsize=14, weight='bold')
plt.ylabel('Gęstość', fontsize=14, weight='bold')
plt.grid(axis='y', linestyle='--', alpha=0.6)
plt.tight_layout()

plt.savefig(os.path.join(output_dir, 'kde_price_diff_enhanced_country_color_mapping.png'))
plt.show()
```

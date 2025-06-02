# Analyzing Unicorn Companies

In this project, I have analyzed trends in high-growth companies to understand which industries are producing the highest valuations and the rate at which new high-value companies are emerging.  

The output of this analysis provides competitive insight into industry trends and how to better construct a portfolio looking forward.

---

## Database Tables

### `dates`

| Column      | Description                                 |
|-------------|---------------------------------------------|
| company_id  | A unique ID for the company.                 |
| date_joined | The date that the company became a unicorn. |
| year_founded| The year that the company was founded.       |

### `funding`

| Column          | Description                                 |
|-----------------|---------------------------------------------|
| company_id      | A unique ID for the company.                 |
| valuation       | Company value in US dollars.                  |
| funding         | The amount of funding raised in US dollars.  |
| select_investors| A list of key investors in the company.       |

### `industries`

| Column      | Description                              |
|-------------|------------------------------------------|
| company_id  | A unique ID for the company.              |
| industry    | The industry that the company operates in.|

### `companies`

| Column      | Description                                      |
|-------------|-------------------------------------------------|
| company_id  | A unique ID for the company.                      |
| company    | The name of the company.                          |
| city       | The city where the company is headquartered.     |
| country    | The country where the company is headquartered.  |
| continent  | The continent where the company is headquartered.|

---

## Code

WITH top_industries AS (
    SELECT i.industry,
           COUNT(i.*)
    FROM industries AS i
    INNER JOIN dates AS d
    ON i.company_id = d.company_id
    WHERE EXTRACT(year FROM d.date_joined) IN (2019, 2020, 2021)
    GROUP BY i.industry
    ORDER BY count DESC
   LIMIT 3
),

valuation AS (
    SELECT i.industry,
           EXTRACT(YEAR FROM d.date_joined) AS year,
           COUNT(i.*) AS num_unicorns,
           AVG(f.valuation) AS average_valuation
    FROM industries AS i
    INNER JOIN funding AS f
    ON i.company_id = f.company_id
    INNER JOIN dates AS d
    ON f.company_id = d.company_id
    GROUP BY industry, year
)

SELECT industry, valuation.year, valuation.num_unicorns, ROUND(AVG(average_valuation/1000000000),2) AS average_valuation_billions
FROM valuation
	WHERE year IN (2019,2020,2021)
	AND industry IN (SELECT industry
					FROM top_industries)
	GROUP BY industry, num_unicorns, year
	ORDER BY year DESC, num_unicorns DESC;

---

## Output

| Index | Industry                        | Year | Number of Unicorns| Average Valuation (Billions USD) |
|-------|---------------------------------|------|-------------------|----------------------------------|
| 0     | Fintech                         | 2021 | 138               | 2.75                             |
| 1     | Internet software & services    | 2021 | 119               | 2.15                             |
| 2     | E-commerce & direct-to-consumer | 2021 | 47                | 2.47                             |
| 3     | Internet software & services    | 2020 | 20                | 4.35                             |
| 4     | E-commerce & direct-to-consumer | 2020 | 16                | 4.00                             |
| 5     | Fintech                         | 2020 | 15                | 4.33                             |
| 6     | Fintech                         | 2019 | 20                | 6.80                             |
| 7     | Internet software & services    | 2019 | 13                | 4.23                             |
| 8     | E-commerce & direct-to-consumer | 2019 | 12                | 2.58                             |

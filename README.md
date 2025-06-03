# Analyzing Unicorn Companies

This comprehensive dataset includes:

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

-- Get the top 3 industries by number of unicorns that joined in 2019-2021
WITH top_industries AS (
    SELECT 
        i.industry,
        COUNT(*) AS unicorn_count
    FROM industries AS i
    INNER JOIN dates AS d ON i.company_id = d.company_id
    WHERE EXTRACT(YEAR FROM d.date_joined) IN (2019, 2020, 2021)
    GROUP BY i.industry
    ORDER BY unicorn_count DESC
    LIMIT 3
),

-- Calculate yearly number of unicorns and average valuation per industry
valuation AS (
    SELECT 
        i.industry,
        EXTRACT(YEAR FROM d.date_joined) AS year,
        COUNT(*) AS num_unicorns,
        AVG(f.valuation) AS average_valuation
    FROM industries AS i
    INNER JOIN funding AS f ON i.company_id = f.company_id
    INNER JOIN dates AS d ON f.company_id = d.company_id
    GROUP BY i.industry, year
)

-- Select final results for top industries and years of interest,
-- converting average valuation to billions USD and rounding to 2 decimals
SELECT 
    v.industry, 
    v.year, 
    v.num_unicorns, 
    ROUND(AVG(v.average_valuation) / 1000000000, 2) AS average_valuation_billions
FROM valuation v
WHERE v.year IN (2019, 2020, 2021)
  AND v.industry IN (SELECT industry FROM top_industries)
GROUP BY v.industry, v.year, v.num_unicorns
ORDER BY v.year DESC, v.num_unicorns DESC;


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

# Examen Sql Avancé B2

### Exercice 1 - Commandes et employés

```sql
SELECT oi.order_id, e.first_name, e.last_name
FROM order_items oi
JOIN employees e ON e.employee_id = oi.employee_id;
```

### Exercice 2 — Commandes avec produits chers

```sql
SELECT distinct oi.order_id
FROM order_items oi
JOIN products p ON p.product_id = oi.product_id
WHERE p.price > 500
ORDER BY oi.order_id;
```

### Exercice 3 — Ventes par pays

```sql
SELECT distinct r.country, count(r.country) OVER(partition by r.country) AS nbr_order,
sum(oi.quantity) OVER(partition by r.country) AS nbr_product
FROM regions r
JOIN clients c ON c.region_id = r.region_id
JOIN orders o ON o.client_id = c.client_id
JOIN order_items oi ON oi.order_id = o.order_id;
```

### Exercice 4 — Employés et clients (même région)

```sql
SELECT order_id, client_name, first_name, last_name
FROM (
SELECT c.client_name, c.region_id AS c_region_id , oi.employee_id, oi.order_id
FROM clients c
JOIN orders o ON c.client_id = o.client_id
JOIN order_items oi ON oi.order_id = o.order_id
) relation
JOIN employees e ON relation.employee_id = e.employee_id
WHERE c_region_id = e.region_id
```

### Exercice 5 – Nouveaux employés par pays

```sql
WITH employees_per_country AS (
    SELECT e.employee_id, e.first_name, e.last_name, r.region_name, r.country
    FROM employees e
    JOIN regions r ON e.region_id = r.region_id
    LEFT JOIN order_items oi ON e.employee_id = oi.employee_id
    WHERE oi.order_item_id IS NULL
)
SELECT first_name, last_name, region_name, count(*) over(partition by country) as nbr
FROM employees_per_country
```

### Exercice 6 — Chiffre d’affaires par employé

```sql
SELECT distinct first_name, last_name,count(e.employee_id) OVER(partition by e.employee_id) as nbr_commande,
sum(quantity * price) OVER(partition by e.employee_id) as ca
FROM (
SELECT oi.order_id, oi.employee_id, oi.product_id, oi.quantity, p.price
FROM order_items oi
LEFT JOIN products p ON p.product_id = oi.product_id
) datas
LEFT JOIN employees e ON datas.employee_id = e.employee_id
```

### Exercice 7 — Commandes au-dessus de la moyenne

```sql
SELECT oi.order_id
FROM order_items oi
JOIN products p ON p.product_id = oi.product_id
WHERE (p.price * oi.quantity ) > (
SELECT avg(p.price * oi.quantity)
FROM order_items oi
JOIN products p ON p.product_id = oi.product_id
) 
```

### Exercice 8 — Top employé par pays

```sql
WITH EmployeeSales AS (
SELECT e.employee_id, e.first_name, e.last_name, e.region_id, SUM(oi.quantity * p.price) AS ca
FROM employees e
JOIN order_items oi ON e.employee_id = oi.employee_id
JOIN products p ON oi.product_id = p.product_id
GROUP BY e.employee_id, e.first_name, e.last_name, e.region_id
), RankedSales AS (
SELECT first_name, last_name, region_id, ca,
RANK() OVER (PARTITION BY region_id ORDER BY ca DESC) as rnk
FROM EmployeeSales
)
SELECT first_name, last_name, region_id, ca
FROM RankedSales
WHERE rnk = 1;
```

### Exercice 9 — Managers actifs

```sql
select  *, count(e.employee_id) over(partition by e.manager_id) as nbr_sub
from employees e
join order_items oi on oi.employee_id = e.employee_id
```

### Exercice 10 — Vue sur une requête courante (VIEW)
```sql
CREATE VIEW v_employee_order_items AS
SELECT oi.order_item_id, oi.order_id, e.first_name, e.last_name,
c.client_name, p.product_name, oi.quantity
FROM order_items oi
JOIN orders o ON oi.order_id = o.order_id
JOIN employees e ON oi.employee_id = e.employee_id
JOIN clients c ON o.client_id = c.client_id
JOIN products p ON oi.product_id = p.product_id;

```

### Exercice 11 - Optimisation

```sql
CREATE INDEX employee_order
ON orders_items (employee_id);
```
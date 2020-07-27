/*Query 1.3-query used for first insight*/
WITH t1 AS
(
  SELECT f.title filme_title,
        c.name category_name,
        f.rental_duration rental_duration,
        NTILE(4)OVER(ORDER BY rental_duration) standard_quartile
FROM film f
JOIN film_category fc
ON f.film_id=fc.film_id
JOIN category c
ON c.category_id=fc.category_id
WHERE c.name IN ('Animation', 'Children', 'Classics', 'Comedy', 'Family', 'Music')
  )
SELECT DISTINCT(category_name),
		  standard_quartile,
      COUNT(*)
FROM t1
GROUP BY 1,2
ORDER BY 1,2;


/*Query 2.1-query used for second insight*/
SELECT DATE_PART('month',r.rental_date) AS Rental_month,
      DATE_PART('year',r.rental_date) AS Rental_year,
      st.store_id Store_ID,
      COUNT(r.rental_id)
FROM rental r
JOIN staff st
ON st.staff_id=r.staff_id
GROUP BY 1,2,3
ORDER BY 4 DESC;



/*Query 2.2-query used for third insight*/
WITH t1 AS
(
  SELECT customer_id cus, SUM(amount) total_amt
  FROM payment
  GROUP BY 1
  ORDER BY total_amt DESC
  LIMIT 10
  )
SELECT DATE_TRUNC('month',p.payment_date) AS month,
      c.first_name||' '||c.last_name full_name,
      COUNT(p.amount) pay_counterpermonth,
      SUM(p.amount) pay_amount
FROM customer c
JOIN payment p
ON c.customer_id=p.customer_id
JOIN t1
ON t1.cus=p.customer_id
GROUP BY 2,1
ORDER BY 2;



/*Query 2.3-query used for fourth insight*/
WITH t1 AS
(
  SELECT customer_id cus,
        SUM(amount) total_amt
  FROM payment
  GROUP BY 1
  ORDER BY total_amt DESC
  LIMIT 10
  ),
 t2 AS
  (
SELECT DATE_TRUNC('month',p.payment_date) AS month,
      c.first_name||' '||c.last_name full_name,
      SUM(p.amount) pay_amount
FROM customer c
JOIN payment p
ON c.customer_id=p.customer_id
JOIN t1
ON t1.cus=p.customer_id
GROUP BY 2,1
ORDER BY 2
)
SELECT t2.month,
      t2.full_name,
      t2.pay_amount,
      LEAD(t2.pay_amount)OVER(PARTITION BY t2.full_name ORDER BY t2.pay_amount) AS lead,
      LEAD(pay_amount)OVER(PARTITION BY t2.full_name ORDER BY t2.pay_amount)-t2.pay_amount AS lead_difference,
      COALESCE(LEAD(pay_amount)OVER(PARTITION BY t2.full_name ORDER BY t2.pay_amount)-t2.pay_amount,0) AS lead_difference_adjust
FROM t2
ORDER BY 6 DESC;

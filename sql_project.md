# Strategic Analytics System: Optimizing Insights for Atliq's Growth

Embark on an innovative project catering to Atliq's diverse needs. Develop an integrated system offering a precise monthly gross sales report for Croma India, empowering the product owner to manage relationships effectively.

The system extends its capabilities to provide strategic insights, including:

- **Top Market Report:** Identifying the top markets in each region based on FY=2021 gross sales.
- **Market Segmentation:** Analysis and identification of the top 2 markets in each region.
- **Forecast Accuracy Report:** Addressing supply chain concerns by generating a detailed forecast accuracy report. This report identifies customers with accuracy drops from 2020 to 2021.

This project ensures informed decision-making, strategic market targeting, and responsive supply chain management, aligning with Atliq's growth objectives.

### Below are the requests :

- ##### As a product owner, I need aggregate monthly gross sales report of Croma India customer so that I can track how much sales this particular customer is generated      for Atliq and manage our relationship accordingly
  
  Report should follow below fields :

  1.  Month
  2. Total gross sales amounts to Croma India this month


##### Query :

```
select 
 s.date, 
round(sum(s.sold_quantity * g.gross_price),2) as gross_price_total
from fact_sales_monthly as s
join  fact_gross_price as g
on g.product_code = s.product_code and 
g.fiscal_year = get_fiscal_year(s.date)
where customer_code = 90002002 
group by s.date
 order by  s.date asc;
```

#### Result : 

```
Date          Gross_price_total

2017-09-01	  122407.56
2017-10-01	  162687.57
2017-12-01	  245673.80
2018-01-01	  127574.74
2018-02-01	  144799.52
2018-04-01	  130643.90
2018-05-01	  139165.10
2018-06-01	  125735.38
2018-08-01	  125409.88
2018-09-01	  343337.17
2018-10-01	  440562.08

```


- ##### Find the top 5 markets from the given dataset

  Report should following fields :
  1.  Market
  2.  net_sales_mln


##### Query :
```
select 
 c.market,
round(sum(net_sales)/1000000,2) as net_sales_mln
from gdb0041.net_sales as ns
join  dim_customer as c on
 c.customer_code = ns.customer_code
where fiscal_year = 2021
group by c.market
order by net_sales_mln desc
limit 5;
```

#### Result :

```
Market                        net_sales_mln
India                         210.67
USA                           132.05
South Korea                   64.01
Canada                        45.89
United Kingdom                44.73

```

- ##### Retrieve the top 2 markets in every region by their gross sales amount in FY=2021.

##### Query :

```
with cte1 as (SELECT  c.market, c.region, round(sum(gross_price_total)/1000000,2) as gross_sales_mln 
FROM gross_sales as gs
join dim_customer as c on
gs.customer_code = c.customer_code
group by c.market, c.region
order by gross_sales_mln desc),
 cte2 as (select * ,
 dense_rank() over(partition by region order by gross_sales_mln desc) as drank from cte1)

 select * from cte2 where drank<=2
```

#### Result :

```
market          	  region	          gross_sales_mln	        drank
India	                APAC	               4813.69	                 1
South Korea	          APAC	               1488.72	                 2
United Kingdom	      EU	               778.81	                 1
France	              EU	               671.18	                 2
Brazil	              LATAM	               33.25	                 1
Mexico	              LATAM	               27.57	                 2
USA                   NA	               2718.81	                 1
Canada	              NA	               908.28	                 2
```


- ##### The supply chain business manager wants to see which customers’ forecast accuracy has dropped from 2020 to 2021. Provide a complete report with these columns: customer_code, customer_name, market, forecast_accuracy_2020, forecast_accuracy_2021

##### Query :

```
 select f_2020.customer_code,
f_2020.customer,
f_2020.market,
f_2020.forecast_accuracy as forecast_acc_2020,
 f_2021.forecast_accuracy as forecast_acc_2021
 from forecast_accuracy_2020 as f_2020
 join forecast_accuracy_2021 as f_2021
 on f_2021.customer_code = f_2020.customer_code
 where f_2021.forecast_accuracy < f_2020.forecast_accuracy
 order by forecast_acc_2020 desc;
```

#### Result :

```
customer_code      customer                      market         forecast_acc_2020   forecast_acc_2021                 
70006158	        Atliq e Store	                Philiphines          	42.6505	      24.4904
70008170	        Atliq e Store	                Australia	            40.9573	      38.7404
90005161	        Zone	                        Pakistan	            40.0813	      37.0962
90014140	        Radio Popular      	          Netherlands	          38.5260	      0.0000
90008166	        Sound                        	Australia	            38.5111	      36.7911
70014143	        Atliq e Store	                Netherlands	          38.3174	      0.0000
90004062	        Flawless Stores	              Japan	                38.2162	      32.5617
90014137	        Media Markt	                  Netherlands	          37.8548	      0.0000
90014138	        Mbit	                        Netherlands	          37.8277	      0.0000
70004069	        Atliq Exclusive	              Japan	                37.6207	      32.0943
90014136	        Reliance Digital	            Netherlands	          37.5855	      0.0000
80006154	        Synthetic	                    Philiphines	          37.4905	      24.6261
70014142	        Atliq Exclusive	              Netherlands	          37.4290	      0.0000
90014141	        Amazon 	                      Netherlands	          37.3913      	0.0000
90005160	        Nomad Stores	                Pakistan	            37.3022	      37.2865
90006156	        Amazon 	                      Philiphines	          37.2124	      27.9378

```

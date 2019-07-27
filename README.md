# aws-athena-cost-billing-queries

Amazon Athena SQL queries to summarize and explore your detailed cost reports. 

Example screenshots show the Athena results downloaded as CSV within Microsoft Excel, otherwise unmodified. 

## Total cost by service and line-item, last 30 days

```sql
SELECT product_region                       AS region,
       product_product_name                 AS service,
       line_item_usage_type                 AS usage_type,
       line_item_line_item_description      AS description,
       round(sum(line_item_blended_cost),0) AS blended_cost,
       count(*)                             AS records
FROM     hourly_cost_for_athena
WHERE    line_item_usage_start_date >= (CURRENT_DATE - interval '30' day)
GROUP BY product_region, 
         product_product_name, 
         line_item_usage_type, 
         line_item_line_item_description
HAVING   round(sum(line_item_blended_cost),0) <> 0
ORDER BY  blended_cost DESC; 
```

![alt text](images/cost-by-service-and-line-item-last-30-days.png)

## Total cost by resource ID, last 30 days

Want to know how much a specific resource ID (e.g. EC2 instance, EBS volume, load balancer, etc.) is costing you? You can use this report *if* you have enabled resource IDs in your cost report. Note that enabling resource IDs may significantly increase the storage size of your cost reports in S3:

```sql
SELECT product_region                       AS region,
       line_item_resource_id                AS resource_id,
       round(sum(line_item_blended_cost),0) AS blended_cost,
       count(*)                             AS records
FROM     hourly_cost_for_athena
WHERE    line_item_usage_start_date >= (CURRENT_DATE - interval '30' day)
and product_product_name = 'Amazon Elastic Compute Cloud'
GROUP BY product_region, 
         line_item_resource_id
HAVING   round(sum(line_item_blended_cost),0) <> 0
ORDER BY  blended_cost DESC; 
```

![alt text](images/cost-by-resource-id-last-30-days.png)


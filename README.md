# BNB-MUMBAI


``````sql
-- Load customer reviews data into BigQuery from a CSV file
LOAD DATA OVERWRITE gemini_demo.customer_reviews 
(customer_review_id INT64, customer_id INT64, location_id INT64, review_datetime DATETIME, review_text STRING, social_media_source STRING, social_media_handle STRING)
FROM FILES (
  format = 'CSV',
  uris = ['gs://your-bucket-name/customer_reviews.csv']
);


---

# Sentiment Analysis of Movies by Social Media Comments

## Table of Contents
1. [Introduction](#introduction)
2. [Requirements](#requirements)
3. [Step-by-Step Guide](#step-by-step-guide)
    1. [Enable APIs on Google Cloud](#enable-apis-on-google-cloud)
    2. [Assign IAM Roles for Vertex AI User to Gemini-Connection](#assign-iam-roles-for-vertex-ai-user-to-gemini-connection)
    3. [Load Data into BigQuery](#load-data-into-bigquery)
    4. [Create Model for Sentiment Analysis](#create-model-for-sentiment-analysis)
    5. [Generate Sentiment Boolean](#generate-sentiment-boolean)
    6. [Order by DateTime](#order-by-datetime)
    7. [Count for Apps](#count-for-apps)
4. [Usage Example](#usage-example)
5. [License](#license)

---

## Introduction

In this project, we perform sentiment analysis on social media comments related to movies using **Google Cloud** services such as **Vertex AI**, **BigQuery**, and **Gemini**. The goal is to understand the general sentiment (positive, negative, or neutral) of social media comments about movies, visualize trends, and analyze which movies are receiving favorable or unfavorable attention.

---

## Requirements

Before you begin, make sure you have the following:
- A **Google Cloud Platform (GCP)** account.
- Google Cloud SDK installed on your local machine.
- Access to **Vertex AI**, **BigQuery**, and **Gemini APIs**.
- A dataset containing **movie-related comments** (social media comments in CSV/JSON format).

---

## Step-by-Step Guide Flow
```Start
  ↓
Data Source (Social Media Comments)  
  ↓
Load Data into BigQuery
  ↓
Enable APIs (Vertex AI, BigQuery, Gemini)
  ↓
Assign IAM Roles
  ↓
Create Sentiment Analysis Model
  ↓
Generate Sentiment Boolean (Positive/Negative)
  ↓
Order by DateTime
  ↓
Count Sentiment Results (By Movie/Social Media Source)
  ↓
Visualize Sentiment Trends (Looker Studio)
  ↓
End
```


### 1. **Enable APIs on Google Cloud**

You need to enable the following APIs in your Google Cloud project:
- **Vertex AI API**: For creating and managing machine learning models.
- **BigQuery API**: For storing and querying your data.
- **Gemini API**: For performing sentiment analysis and text classification.

#### Steps:
1. Go to the [Google Cloud Console](https://console.cloud.google.com/).
2. Search for the required APIs: Vertex AI, BigQuery, and Gemini.
3. Click **Enable** to activate them for your project.

---

### 2. **Assign IAM Roles for Vertex AI User to Gemini-Connection**

Make sure that users or service accounts have the necessary permissions to access **Vertex AI** and **Gemini** services.

#### Steps:
1. Go to the **IAM & Admin** section in the Google Cloud Console.
2. Assign the **Vertex AI User** role to users interacting with **Vertex AI**.
3. Assign the **BigQuery User** role to users working with **BigQuery**.
4. Assign the **Gemini-Connection User** role for using the **Gemini API**.

---

### 3. **Load Data into BigQuery**

Upload your social media comments dataset into **BigQuery**. Ensure the dataset contains relevant information such as the movie name, comment, and timestamp.

#### Steps:
1. In the **BigQuery Console**, create a new dataset.
2. Upload your dataset (in CSV/JSON format) into a table.

#### SQL to load the data:
```sql
-- Load customer reviews data into BigQuery from a CSV file
LOAD DATA OVERWRITE gemini_demo.customer_reviews 
(customer_review_id INT64, customer_id INT64, location_id INT64, review_datetime DATETIME, review_text STRING, social_media_source STRING, social_media_handle STRING)
FROM FILES (
  format = 'CSV',
  uris = ['gs://your-bucket-name/customer_reviews.csv']
);
```

---

### 4. **Create Model for Sentiment Analysis**

Train a sentiment analysis model using **Vertex AI** and **Gemini**. This model will classify social media comments into **positive**, **negative**, or **neutral** sentiments.

#### Steps:
1. Go to **Vertex AI** in the Google Cloud Console.
2. Select **AutoML** and upload your labeled dataset.
3. Choose **Text Classification** as the model type.
4. Train the model and deploy it.

#### SQL to create the sentiment analysis model:
```sql
-- Create or replace a Gemini model in Vertex AI for sentiment analysis
CREATE OR REPLACE MODEL `gemini_demo.gemini_pro` 
REMOTE WITH CONNECTION `us.gemini_conn`
OPTIONS (endpoint = 'gemini-pro');
```

---

### 5. **Generate Sentiment Boolean**

Once the model is deployed, use it to analyze the sentiment of each comment. The model will output a sentiment boolean for each comment, indicating whether the sentiment is positive, negative, or neutral.

#### SQL to generate sentiment analysis:
```sql
-- Use the Gemini model to classify sentiment as positive or negative
CREATE OR REPLACE TABLE 
`gemini_demo.customer_reviews_analysis` AS (
  SELECT ml_generate_text_llm_result, social_media_source, review_text, customer_id, location_id, review_datetime
  FROM
  ML.GENERATE_TEXT(
    MODEL `gemini_demo.gemini_pro`,
    (
      SELECT social_media_source, customer_id, location_id, review_text, review_datetime, CONCAT(
        'Classify the sentiment of the following text as positive or negative.',
        review_text, " In your response don't include the sentiment explanation. Remove all extraneous information from your response, it should be a boolean response either positive or negative."
      ) AS prompt
      FROM `gemini_demo.customer_reviews`
    ),
    STRUCT(
      0.2 AS temperature, TRUE AS flatten_json_output
    )
  )
);
```

---

### 6. **Order by DateTime**

You can order the data based on the timestamp of the comments to analyze sentiment over time.

#### SQL to order by datetime:
```sql
-- Retrieve the sentiment analysis results ordered by the review datetime
SELECT * 
FROM `gemini_demo.customer_reviews_analysis`
ORDER BY review_datetime;
```

---

### 7. **Count for Apps**

You can aggregate the sentiment analysis results to count the number of positive, negative, and neutral comments for each movie.

#### SQL to count sentiment by social media source:
```sql
-- Count the number of positive and negative sentiments by social media source
SELECT sentiment, social_media_source, COUNT(*) AS count 
FROM `gemini_demo.cleaned_data_view`
WHERE sentiment IN ('positive') OR sentiment IN ('negative')
GROUP BY sentiment, social_media_source
ORDER BY sentiment, count;
```

---

## Usage Example

Here’s how you can use this setup to analyze social media comments:

1. **Load Comments**: Import your social media data into BigQuery.
2. **Run Sentiment Analysis**: Use **Vertex AI** and **Gemini** to analyze the sentiment of the comments.
3. **Query Data**: Use **BigQuery** queries to order comments by datetime and count the sentiment for each movie.
4. **Visualize Results**: You can use **Looker Studio** to visualize sentiment trends and see how different movies are being discussed on social media.

---

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

This **README** serves as a guide for setting up sentiment analysis on movie-related social media comments using **Google Cloud** services. Follow the instructions carefully to load data, create models, and analyze sentiment using **BigQuery** and **Vertex AI**.

--- 

This structure integrates the provided SQL code into the appropriate steps, making it part of a full guide to set up sentiment analysis using **Google Cloud** tools.

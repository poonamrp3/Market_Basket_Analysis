## Overview

This project performs market basket analysis on transactional data to discover frequent itemsets and association rules across different countries. The analysis helps identify purchasing patterns and customer behavior insights through advanced data mining techniques.

## Features

- **Data Cleaning & Preprocessing**: Handles inconsistent data and prepares transaction datasets
- **Frequent Itemset Mining**: Implements both FP-Growth and Apriori algorithms
- **Country-wise Analysis**: Analyzes purchasing patterns across different regions
- **Association Rules**: Generates rules with support, confidence, and lift metrics
- **Graph Database Storage**: Stores results in Neo4j for advanced querying and visualization
- **Visual Analytics**: Creates network graphs to visualize item relationships

## Prerequisites

The complete Jupyter notebook (`main.ipynb`) includes all necessary code and dependencies. All required libraries and their installation commands are provided within the notebook itself.

### Data Requirements

The transaction dataset is hosted on Google Drive due to file size [https://drive.google.com/file/d/1HoePAcsBkNkrbXkugh3BmXTUgpkbm4Sg/view?usp=sharing]. Download the dataset from the link provided in the notebook and place it in the `data/` directory, or update the file path in the notebook accordingly.

## Project Structure

```
market-basket-analysis/
│
├── main.ipynb              # Complete Jupyter notebook with analysis
├── data/                   # Data directory (download from Drive link)
│   └── Assignment-1_Data.csv # Transaction dataset (from Google Drive)
├── README.md              # This file
└── association_rules_*.png # Generated graph visualizations
```

## Installation & Setup

1. **Clone the repository**

2. **Download the dataset**
   - The Google Drive link for the dataset is provided in the notebook
   - Download and place the CSV file in the `data/` directory
   - Alternatively, update the file path in the notebook to your download location

3. **Install dependencies**
   The notebook includes all necessary installation commands. Simply run the first few cells of `main.ipynb` to install required packages.

3. **Set up Neo4j Database (Required for graph visualization)**
   - Create a free Neo4j Aura instance following the setup guide below
   - Update the connection details in the notebook

## Neo4j Aura Setup Guide

### Step 1: Create Neo4j Aura Instance

1. **Go to Neo4j Aura**: Visit [https://neo4j.com/cloud/aura/](https://neo4j.com/cloud/aura/)

2. **Sign Up/Login**: Create a free account or login to your existing account

3. **Create New Instance**:
   - Click "Create New Instance"
   - Choose "AuraDB Free" (suitable for this project)
   - Select your preferred region
   - Give your instance a meaningful name (e.g., "market-basket-analysis")

4. **Instance Configuration**:
   - **Database Name**: Keep default or choose your own
   - **Initial Password**: Choose a strong password and **save it securely**
   - Click "Create Instance"

5. **Wait for Deployment**: Your instance will take a few minutes to deploy

### Step 2: Get Connection Details

Once your instance is ready:

1. **Connection URI**: Copy the connection URI (looks like `neo4j+s://xxxxxxxx.databases.neo4j.io`)
2. **Username**: Default is `neo4j`
3. **Password**: The password you set during creation

### Step 3: Configure the Notebook

In the `main.ipynb` notebook, update the Neo4j connection section:

```python
# Neo4j connection details - UPDATE THESE WITH YOUR INSTANCE DETAILS
uri = "neo4j+s://YOUR_INSTANCE_ID.databases.neo4j.io"  # Your Aura instance URI
username = "neo4j"  # Default username
password = "YOUR_PASSWORD"  # Your instance password
```

### Step 4: Test Connection

The notebook includes connection testing code. Run the Neo4j sections to verify your connection works.

### Important Security Notes:
- **Never commit credentials**: Use environment variables in production
- **Keep passwords secure**: Store your Neo4j password safely
- **Monitor usage**: Free tier has limitations on storage and compute

### Accessing Your Graph Database

Once the notebook completes execution:

1. **Open Neo4j Browser**: Go to your Aura instance dashboard and click "Open with Neo4j Browser"
2. **Login**: Use your instance credentials
3. **Explore Data**: Use the Cypher queries provided in the Neo4j Graph Database section below

## Usage

### Running the Analysis

1. **Open Jupyter Notebook**:
   ```bash
   jupyter notebook main.ipynb
   ```

2. **Run all cells** or execute step by step to see the analysis progress

3. **Monitor outputs** for frequent itemsets and association rules

### Key Parameters

- **Minimum Support**: 0.1 (10% of transactions must contain the itemset)
- **Minimum Confidence**: 0.8 (80% confidence threshold for association rules)
- **Country Filter**: Only countries with >1000 transactions are analyzed

### Data Flow

1. **Data Import**: Load CSV data with error handling for malformed rows
2. **Data Cleaning**: 
   - Keep only relevant columns (`BillNo`, `Itemname`, `Country`)
   - Remove missing values and whitespace
   - Filter countries by transaction volume
3. **Transaction Grouping**: Group items by bill number to create market baskets
4. **Encoding**: Convert to binary matrix format for algorithm processing
5. **Mining**: Apply FP-Growth and Apriori algorithms
6. **Rule Generation**: Create association rules with metrics
7. **Storage**: Save results to Neo4j graph database
8. **Visualization**: Generate network graphs

## Neo4j Graph Database

### Your Custom Setup
After creating your Neo4j Aura instance (see setup guide above), the project will connect to your personal database for storing and querying association rules.

### Graph Structure
- **Country Nodes**: Represent different countries
- **Antecedent Nodes**: Items that appear in rule premises
- **Consequent Nodes**: Items that appear in rule conclusions
- **Relationships**: 
  - `HAS_RULE`: Links countries to antecedents
  - `RESULTS_INTO`: Links antecedents to consequents with metrics

### Sample Queries

**View all rules for a specific country:**
```cypher
MATCH (ct:Country {name: 'United Kingdom'})-[r1:HAS_RULE]->(a:Antecedent)-[r2:RESULTS_INTO]->(c:Consequent)
RETURN ct, a, r1, r2, c
```

**Find high-confidence rules:**
```cypher
MATCH (a:Antecedent)-[r:RESULTS_INTO]->(c:Consequent)
WHERE r.confidence > 0.9
RETURN a.items, c.items, r.confidence, r.support, r.lift
ORDER BY r.confidence DESC
```

## Output Interpretation

### Association Rules Metrics

- **Support**: Percentage of transactions containing both antecedent and consequent
- **Confidence**: Probability of consequent given antecedent
- **Lift**: Ratio of observed to expected frequency if items were independent
  - Lift > 1: Positive correlation (items bought together more than expected)
  - Lift = 1: Independence
  - Lift < 1: Negative correlation

### Example Rule
```
Antecedent: {White Tea, Green Tea} → Consequent: {Herbal Tea}
Support: 0.15 (15% of transactions)
Confidence: 0.85 (85% probability)
Lift: 2.1 (210% more likely than random)
```

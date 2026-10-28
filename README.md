# E-Commerce Intelligence Dashboard

A Flask dashboard for customer purchase propensity, personalized recommendations,
product demand forecasts, slow-moving product analysis, and homepage ranking.
The included Retailrocket event data is converted into an SQLite database on the
first run; the large item-property CSVs are not loaded because their fields do
not provide inventory, price, or rating information.

## Requirements and run

```powershell
python -m pip install -r requirements.txt
python train_models.py
python app.py
```

Open <http://127.0.0.1:5000/>. The first training run processes `data/events.csv`
in chunks and may take several minutes. It creates `data/ecommerce.db` and
`model/purchase_model.pkl`. Later runs reuse these files unless the source CSV
changes. Remove the generated SQLite database if you want to force a rebuild.

## Models and data mapping

- **Logistic Regression:** estimates historical purchase propensity from a
  visitor's recorded product views and cart additions. The target is whether
  the visitor has any recorded transaction. The dashboard reports accuracy,
  precision, recall, and F1 on a stratified holdout. This is historical event
  propensity, not a guaranteed prediction of future behavior.
- **KNN:** recommends unseen products based on similarity in observed views,
  cart additions, and transactions for a visitor's previously interacted
  products.
- **Linear Regression:** forecasts the next month's transaction count for a
  selected product from its monthly transaction history. MSE, RMSE, and R² are
  shown when the available holdout history supports those metrics; very short
  product histories are identified as limited.
- **Slow movers:** products with fewer than 10 recorded transactions.
- **Homepage ranking:** min-max-normalized score using 50% transactions, 30%
  cart additions, and 20% views.
- **Restock and discounts:** not inferred. The supplied events do not contain
  stock, price, rating, age, gender, income, or a product category catalog, so
  the dashboard explains why stock thresholds and demographic purchase inputs
  cannot be evaluated honestly from these files.

## API

All request bodies are JSON.

```text
POST /api/predict/purchase
{"views": 8, "cart_adds": 2}

POST /api/recommend
{"visitor_id": 257597}

GET /api/forecast/<item_id>
```

The recommendation endpoint returns up to five previously unseen similar
products. A visitor ID must have recorded product activity. The forecast API
returns monthly history, next-month forecast, and evaluation metrics.

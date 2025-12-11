---
hidden: true
---

# Hyperparameter Tune XGBoost with 1,000 CPUs

In this example we:

* Download [this 1.4GB Kaggle Dataset](https://www.kaggle.com/datasets/robikscube/flight-delay-dataset-20182022/data?select=Combined_Flights_2022.csv) of commercial flight delays.
* Train 36 XGBoost models with different parameters using 13, 80-CPU machines.
* Identify the best model from training results.

### Step 1: Upload your data to the cluster

Download the following CSV file:

{% embed url="https://www.kaggle.com/datasets/robikscube/flight-delay-dataset-20182022?resource=download&select=Combined_Flights_2022.csv" %}

Then upload it to your Burla cluster filesystem:

<figure><img src="../.gitbook/assets/upload_demo.gif" alt=""><figcaption><p>(upload section is fast-forwarded)</p></figcaption></figure>

Any files uploaded here will appear in a network attached folder at `./shared` inside every container in the cluster. Conversely, any files your code leaves in this folder will appear in the "Filesystem" tab where you can download them later, or store for future work!

### Step 2: Boot some VMs

In the "Settings" tab, select the hardware and quantity of machines you want, then hit **⏻ Start** !\
Here we boot 13, 80-CPU VM's, these VM's delete themself after 15min. of inactivity.

<figure><img src="../.gitbook/assets/cluster_boot_demo.gif" alt=""><figcaption><p>(Booting is fast-forwarded, this cluster actually took 1.5 min. to boot up)</p></figcaption></figure>

Now that our machines are ready, we can call `remote_parallel_map` !

You may have noticed in the settings we're using the `python:3.12` docker image.\
This is the image the code will run inside, and it doesn't come with any of the packages we need (like XGBoost, Pandas, etc). This is ok because Burla detect's local packages at runtime and quickly installs them in all containers, usually in just a few seconds.

### Step 3: Write a function to train one model.

This function:

* Loads `Combined_Flights_2022.csv` from the `./shared` folder as a Pandas DataFrame.
* Cleans and separates data into train / test sets.
* Trains one XGBoost model using the provided `params` dict, and 80 CPUs.
* Scores the model on the test set, then returns the AUC.

```python
import pandas as pd
import xgboost as xgb
from sklearn.model_selection import train_test_split
from sklearn.metrics import roc_auc_score

# avoid leakage and messy data
columns_to_train_on = [
    "Airline","Operating_Airline","Marketing_Airline_Network",
    "IATA_Code_Marketing_Airline","IATA_Code_Operating_Airline",
    "Origin","Dest","OriginState","DestState","OriginWac","DestWac",
    "OriginAirportID","DestAirportID","OriginCityMarketID","DestCityMarketID",
    "Distance","DistanceGroup","CRSElapsedTime",
    "DayOfWeek","Month","Quarter","DayofMonth","Year",
    "DepTimeBlk","ArrTimeBlk","CRSDepTime","CRSArrTime"
]

def train_model(params):
    print("Loading dataset ...")
    df = pd.read_csv("./shared/Combined_Flights_2022.csv")
    df = df.dropna(subset=["ArrDel15"])  # <- null when flight never arrived (cancelled/diverted/etc)
    
    y = df["ArrDel15"].astype(int)
    X = pd.get_dummies(df[columns_to_train_on], dummy_na=False)
    X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2)
    data_size_gb = (X_train.memory_usage(deep=True).sum() + y_train.memory_usage(deep=True)) / (1024**3)

    # `data_size_gb` is > than original csv size mostly from one hot encoding
    print(f"Training XGBClassifier on {format(len(df), ",")} rows ({data_size_gb:.1f}GB) using 80 CPUs ...")
    model = xgb.XGBClassifier(tree_method="hist", n_jobs=80, eval_metric="auc", **params)
    model.fit(X_train, y_train)
    auc = roc_auc_score(y_test, model.predict_proba(X_test)[:, 1])

    print(f"Done training, [AUC={auc:.4f}] Hyperparameters: {params}")
    return {"auc": auc, "params": params}

```

To test out the function we call it on just one machine, by passing it one set of parameters:

```python
from burla import remote_parallel_map

parameter_grid = [{"n_estimators": 300, "max_depth": 4, "eta": 0.1}]

results = remote_parallel_map(train_model, parameter_grid, func_cpu=80)

print(f"Model with params {parameter_grid[0]} yielded an AUC of: {results[0]}!")
```

We also pass `func_cpu=80` to tell Burla this function call should have 80 CPU's made available to it.\
We'll need this since we're passing `n_jobs=80` to XGBoost inside the `train_model` function.

### Step 4: Call the function in parallel, on 13 separate VMs!

Here we pass 36 sets of parameters to `train_model`.\
Because each function call requires 80 CPUs, and we have 13, 80CPU machines, this will immediately start 13 function calls, and queue the remaining 26. Burla can reliably queue up to 10 million of inputs.

```python
parameter_grid = [
    {"n_estimators": n, "max_depth": d, "eta": e}
    for n in [300, 600, 900] for d in [4, 6, 8, 10] for e in [0.05, 0.1, 0.15]
]

results = remote_parallel_map(train_model, parameter_grid, func_cpu=80)

best = max(results, key=lambda r: r["auc"])
print("Best:", best)
```

Once submitted, we can monitor progress and view logs from the "Jobs" tab in the dashboard:

<figure><img src="../.gitbook/assets/demo.gif" alt=""><figcaption></figcaption></figure>




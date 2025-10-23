[ML_Project.ipynb](https://github.com/user-attachments/files/23098758/ML_Project.ipynb)
{
  "nbformat": 4,
  "nbformat_minor": 0,
  "metadata": {
    "colab": {
      "provenance": []
    },
    "kernelspec": {
      "name": "python3",
      "display_name": "Python 3"
    },
    "language_info": {
      "name": "python"
    }
  },
  "cells": [
    {
      "cell_type": "markdown",
      "source": [
        "## 💰 Money Laundering Detection System (ML Project)\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "\n",
        "This project uses **Machine Learning** to detect suspicious or fraudulent financial transactions.  \n",
        "We preprocess AML data, train an **XGBoost model**, evaluate its performance, and deploy it using **Streamlit + Ngrok** for real-time predictions.\n",
        "\n",
        "**Steps Covered:**\n",
        "1. Data loading and preprocessing  \n",
        "2. Feature selection and model training  \n",
        "3. Evaluation and feature importance  \n",
        "4. Model saving and web app deployment\n"
      ],
      "metadata": {
        "id": "PlApF7wzbp2-"
      }
    },
    {
      "cell_type": "markdown",
      "source": [
        "## Install Required Libraries\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "*italicized text*We install all necessary Python libraries such as Streamlit, Pyngrok, Scikit-learn, and XGBoost for model training and deployment.\n"
      ],
      "metadata": {
        "id": "63kOHi64WOhS"
      }
    },
    {
      "cell_type": "code",
      "execution_count": null,
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "Z8w1fUerTKUd",
        "outputId": "b87eb9de-7281-4b27-a12d-72cc700ac2d7"
      },
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "\u001b[2K   \u001b[90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\u001b[0m \u001b[32m10.1/10.1 MB\u001b[0m \u001b[31m8.7 MB/s\u001b[0m eta \u001b[36m0:00:00\u001b[0m\n",
            "\u001b[2K   \u001b[90m━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━\u001b[0m \u001b[32m6.9/6.9 MB\u001b[0m \u001b[31m21.4 MB/s\u001b[0m eta \u001b[36m0:00:00\u001b[0m\n",
            "\u001b[?25h"
          ]
        }
      ],
      "source": [
        "!pip install streamlit pyngrok scikit-learn xgboost --quiet"
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 📚 Import Dependencies\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Import essential libraries for data manipulation, preprocessing, and model training (Pandas, NumPy, Sklearn, XGBoost, etc.).\n"
      ],
      "metadata": {
        "id": "7FlMjwMTVWGr"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "import pandas as pd\n",
        "import numpy as np\n",
        "from sklearn.preprocessing import LabelEncoder\n",
        "from sklearn.model_selection import train_test_split\n",
        "from sklearn.metrics import classification_report, confusion_matrix, roc_auc_score\n",
        "import matplotlib.pyplot as plt\n",
        "import seaborn as sns\n",
        "import xgboost as xgb\n",
        "from pyngrok import ngrok\n",
        "import streamlit as st\n",
        "import random\n",
        "from datetime import datetime, timedelta"
      ],
      "metadata": {
        "id": "VZCpxHocTWGR"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "## ⚙️ Generate or Preprocess Transaction Data\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "This section defines logic to simulate or preprocess anti-money-laundering (AML) transaction data for model input.\n"
      ],
      "metadata": {
        "id": "ptOjsgW_WLNf"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "import pandas as pd\n",
        "import numpy as np\n",
        "import random\n",
        "from datetime import datetime, timedelta\n",
        "from collections import defaultdict\n",
        "import math\n",
        "\n",
        "# -----------------------------\n",
        "# CONFIGURATION\n",
        "# -----------------------------\n",
        "RANDOM_SEED = 42\n",
        "random.seed(RANDOM_SEED)\n",
        "np.random.seed(RANDOM_SEED)\n",
        "\n",
        "num_records = 100_000                # Total transactions to generate\n",
        "fraud_ratio = 0.05                   # Base random fraud probability\n",
        "max_gap_minutes = 120                # threshold for \"rapid transaction\" (in minutes)\n",
        "num_accounts = 4000                  # size of account pool (increase to diversify)\n",
        "start_date = datetime(2023, 1, 1)\n",
        "end_date = datetime(2025, 10, 1)\n",
        "\n",
        "accounts = [f\"A{idx}\" for idx in range(1000, 1000 + num_accounts)]\n",
        "# receiver pool slightly larger to allow many-to-many mappings\n",
        "receiver_pool = [f\"A{idx}\" for idx in range(2000, 2000 + (num_accounts * 2))]\n",
        "\n",
        "countries = ['India', 'USA', 'UK', 'Singapore', 'UAE', 'Germany', 'Nigeria', 'China']\n",
        "channels = ['Online', 'ATM', 'Bank Transfer', 'Mobile App']\n",
        "account_types = ['Savings', 'Current', 'Business', 'Offshore']\n",
        "\n",
        "# -----------------------------\n",
        "# HELPERS\n",
        "# -----------------------------\n",
        "def random_between_dates(start, end):\n",
        "    \"\"\"Return random datetime between start and end (uniform).\"\"\"\n",
        "    delta = end - start\n",
        "    rand_seconds = random.randint(0, int(delta.total_seconds()))\n",
        "    return start + timedelta(seconds=rand_seconds)\n",
        "\n",
        "def generate_countries():\n",
        "    return random.choice(countries), random.choice(countries)\n",
        "\n",
        "def generate_channel():\n",
        "    return random.choice(channels)\n",
        "\n",
        "def generate_account_type():\n",
        "    return random.choice(account_types)\n",
        "\n",
        "def minutes_to_timedelta(minutes):\n",
        "    return timedelta(seconds=int(minutes * 60))\n",
        "\n",
        "# -----------------------------\n",
        "# TRANSACTION COUNT DISTRIBUTION ACROSS ACCOUNTS\n",
        "# -----------------------------\n",
        "# We'll allocate transactions per account using a Poisson-like distribution so some accounts have many txns,\n",
        "# most have few, which is realistic.\n",
        "mean_tx_per_account = num_records / num_accounts  # ~25\n",
        "# sample counts, then rescale to sum to num_records\n",
        "raw_counts = np.random.poisson(lam=mean_tx_per_account, size=num_accounts).astype(int)\n",
        "# Ensure at least 1 transaction per account to avoid zeros (optional)\n",
        "raw_counts[raw_counts == 0] = 1\n",
        "\n",
        "# Scale counts to exactly num_records\n",
        "total_raw = raw_counts.sum()\n",
        "scaling_factor = num_records / total_raw\n",
        "scaled = np.floor(raw_counts * scaling_factor).astype(int)\n",
        "\n",
        "# correct rounding errors to match exactly num_records\n",
        "diff = num_records - scaled.sum()\n",
        "idx = 0\n",
        "while diff > 0:\n",
        "    scaled[idx % num_accounts] += 1\n",
        "    idx += 1\n",
        "    diff -= 1\n",
        "while diff < 0:\n",
        "    # remove 1 from accounts with >1\n",
        "    for j in range(num_accounts):\n",
        "        if scaled[j] > 1 and diff < 0:\n",
        "            scaled[j] -= 1\n",
        "            diff += 1\n",
        "        if diff == 0:\n",
        "            break\n",
        "\n",
        "txn_counts_per_account = dict(zip(accounts, scaled))\n",
        "\n",
        "# -----------------------------\n",
        "# GENERATE TRANSACTIONS PER ACCOUNT (chronological)\n",
        "# -----------------------------\n",
        "rows = []\n",
        "transaction_counter = 0\n",
        "\n",
        "for account_id, count in txn_counts_per_account.items():\n",
        "    # pick a random start time for this account somewhere between start_date and end_date - 1 day\n",
        "    account_start = random_between_dates(start_date, end_date - timedelta(days=1))\n",
        "    last_time = account_start\n",
        "    last_receiver_country = None\n",
        "    # Simulate inter-arrival times (minutes) using a mixture: many short gaps, some long gaps\n",
        "    # We'll draw from exponential (for short bursts) plus occasional long gaps\n",
        "    for n in range(count):\n",
        "        transaction_counter += 1\n",
        "        transaction_id = f\"T{transaction_counter:07d}\"  # zero-padded\n",
        "\n",
        "        # Interarrival minutes\n",
        "        if n == 0:\n",
        "            # first transaction time is last_time (already set)\n",
        "            txn_time = last_time\n",
        "        else:\n",
        "            # with 80% chance use short exponential gap (burst), else a longer gap\n",
        "            if random.random() < 0.8:\n",
        "                gap_minutes = np.random.exponential(scale=60)  # avg 60 min\n",
        "            else:\n",
        "                gap_minutes = np.random.exponential(scale=60*24)  # long gap avg 1 day\n",
        "            # clamp gap so we don't jump too far beyond end_date, and make it at least 1 minute\n",
        "            gap_minutes = max(1, min(gap_minutes, (end_date - last_time).total_seconds() / 60 - 1))\n",
        "            txn_time = last_time + minutes_to_timedelta(gap_minutes)\n",
        "\n",
        "        # If txn_time goes beyond end_date, wrap back into allowed range (rare)\n",
        "        if txn_time > end_date:\n",
        "            txn_time = random_between_dates(start_date, end_date)\n",
        "\n",
        "        # Receiver and party details\n",
        "        receiver_id = random.choice(receiver_pool)\n",
        "        amount = round(random.uniform(100, 1_000_000), 2)\n",
        "        sender_country, receiver_country = generate_countries()\n",
        "        channel = generate_channel()\n",
        "        account_type = generate_account_type()\n",
        "        is_weekend = 1 if txn_time.weekday() >= 5 else 0\n",
        "        high_value = 1 if amount > 500_000 else 0\n",
        "        international = 1 if sender_country != receiver_country else 0\n",
        "\n",
        "        # previous transaction info for this account\n",
        "        if last_time:\n",
        "            previous_transaction_time = last_time\n",
        "            time_diff_minutes = (txn_time - last_time).total_seconds() / 60.0\n",
        "            previous_receiver_country = last_receiver_country\n",
        "            country_changed = 1 if (previous_receiver_country is not None and previous_receiver_country != receiver_country) else 0\n",
        "            same_account_multiple_txn = 1 if time_diff_minutes <= max_gap_minutes else 0\n",
        "        else:\n",
        "            previous_transaction_time = None\n",
        "            time_diff_minutes = None\n",
        "            previous_receiver_country = None\n",
        "            country_changed = 0\n",
        "            same_account_multiple_txn = 0\n",
        "\n",
        "        # FRAUD LOGIC: combine rules + randomness\n",
        "        fraud = 0\n",
        "        # Rule 1: High-value international from offshore\n",
        "        if high_value and international and account_type == 'Offshore':\n",
        "            fraud = 1\n",
        "        # Rule 2: Rapid multi-country transfers\n",
        "        elif same_account_multiple_txn and country_changed and time_diff_minutes is not None and time_diff_minutes <= max_gap_minutes:\n",
        "            # add randomness to not flag ALL such cases; real systems use scores\n",
        "            if random.random() < 0.95:\n",
        "                fraud = 1\n",
        "        # Rule 3: Weekend unusual high value\n",
        "        elif is_weekend and high_value and random.random() < 0.25:\n",
        "            fraud = 1\n",
        "        # Rule 4: repeated similar amounts to many destinations in short time (smurfing)\n",
        "        elif same_account_multiple_txn and high_value and random.random() < 0.4:\n",
        "            fraud = 1\n",
        "        # Base random fraud\n",
        "        elif random.random() < fraud_ratio:\n",
        "            fraud = 1\n",
        "\n",
        "        rows.append({\n",
        "            'transaction_id': transaction_id,\n",
        "            'account_id': account_id,\n",
        "            'receiver_id': receiver_id,\n",
        "            'amount': amount,\n",
        "            'sender_country': sender_country,\n",
        "            'receiver_country': receiver_country,\n",
        "            'channel': channel,\n",
        "            'account_type': account_type,\n",
        "            'transaction_time': txn_time.strftime(\"%Y-%m-%d %H:%M:%S\"),\n",
        "            'is_weekend': is_weekend,\n",
        "            'high_value': high_value,\n",
        "            'international': international,\n",
        "            'previous_transaction_time': previous_transaction_time.strftime(\"%Y-%m-%d %H:%M:%S\") if previous_transaction_time else None,\n",
        "            'time_diff_minutes': round(time_diff_minutes, 2) if time_diff_minutes is not None else None,\n",
        "            'previous_receiver_country': previous_receiver_country if previous_receiver_country else None,\n",
        "            'country_changed': country_changed,\n",
        "            'same_account_multiple_txn': same_account_multiple_txn,\n",
        "            'is_fraud': fraud\n",
        "        })\n",
        "\n",
        "        # update last_time / last_receiver_country for this account\n",
        "        last_time = txn_time\n",
        "        last_receiver_country = receiver_country\n",
        "\n",
        "# -----------------------------\n",
        "# CREATE DATAFRAME & SAVE CSV\n",
        "# -----------------------------\n",
        "df = pd.DataFrame(rows)\n",
        "\n",
        "# Shuffle dataset globally so records are not sorted by account\n",
        "df = df.sample(frac=1, random_state=RANDOM_SEED).reset_index(drop=True)\n",
        "\n",
        "# Save to CSV\n",
        "output_filename = 'aml_transactions_full_100k.csv'\n",
        "df.to_csv(output_filename, index=False)\n",
        "\n",
        "# -----------------------------\n",
        "# QUICK SUMMARY STATISTICS\n",
        "# -----------------------------\n",
        "fraud_counts = df['is_fraud'].value_counts()\n",
        "fraud_pct = df['is_fraud'].value_counts(normalize=True) * 100\n",
        "\n",
        "print(\"✅ Dataset generated and saved to:\", output_filename)\n",
        "print(\"\\n-- Basic counts --\")\n",
        "print(fraud_counts)\n",
        "print(\"\\n-- Percentages --\")\n",
        "print(fraud_pct.round(3))\n",
        "\n",
        "# Example head\n",
        "print(\"\\n-- Sample rows --\")\n",
        "print(df.head(8).to_string(index=False))"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "4xb0UmjaTd4h",
        "outputId": "04c6edef-3895-4fa5-9db7-a063364bfc75"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "✅ Dataset generated and saved to: aml_transactions_full_100k.csv\n",
            "\n",
            "-- Basic counts --\n",
            "is_fraud\n",
            "1    66190\n",
            "0    33810\n",
            "Name: count, dtype: int64\n",
            "\n",
            "-- Percentages --\n",
            "is_fraud\n",
            "1    66.19\n",
            "0    33.81\n",
            "Name: proportion, dtype: float64\n",
            "\n",
            "-- Sample rows --\n",
            "transaction_id account_id receiver_id    amount sender_country receiver_country       channel account_type    transaction_time  is_weekend  high_value  international previous_transaction_time  time_diff_minutes previous_receiver_country  country_changed  same_account_multiple_txn  is_fraud\n",
            "      T0075722      A4029       A6866 260858.64          India        Singapore Bank Transfer      Current 2024-08-04 16:37:51           1           0              1       2024-08-04 16:14:40              23.18                   Germany                1                          1         1\n",
            "      T0080185      A4210       A8211 103492.63      Singapore        Singapore        Online      Savings 2024-03-18 15:30:05           0           0              0       2024-03-18 14:19:01              71.07                   Nigeria                1                          1         1\n",
            "      T0019865      A1792       A9160 272212.01             UK            China    Mobile App      Savings 2024-11-28 23:24:01           0           0              1       2024-11-28 22:32:19              51.70                       USA                1                          1         1\n",
            "      T0076700      A4068       A4543  14355.62          India            China    Mobile App      Savings 2023-07-30 17:29:14           1           0              1       2023-07-30 15:32:44             116.50                       USA                1                          1         1\n",
            "      T0092992      A4720       A9574 518332.04      Singapore               UK           ATM     Business 2024-10-22 13:19:34           0           1              1       2024-10-22 11:04:07             135.45                     India                1                          0         0\n",
            "      T0076435      A4058       A2593 111780.45            USA          Germany        Online      Current 2024-05-29 00:27:21           0           0              1       2024-05-28 23:20:30              66.85                 Singapore                1                          1         1\n",
            "      T0084005      A4359       A2855 398742.94          India            China    Mobile App     Offshore 2024-03-21 23:17:35           0           0              1       2024-03-21 23:16:35               1.00                       USA                1                          1         1\n",
            "      T0080918      A4240       A9250 358047.79        Nigeria          Germany Bank Transfer     Business 2024-04-29 03:55:22           0           0              1       2024-04-27 19:24:28            1950.90                       USA                1                          0         0\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 🧹 Data Cleaning\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Load the AML transaction dataset, handle missing values, and display the shape of the data to verify successful loading.\n"
      ],
      "metadata": {
        "id": "9OYpK5kYWnWS"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "df = pd.read_csv(\"aml_transactions_full_100k.csv\")\n",
        "df['previous_receiver_country'].fillna('Unknown', inplace=True)\n",
        "print(\"✅ Dataset loaded. Shape:\", df.shape)\n",
        "df.head()"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 452
        },
        "id": "-WsA3JX0TmvP",
        "outputId": "0d4e2943-1540-4673-bd5a-6f4985acbf48"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stderr",
          "text": [
            "/tmp/ipython-input-2016814715.py:2: FutureWarning: A value is trying to be set on a copy of a DataFrame or Series through chained assignment using an inplace method.\n",
            "The behavior will change in pandas 3.0. This inplace method will never work because the intermediate object on which we are setting values always behaves as a copy.\n",
            "\n",
            "For example, when doing 'df[col].method(value, inplace=True)', try using 'df.method({col: value}, inplace=True)' or df[col] = df[col].method(value) instead, to perform the operation inplace on the original object.\n",
            "\n",
            "\n",
            "  df['previous_receiver_country'].fillna('Unknown', inplace=True)\n"
          ]
        },
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "✅ Dataset loaded. Shape: (100000, 18)\n"
          ]
        },
        {
          "output_type": "execute_result",
          "data": {
            "text/plain": [
              "  transaction_id account_id receiver_id     amount sender_country  \\\n",
              "0       T0075722      A4029       A6866  260858.64          India   \n",
              "1       T0080185      A4210       A8211  103492.63      Singapore   \n",
              "2       T0019865      A1792       A9160  272212.01             UK   \n",
              "3       T0076700      A4068       A4543   14355.62          India   \n",
              "4       T0092992      A4720       A9574  518332.04      Singapore   \n",
              "\n",
              "  receiver_country        channel account_type     transaction_time  \\\n",
              "0        Singapore  Bank Transfer      Current  2024-08-04 16:37:51   \n",
              "1        Singapore         Online      Savings  2024-03-18 15:30:05   \n",
              "2            China     Mobile App      Savings  2024-11-28 23:24:01   \n",
              "3            China     Mobile App      Savings  2023-07-30 17:29:14   \n",
              "4               UK            ATM     Business  2024-10-22 13:19:34   \n",
              "\n",
              "   is_weekend  high_value  international previous_transaction_time  \\\n",
              "0           1           0              1       2024-08-04 16:14:40   \n",
              "1           0           0              0       2024-03-18 14:19:01   \n",
              "2           0           0              1       2024-11-28 22:32:19   \n",
              "3           1           0              1       2023-07-30 15:32:44   \n",
              "4           0           1              1       2024-10-22 11:04:07   \n",
              "\n",
              "   time_diff_minutes previous_receiver_country  country_changed  \\\n",
              "0              23.18                   Germany                1   \n",
              "1              71.07                   Nigeria                1   \n",
              "2              51.70                       USA                1   \n",
              "3             116.50                       USA                1   \n",
              "4             135.45                     India                1   \n",
              "\n",
              "   same_account_multiple_txn  is_fraud  \n",
              "0                          1         1  \n",
              "1                          1         1  \n",
              "2                          1         1  \n",
              "3                          1         1  \n",
              "4                          0         0  "
            ],
            "text/html": [
              "\n",
              "  <div id=\"df-4388cca2-9965-417b-ac68-de72ad1bd3c2\" class=\"colab-df-container\">\n",
              "    <div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>transaction_id</th>\n",
              "      <th>account_id</th>\n",
              "      <th>receiver_id</th>\n",
              "      <th>amount</th>\n",
              "      <th>sender_country</th>\n",
              "      <th>receiver_country</th>\n",
              "      <th>channel</th>\n",
              "      <th>account_type</th>\n",
              "      <th>transaction_time</th>\n",
              "      <th>is_weekend</th>\n",
              "      <th>high_value</th>\n",
              "      <th>international</th>\n",
              "      <th>previous_transaction_time</th>\n",
              "      <th>time_diff_minutes</th>\n",
              "      <th>previous_receiver_country</th>\n",
              "      <th>country_changed</th>\n",
              "      <th>same_account_multiple_txn</th>\n",
              "      <th>is_fraud</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>T0075722</td>\n",
              "      <td>A4029</td>\n",
              "      <td>A6866</td>\n",
              "      <td>260858.64</td>\n",
              "      <td>India</td>\n",
              "      <td>Singapore</td>\n",
              "      <td>Bank Transfer</td>\n",
              "      <td>Current</td>\n",
              "      <td>2024-08-04 16:37:51</td>\n",
              "      <td>1</td>\n",
              "      <td>0</td>\n",
              "      <td>1</td>\n",
              "      <td>2024-08-04 16:14:40</td>\n",
              "      <td>23.18</td>\n",
              "      <td>Germany</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>1</th>\n",
              "      <td>T0080185</td>\n",
              "      <td>A4210</td>\n",
              "      <td>A8211</td>\n",
              "      <td>103492.63</td>\n",
              "      <td>Singapore</td>\n",
              "      <td>Singapore</td>\n",
              "      <td>Online</td>\n",
              "      <td>Savings</td>\n",
              "      <td>2024-03-18 15:30:05</td>\n",
              "      <td>0</td>\n",
              "      <td>0</td>\n",
              "      <td>0</td>\n",
              "      <td>2024-03-18 14:19:01</td>\n",
              "      <td>71.07</td>\n",
              "      <td>Nigeria</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>T0019865</td>\n",
              "      <td>A1792</td>\n",
              "      <td>A9160</td>\n",
              "      <td>272212.01</td>\n",
              "      <td>UK</td>\n",
              "      <td>China</td>\n",
              "      <td>Mobile App</td>\n",
              "      <td>Savings</td>\n",
              "      <td>2024-11-28 23:24:01</td>\n",
              "      <td>0</td>\n",
              "      <td>0</td>\n",
              "      <td>1</td>\n",
              "      <td>2024-11-28 22:32:19</td>\n",
              "      <td>51.70</td>\n",
              "      <td>USA</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>3</th>\n",
              "      <td>T0076700</td>\n",
              "      <td>A4068</td>\n",
              "      <td>A4543</td>\n",
              "      <td>14355.62</td>\n",
              "      <td>India</td>\n",
              "      <td>China</td>\n",
              "      <td>Mobile App</td>\n",
              "      <td>Savings</td>\n",
              "      <td>2023-07-30 17:29:14</td>\n",
              "      <td>1</td>\n",
              "      <td>0</td>\n",
              "      <td>1</td>\n",
              "      <td>2023-07-30 15:32:44</td>\n",
              "      <td>116.50</td>\n",
              "      <td>USA</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>T0092992</td>\n",
              "      <td>A4720</td>\n",
              "      <td>A9574</td>\n",
              "      <td>518332.04</td>\n",
              "      <td>Singapore</td>\n",
              "      <td>UK</td>\n",
              "      <td>ATM</td>\n",
              "      <td>Business</td>\n",
              "      <td>2024-10-22 13:19:34</td>\n",
              "      <td>0</td>\n",
              "      <td>1</td>\n",
              "      <td>1</td>\n",
              "      <td>2024-10-22 11:04:07</td>\n",
              "      <td>135.45</td>\n",
              "      <td>India</td>\n",
              "      <td>1</td>\n",
              "      <td>0</td>\n",
              "      <td>0</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>\n",
              "    <div class=\"colab-df-buttons\">\n",
              "\n",
              "  <div class=\"colab-df-container\">\n",
              "    <button class=\"colab-df-convert\" onclick=\"convertToInteractive('df-4388cca2-9965-417b-ac68-de72ad1bd3c2')\"\n",
              "            title=\"Convert this dataframe to an interactive table.\"\n",
              "            style=\"display:none;\">\n",
              "\n",
              "  <svg xmlns=\"http://www.w3.org/2000/svg\" height=\"24px\" viewBox=\"0 -960 960 960\">\n",
              "    <path d=\"M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z\"/>\n",
              "  </svg>\n",
              "    </button>\n",
              "\n",
              "  <style>\n",
              "    .colab-df-container {\n",
              "      display:flex;\n",
              "      gap: 12px;\n",
              "    }\n",
              "\n",
              "    .colab-df-convert {\n",
              "      background-color: #E8F0FE;\n",
              "      border: none;\n",
              "      border-radius: 50%;\n",
              "      cursor: pointer;\n",
              "      display: none;\n",
              "      fill: #1967D2;\n",
              "      height: 32px;\n",
              "      padding: 0 0 0 0;\n",
              "      width: 32px;\n",
              "    }\n",
              "\n",
              "    .colab-df-convert:hover {\n",
              "      background-color: #E2EBFA;\n",
              "      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);\n",
              "      fill: #174EA6;\n",
              "    }\n",
              "\n",
              "    .colab-df-buttons div {\n",
              "      margin-bottom: 4px;\n",
              "    }\n",
              "\n",
              "    [theme=dark] .colab-df-convert {\n",
              "      background-color: #3B4455;\n",
              "      fill: #D2E3FC;\n",
              "    }\n",
              "\n",
              "    [theme=dark] .colab-df-convert:hover {\n",
              "      background-color: #434B5C;\n",
              "      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);\n",
              "      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));\n",
              "      fill: #FFFFFF;\n",
              "    }\n",
              "  </style>\n",
              "\n",
              "    <script>\n",
              "      const buttonEl =\n",
              "        document.querySelector('#df-4388cca2-9965-417b-ac68-de72ad1bd3c2 button.colab-df-convert');\n",
              "      buttonEl.style.display =\n",
              "        google.colab.kernel.accessAllowed ? 'block' : 'none';\n",
              "\n",
              "      async function convertToInteractive(key) {\n",
              "        const element = document.querySelector('#df-4388cca2-9965-417b-ac68-de72ad1bd3c2');\n",
              "        const dataTable =\n",
              "          await google.colab.kernel.invokeFunction('convertToInteractive',\n",
              "                                                    [key], {});\n",
              "        if (!dataTable) return;\n",
              "\n",
              "        const docLinkHtml = 'Like what you see? Visit the ' +\n",
              "          '<a target=\"_blank\" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'\n",
              "          + ' to learn more about interactive tables.';\n",
              "        element.innerHTML = '';\n",
              "        dataTable['output_type'] = 'display_data';\n",
              "        await google.colab.output.renderOutput(dataTable, element);\n",
              "        const docLink = document.createElement('div');\n",
              "        docLink.innerHTML = docLinkHtml;\n",
              "        element.appendChild(docLink);\n",
              "      }\n",
              "    </script>\n",
              "  </div>\n",
              "\n",
              "\n",
              "    <div id=\"df-c2b4183e-2871-4d42-b119-7d46478af901\">\n",
              "      <button class=\"colab-df-quickchart\" onclick=\"quickchart('df-c2b4183e-2871-4d42-b119-7d46478af901')\"\n",
              "                title=\"Suggest charts\"\n",
              "                style=\"display:none;\">\n",
              "\n",
              "<svg xmlns=\"http://www.w3.org/2000/svg\" height=\"24px\"viewBox=\"0 0 24 24\"\n",
              "     width=\"24px\">\n",
              "    <g>\n",
              "        <path d=\"M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z\"/>\n",
              "    </g>\n",
              "</svg>\n",
              "      </button>\n",
              "\n",
              "<style>\n",
              "  .colab-df-quickchart {\n",
              "      --bg-color: #E8F0FE;\n",
              "      --fill-color: #1967D2;\n",
              "      --hover-bg-color: #E2EBFA;\n",
              "      --hover-fill-color: #174EA6;\n",
              "      --disabled-fill-color: #AAA;\n",
              "      --disabled-bg-color: #DDD;\n",
              "  }\n",
              "\n",
              "  [theme=dark] .colab-df-quickchart {\n",
              "      --bg-color: #3B4455;\n",
              "      --fill-color: #D2E3FC;\n",
              "      --hover-bg-color: #434B5C;\n",
              "      --hover-fill-color: #FFFFFF;\n",
              "      --disabled-bg-color: #3B4455;\n",
              "      --disabled-fill-color: #666;\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart {\n",
              "    background-color: var(--bg-color);\n",
              "    border: none;\n",
              "    border-radius: 50%;\n",
              "    cursor: pointer;\n",
              "    display: none;\n",
              "    fill: var(--fill-color);\n",
              "    height: 32px;\n",
              "    padding: 0;\n",
              "    width: 32px;\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart:hover {\n",
              "    background-color: var(--hover-bg-color);\n",
              "    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);\n",
              "    fill: var(--button-hover-fill-color);\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart-complete:disabled,\n",
              "  .colab-df-quickchart-complete:disabled:hover {\n",
              "    background-color: var(--disabled-bg-color);\n",
              "    fill: var(--disabled-fill-color);\n",
              "    box-shadow: none;\n",
              "  }\n",
              "\n",
              "  .colab-df-spinner {\n",
              "    border: 2px solid var(--fill-color);\n",
              "    border-color: transparent;\n",
              "    border-bottom-color: var(--fill-color);\n",
              "    animation:\n",
              "      spin 1s steps(1) infinite;\n",
              "  }\n",
              "\n",
              "  @keyframes spin {\n",
              "    0% {\n",
              "      border-color: transparent;\n",
              "      border-bottom-color: var(--fill-color);\n",
              "      border-left-color: var(--fill-color);\n",
              "    }\n",
              "    20% {\n",
              "      border-color: transparent;\n",
              "      border-left-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "    }\n",
              "    30% {\n",
              "      border-color: transparent;\n",
              "      border-left-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "      border-right-color: var(--fill-color);\n",
              "    }\n",
              "    40% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "    }\n",
              "    60% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "    }\n",
              "    80% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "      border-bottom-color: var(--fill-color);\n",
              "    }\n",
              "    90% {\n",
              "      border-color: transparent;\n",
              "      border-bottom-color: var(--fill-color);\n",
              "    }\n",
              "  }\n",
              "</style>\n",
              "\n",
              "      <script>\n",
              "        async function quickchart(key) {\n",
              "          const quickchartButtonEl =\n",
              "            document.querySelector('#' + key + ' button');\n",
              "          quickchartButtonEl.disabled = true;  // To prevent multiple clicks.\n",
              "          quickchartButtonEl.classList.add('colab-df-spinner');\n",
              "          try {\n",
              "            const charts = await google.colab.kernel.invokeFunction(\n",
              "                'suggestCharts', [key], {});\n",
              "          } catch (error) {\n",
              "            console.error('Error during call to suggestCharts:', error);\n",
              "          }\n",
              "          quickchartButtonEl.classList.remove('colab-df-spinner');\n",
              "          quickchartButtonEl.classList.add('colab-df-quickchart-complete');\n",
              "        }\n",
              "        (() => {\n",
              "          let quickchartButtonEl =\n",
              "            document.querySelector('#df-c2b4183e-2871-4d42-b119-7d46478af901 button');\n",
              "          quickchartButtonEl.style.display =\n",
              "            google.colab.kernel.accessAllowed ? 'block' : 'none';\n",
              "        })();\n",
              "      </script>\n",
              "    </div>\n",
              "\n",
              "    </div>\n",
              "  </div>\n"
            ],
            "application/vnd.google.colaboratory.intrinsic+json": {
              "type": "dataframe",
              "variable_name": "df",
              "summary": "{\n  \"name\": \"df\",\n  \"rows\": 100000,\n  \"fields\": [\n    {\n      \"column\": \"transaction_id\",\n      \"properties\": {\n        \"dtype\": \"string\",\n        \"num_unique_values\": 100000,\n        \"samples\": [\n          \"T0048299\",\n          \"T0081048\",\n          \"T0092755\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"account_id\",\n      \"properties\": {\n        \"dtype\": \"category\",\n        \"num_unique_values\": 4000,\n        \"samples\": [\n          \"A3096\",\n          \"A2777\",\n          \"A4861\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"receiver_id\",\n      \"properties\": {\n        \"dtype\": \"category\",\n        \"num_unique_values\": 8000,\n        \"samples\": [\n          \"A2612\",\n          \"A4206\",\n          \"A2387\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"amount\",\n      \"properties\": {\n        \"dtype\": \"number\",\n        \"std\": 288283.52897280926,\n        \"min\": 116.1,\n        \"max\": 999988.86,\n        \"num_unique_values\": 99948,\n        \"samples\": [\n          806862.02,\n          495495.94,\n          944171.71\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"sender_country\",\n      \"properties\": {\n        \"dtype\": \"category\",\n        \"num_unique_values\": 8,\n        \"samples\": [\n          \"Singapore\",\n          \"Germany\",\n          \"India\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"receiver_country\",\n      \"properties\": {\n        \"dtype\": \"category\",\n        \"num_unique_values\": 8,\n        \"samples\": [\n          \"China\",\n          \"Nigeria\",\n          \"Singapore\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"channel\",\n      \"properties\": {\n        \"dtype\": \"category\",\n        \"num_unique_values\": 4,\n        \"samples\": [\n          \"Online\",\n          \"ATM\",\n          \"Bank Transfer\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"account_type\",\n      \"properties\": {\n        \"dtype\": \"category\",\n        \"num_unique_values\": 4,\n        \"samples\": [\n          \"Savings\",\n          \"Offshore\",\n          \"Current\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"transaction_time\",\n      \"properties\": {\n        \"dtype\": \"object\",\n        \"num_unique_values\": 99922,\n        \"samples\": [\n          \"2025-04-15 00:42:12\",\n          \"2023-06-02 20:10:15\",\n          \"2023-02-15 11:45:04\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"is_weekend\",\n      \"properties\": {\n        \"dtype\": \"number\",\n        \"std\": 0,\n        \"min\": 0,\n        \"max\": 1,\n        \"num_unique_values\": 2,\n        \"samples\": [\n          0,\n          1\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"high_value\",\n      \"properties\": {\n        \"dtype\": \"number\",\n        \"std\": 0,\n        \"min\": 0,\n        \"max\": 1,\n        \"num_unique_values\": 2,\n        \"samples\": [\n          1,\n          0\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"international\",\n      \"properties\": {\n        \"dtype\": \"number\",\n        \"std\": 0,\n        \"min\": 0,\n        \"max\": 1,\n        \"num_unique_values\": 2,\n        \"samples\": [\n          0,\n          1\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"previous_transaction_time\",\n      \"properties\": {\n        \"dtype\": \"object\",\n        \"num_unique_values\": 95928,\n        \"samples\": [\n          \"2024-04-07 23:31:55\",\n          \"2024-12-11 05:11:25\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"time_diff_minutes\",\n      \"properties\": {\n        \"dtype\": \"number\",\n        \"std\": 8446.903327639127,\n        \"min\": -1445123.53,\n        \"max\": 15316.38,\n        \"num_unique_values\": 28651,\n        \"samples\": [\n          172.38,\n          70.97\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"previous_receiver_country\",\n      \"properties\": {\n        \"dtype\": \"category\",\n        \"num_unique_values\": 9,\n        \"samples\": [\n          \"UK\",\n          \"Nigeria\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"country_changed\",\n      \"properties\": {\n        \"dtype\": \"number\",\n        \"std\": 0,\n        \"min\": 0,\n        \"max\": 1,\n        \"num_unique_values\": 2,\n        \"samples\": [\n          0,\n          1\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"same_account_multiple_txn\",\n      \"properties\": {\n        \"dtype\": \"number\",\n        \"std\": 0,\n        \"min\": 0,\n        \"max\": 1,\n        \"num_unique_values\": 2,\n        \"samples\": [\n          0,\n          1\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"is_fraud\",\n      \"properties\": {\n        \"dtype\": \"number\",\n        \"std\": 0,\n        \"min\": 0,\n        \"max\": 1,\n        \"num_unique_values\": 2,\n        \"samples\": [\n          0,\n          1\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    }\n  ]\n}"
            }
          },
          "metadata": {},
          "execution_count": 4
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 🔎 Feature Selection and Data Preparation\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Drop irrelevant columns (IDs, timestamps, etc.) and prepare features (`X`) and labels (`y`) for training.\n"
      ],
      "metadata": {
        "id": "Sqmrpj2VWvQ8"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "cols_to_drop = ['transaction_id', 'account_id', 'receiver_id', 'transaction_time', 'previous_transaction_time']\n",
        "X = df.drop(columns=cols_to_drop + ['is_fraud'])\n",
        "y = df['is_fraud']\n",
        "\n",
        "cat_cols = ['sender_country', 'receiver_country', 'channel', 'account_type', 'previous_receiver_country']\n",
        "for col in cat_cols:\n",
        "    X[col] = LabelEncoder().fit_transform(X[col])\n",
        "\n",
        "print(\"✅ Features ready\")"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "vZohMXoETsDo",
        "outputId": "fe6f5c09-7ac7-4d3a-e01d-0cd8837225f9"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "✅ Features ready\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## ✂️ Train-Test Split\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Split the dataset into training and testing subsets (80-20 ratio) to evaluate model performance fairly.\n"
      ],
      "metadata": {
        "id": "S10AC854W6ZP"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "X_train, X_test, y_train, y_test = train_test_split(\n",
        "    X, y, test_size=0.2, stratify=y, random_state=42\n",
        ")\n",
        "print(\"✅ Train/Test split done\")"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "T1kBlBdATvfK",
        "outputId": "4d5dd137-5051-4eba-a0b1-8f0b9b6fe962"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "✅ Train/Test split done\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 🤖 Train the XGBoost Model\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Train an XGBoost classifier to detect potentially fraudulent (money-laundering) transactions using optimized parameters.\n"
      ],
      "metadata": {
        "id": "5rSeBOyuW_Ft"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "xgb_model = xgb.XGBClassifier(\n",
        "    n_estimators=200,\n",
        "    max_depth=10,\n",
        "    scale_pos_weight=(y_train==0).sum()/(y_train==1).sum(),\n",
        "    random_state=42,\n",
        "    use_label_encoder=False,\n",
        "    eval_metric='logloss'\n",
        ")\n",
        "\n",
        "xgb_model.fit(X_train, y_train)\n",
        "print(\"✅ XGBoost trained successfully\")"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "FHyk7zr5TyuT",
        "outputId": "21e7930c-f6b3-4879-c8d2-ee7efef4280c"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stderr",
          "text": [
            "/usr/local/lib/python3.12/dist-packages/xgboost/training.py:183: UserWarning: [13:33:08] WARNING: /workspace/src/learner.cc:738: \n",
            "Parameters: { \"use_label_encoder\" } are not used.\n",
            "\n",
            "  bst.update(dtrain, iteration=i, fobj=obj)\n"
          ]
        },
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "✅ XGBoost trained successfully\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 📈 Evaluate Model Performance\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Make predictions and generate a classification report showing accuracy, precision, recall, and F1-score.\n"
      ],
      "metadata": {
        "id": "rcxwXBchXB42"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "y_pred = xgb_model.predict(X_test)\n",
        "y_proba = xgb_model.predict_proba(X_test)[:,1]\n",
        "\n",
        "print(\"📊 Classification Report:\\n\", classification_report(y_test, y_pred))\n",
        "print(\"🎯 ROC-AUC Score:\", roc_auc_score(y_test, y_proba))\n",
        "\n",
        "cm = confusion_matrix(y_test, y_pred)\n",
        "plt.figure(figsize=(6,5))\n",
        "sns.heatmap(cm, annot=True, fmt='d', cmap='Blues')\n",
        "plt.xlabel(\"Predicted\")\n",
        "plt.ylabel(\"Actual\")\n",
        "plt.title(\"Confusion Matrix\")\n",
        "plt.show()"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 678
        },
        "id": "Zf5rh03KT2C_",
        "outputId": "a2091d3b-2cd5-42e3-bb7c-943127fc149c"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "📊 Classification Report:\n",
            "               precision    recall  f1-score   support\n",
            "\n",
            "           0       0.89      0.89      0.89      6762\n",
            "           1       0.94      0.94      0.94     13238\n",
            "\n",
            "    accuracy                           0.93     20000\n",
            "   macro avg       0.92      0.92      0.92     20000\n",
            "weighted avg       0.93      0.93      0.93     20000\n",
            "\n",
            "🎯 ROC-AUC Score: 0.9501879320012983\n"
          ]
        },
        {
          "output_type": "display_data",
          "data": {
            "text/plain": [
              "<Figure size 600x500 with 2 Axes>"
            ],
            "image/png": "iVBORw0KGgoAAAANSUhEUgAAAhMAAAHWCAYAAADNbgu+AAAAOnRFWHRTb2Z0d2FyZQBNYXRwbG90bGliIHZlcnNpb24zLjEwLjAsIGh0dHBzOi8vbWF0cGxvdGxpYi5vcmcvlHJYcgAAAAlwSFlzAAAPYQAAD2EBqD+naQAAR9VJREFUeJzt3X98T/X///H7a5v9MLYZtlkxC2El8iOW37VMfmRREmVq0g8qP0NFSz9Wk+S31LuokH6REJbFkoWW+RWiSMWGmDFss53vH757fbzaZPM6Mzq3a5dzufR6nud5nud5tXjs8TjPc2yGYRgCAAC4RC5lPQEAAHB1I5gAAABOIZgAAABOIZgAAABOIZgAAABOIZgAAABOIZgAAABOIZgAAABOIZgAAABOIZgAimn37t3q0KGDfH19ZbPZtGjRIlPH37dvn2w2m2bPnm3quFezdu3aqV27dmU9DQAXQTCBq8qvv/6qRx99VNddd508PT3l4+Ojli1batKkSTp9+nSpnjs6Olpbt27VK6+8og8//FBNmzYt1fNdTv369ZPNZpOPj0+R3+Pu3btls9lks9n0xhtvlHj8AwcOKDY2VqmpqSbMFsCVxq2sJwAU19KlS3XvvffKw8NDffv21Y033qicnBytXbtWI0aM0Pbt2zVr1qxSOffp06eVnJys5557ToMGDSqVc4SEhOj06dMqV65cqYx/MW5ubjp16pS++uor9ezZ02Hf3Llz5enpqTNnzlzS2AcOHNCLL76omjVrqlGjRsU+buXKlZd0PgCXF8EErgp79+5Vr169FBISosTERFWrVs2+b+DAgdqzZ4+WLl1aauc/fPiwJMnPz6/UzmGz2eTp6Vlq41+Mh4eHWrZsqfnz5xcKJubNm6fOnTvr888/vyxzOXXqlMqXLy93d/fLcj4AzqHMgatCfHy8Tp48qf/9738OgUSB2rVr6+mnn7Z/Pnv2rF566SXVqlVLHh4eqlmzpp599lllZ2c7HFezZk116dJFa9eu1S233CJPT09dd911+uCDD+x9YmNjFRISIkkaMWKEbDabatasKelceaDg388XGxsrm83m0JaQkKBWrVrJz89PFSpUUN26dfXss8/a91/ononExES1bt1a3t7e8vPzU7du3bRjx44iz7dnzx7169dPfn5+8vX11UMPPaRTp05d+Iv9h969e+vrr79WRkaGvW3jxo3avXu3evfuXaj/0aNHNXz4cDVo0EAVKlSQj4+P7rzzTm3evNneZ/Xq1WrWrJkk6aGHHrKXSwqus127drrxxhuVkpKiNm3aqHz58vbv5Z/3TERHR8vT07PQ9UdGRqpSpUo6cOBAsa8VgHkIJnBV+Oqrr3Tdddfp1ltvLVb//v37a+zYsWrcuLEmTpyotm3bKi4uTr169SrUd8+ePbrnnnt0xx13aMKECapUqZL69eun7du3S5K6d++uiRMnSpLuv/9+ffjhh3rrrbdKNP/t27erS5cuys7O1rhx4zRhwgTddddd+v777//1uG+++UaRkZE6dOiQYmNjNXToUK1bt04tW7bUvn37CvXv2bOnTpw4obi4OPXs2VOzZ8/Wiy++WOx5du/eXTabTV988YW9bd68eapXr54aN25cqP9vv/2mRYsWqUuXLnrzzTc1YsQIbd26VW3btrX/xV6/fn2NGzdOkjRgwAB9+OGH+vDDD9WmTRv7OH///bfuvPNONWrUSG+99Zbat29f5PwmTZqkqlWrKjo6Wnl5eZKkt99+WytXrtSUKVMUHBxc7GsFYCIDuMIdP37ckGR069atWP1TU1MNSUb//v0d2ocPH25IMhITE+1tISEhhiQjKSnJ3nbo0CHDw8PDGDZsmL1t7969hiRj/PjxDmNGR0cbISEhhebwwgsvGOf/7zVx4kRDknH48OELzrvgHO+//769rVGjRkZAQIDx999/29s2b95suLi4GH379i10vocffthhzLvvvtuoXLnyBc95/nV4e3sbhmEY99xzj3H77bcbhmEYeXl5RlBQkPHiiy8W+R2cOXPGyMvLK3QdHh4exrhx4+xtGzduLHRtBdq2bWtIMmbOnFnkvrZt2zq0rVixwpBkvPzyy8Zvv/1mVKhQwYiKirroNQIoPWQmcMXLzMyUJFWsWLFY/ZctWyZJGjp0qEP7sGHDJKnQvRVhYWFq3bq1/XPVqlVVt25d/fbbb5c8538quNfiyy+/VH5+frGOOXjwoFJTU9WvXz/5+/vb22+66Sbdcccd9us832OPPebwuXXr1vr777/t32Fx9O7dW6tXr1ZaWpoSExOVlpZWZIlDOnefhYvLuT9G8vLy9Pfff9tLOD/99FOxz+nh4aGHHnqoWH07dOigRx99VOPGjVP37t3l6empt99+u9jnAmA+gglc8Xx8fCRJJ06cKFb/33//XS4uLqpdu7ZDe1BQkPz8/PT77787tNeoUaPQGJUqVdKxY8cuccaF3XfffWrZsqX69++vwMBA9erVS5988sm/BhYF86xbt26hffXr19eRI0eUlZXl0P7Pa6lUqZIklehaOnXqpIoVK2rBggWaO3eumjVrVui7LJCfn6+JEyeqTp068vDwUJUqVVS1alVt2bJFx48fL/Y5r7nmmhLdbPnGG2/I399fqampmjx5sgICAop9LADzEUzgiufj46Pg4GBt27atRMf98wbIC3F1dS2y3TCMSz5HQT2/gJeXl5KSkvTNN9/owQcf1JYtW3TffffpjjvuKNTXGc5cSwEPDw91795dc+bM0cKFCy+YlZCkV199VUOHDlWbNm300UcfacWKFUpISNANN9xQ7AyMdO77KYlNmzbp0KFDkqStW7eW6FgA5iOYwFWhS5cu+vXXX5WcnHzRviEhIcrPz9fu3bsd2tPT05WRkWFfmWGGSpUqOax8KPDP7Ickubi46Pbbb9ebb76pn3/+Wa+88ooSExP17bffFjl2wTx37dpVaN/OnTtVpUoVeXt7O3cBF9C7d29t2rRJJ06cKPKm1QKfffaZ2rdvr//973/q1auXOnTooIiIiELfSXEDu+LIysrSQw89pLCwMA0YMEDx8fHauHGjaeMDKDmCCVwVnnnmGXl7e6t///5KT08vtP/XX3/VpEmTJJ1L00sqtOLizTfflCR17tzZtHnVqlVLx48f15YtW+xtBw8e1MKFCx36HT16tNCxBQ9v+udy1QLVqlVTo0aNNGfOHIe/nLdt26aVK1far7M0tG/fXi+99JKmTp2qoKCgC/ZzdXUtlPX49NNP9ddffzm0FQQ9RQVeJTVy5Ejt379fc+bM0ZtvvqmaNWsqOjr6gt8jgNLHQ6twVahVq5bmzZun++67T/Xr13d4Aua6dev06aefql+/fpKkhg0bKjo6WrNmzVJGRobatm2rDRs2aM6cOYqKirrgssNL0atXL40cOVJ33323nnrqKZ06dUozZszQ9ddf73AD4rhx45SUlKTOnTsrJCREhw4d0vTp03XttdeqVatWFxx//PjxuvPOOxUeHq6YmBidPn1aU6ZMka+vr2JjY027jn9ycXHR888/f9F+Xbp00bhx4/TQQw/p1ltv1datWzV37lxdd911Dv1q1aol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w+OS5gHAuggmACd5e3urdu3axerbuHFjLViwQAEBAfLx8SmyT7Vq1bR+/Xq1adNGknT27FmlpKSocePGRfZv0KCB8vPztWbNGkVERBTaX5AZycvLs7eFhYXJw8ND+/fvv2BGo379+lq8eLFD2w8//HDxiwRgOdyACVxGffr0UZUqVdStWzd999132rt3r1avXq2nnnpKf/75pyTp6aef1muvvaZFixZp586deuKJJ/71GRE1a9ZUdHS0Hn74YS1atMg+5ieffCJJCgkJkc1m05IlS3T48GGdPHlSFStW1PDhwzVkyBDNmTNHv/76q3766SdNmTJFc+bMkSQ99thj2r17t0aMGKFdu3Zp3rx5mj17dml/RQCuQgQTwGVUvnx5JSUlqUaNGurevbvq16+vmJgYnTlzxp6pGDZsmB588EFFR0crPDxcFStW1N133/2v486YMUP33HOPnnjiCdWrV0+PPPKIsrKyJEnXXHONXnzxRY0aNUqBgYEaNGiQJOmll17SmDFjFBcXp/r166tjx45aunSpQkNDJUk1atTQ559/rkWLFqlhw4aaOXOmXn311VL8dgBcrWzGhe7qAgAAKAYyEwAAwCkEEwAAwCkEEwAAwCkEEwAAwCkEEwAAwCkEEwAAwCkEEwAAwCkEEwAAwCkEEwAAwCkEEwAAwCkEEwAAwCn/DwMMiakau1OLAAAAAElFTkSuQmCC\n"
          },
          "metadata": {}
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 💡 Analyze Feature Importance\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Display feature importance to understand which transaction features contribute most to detecting money laundering.\n"
      ],
      "metadata": {
        "id": "xL4rysBoXG5G"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "importance_xgb = pd.DataFrame({\n",
        "    'Feature': X.columns,\n",
        "    'Importance': xgb_model.feature_importances_\n",
        "}).sort_values(by='Importance', ascending=False)\n",
        "\n",
        "display(importance_xgb.head(10))\n",
        "\n",
        "plt.figure(figsize=(10,5))\n",
        "sns.barplot(data=importance_xgb.head(10), x='Importance', y='Feature')\n",
        "plt.title(\"Top 10 Features for Fraud Detection\")\n",
        "plt.show()"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/",
          "height": 833
        },
        "id": "hJRgr3WfT6MQ",
        "outputId": "7e5ec611-9e8a-4c16-cb23-b7d2496d5ccf"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "display_data",
          "data": {
            "text/plain": [
              "                      Feature  Importance\n",
              "11  same_account_multiple_txn    0.741474\n",
              "10            country_changed    0.126660\n",
              "6                  high_value    0.107323\n",
              "7               international    0.010068\n",
              "4                account_type    0.005540\n",
              "5                  is_weekend    0.002052\n",
              "0                      amount    0.001287\n",
              "9   previous_receiver_country    0.001167\n",
              "8           time_diff_minutes    0.001152\n",
              "2            receiver_country    0.001110"
            ],
            "text/html": [
              "\n",
              "  <div id=\"df-6c9c27e2-1090-46a0-bd4b-a001ec743ebb\" class=\"colab-df-container\">\n",
              "    <div>\n",
              "<style scoped>\n",
              "    .dataframe tbody tr th:only-of-type {\n",
              "        vertical-align: middle;\n",
              "    }\n",
              "\n",
              "    .dataframe tbody tr th {\n",
              "        vertical-align: top;\n",
              "    }\n",
              "\n",
              "    .dataframe thead th {\n",
              "        text-align: right;\n",
              "    }\n",
              "</style>\n",
              "<table border=\"1\" class=\"dataframe\">\n",
              "  <thead>\n",
              "    <tr style=\"text-align: right;\">\n",
              "      <th></th>\n",
              "      <th>Feature</th>\n",
              "      <th>Importance</th>\n",
              "    </tr>\n",
              "  </thead>\n",
              "  <tbody>\n",
              "    <tr>\n",
              "      <th>11</th>\n",
              "      <td>same_account_multiple_txn</td>\n",
              "      <td>0.741474</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>10</th>\n",
              "      <td>country_changed</td>\n",
              "      <td>0.126660</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>6</th>\n",
              "      <td>high_value</td>\n",
              "      <td>0.107323</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>7</th>\n",
              "      <td>international</td>\n",
              "      <td>0.010068</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>4</th>\n",
              "      <td>account_type</td>\n",
              "      <td>0.005540</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>5</th>\n",
              "      <td>is_weekend</td>\n",
              "      <td>0.002052</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>0</th>\n",
              "      <td>amount</td>\n",
              "      <td>0.001287</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>9</th>\n",
              "      <td>previous_receiver_country</td>\n",
              "      <td>0.001167</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>8</th>\n",
              "      <td>time_diff_minutes</td>\n",
              "      <td>0.001152</td>\n",
              "    </tr>\n",
              "    <tr>\n",
              "      <th>2</th>\n",
              "      <td>receiver_country</td>\n",
              "      <td>0.001110</td>\n",
              "    </tr>\n",
              "  </tbody>\n",
              "</table>\n",
              "</div>\n",
              "    <div class=\"colab-df-buttons\">\n",
              "\n",
              "  <div class=\"colab-df-container\">\n",
              "    <button class=\"colab-df-convert\" onclick=\"convertToInteractive('df-6c9c27e2-1090-46a0-bd4b-a001ec743ebb')\"\n",
              "            title=\"Convert this dataframe to an interactive table.\"\n",
              "            style=\"display:none;\">\n",
              "\n",
              "  <svg xmlns=\"http://www.w3.org/2000/svg\" height=\"24px\" viewBox=\"0 -960 960 960\">\n",
              "    <path d=\"M120-120v-720h720v720H120Zm60-500h600v-160H180v160Zm220 220h160v-160H400v160Zm0 220h160v-160H400v160ZM180-400h160v-160H180v160Zm440 0h160v-160H620v160ZM180-180h160v-160H180v160Zm440 0h160v-160H620v160Z\"/>\n",
              "  </svg>\n",
              "    </button>\n",
              "\n",
              "  <style>\n",
              "    .colab-df-container {\n",
              "      display:flex;\n",
              "      gap: 12px;\n",
              "    }\n",
              "\n",
              "    .colab-df-convert {\n",
              "      background-color: #E8F0FE;\n",
              "      border: none;\n",
              "      border-radius: 50%;\n",
              "      cursor: pointer;\n",
              "      display: none;\n",
              "      fill: #1967D2;\n",
              "      height: 32px;\n",
              "      padding: 0 0 0 0;\n",
              "      width: 32px;\n",
              "    }\n",
              "\n",
              "    .colab-df-convert:hover {\n",
              "      background-color: #E2EBFA;\n",
              "      box-shadow: 0px 1px 2px rgba(60, 64, 67, 0.3), 0px 1px 3px 1px rgba(60, 64, 67, 0.15);\n",
              "      fill: #174EA6;\n",
              "    }\n",
              "\n",
              "    .colab-df-buttons div {\n",
              "      margin-bottom: 4px;\n",
              "    }\n",
              "\n",
              "    [theme=dark] .colab-df-convert {\n",
              "      background-color: #3B4455;\n",
              "      fill: #D2E3FC;\n",
              "    }\n",
              "\n",
              "    [theme=dark] .colab-df-convert:hover {\n",
              "      background-color: #434B5C;\n",
              "      box-shadow: 0px 1px 3px 1px rgba(0, 0, 0, 0.15);\n",
              "      filter: drop-shadow(0px 1px 2px rgba(0, 0, 0, 0.3));\n",
              "      fill: #FFFFFF;\n",
              "    }\n",
              "  </style>\n",
              "\n",
              "    <script>\n",
              "      const buttonEl =\n",
              "        document.querySelector('#df-6c9c27e2-1090-46a0-bd4b-a001ec743ebb button.colab-df-convert');\n",
              "      buttonEl.style.display =\n",
              "        google.colab.kernel.accessAllowed ? 'block' : 'none';\n",
              "\n",
              "      async function convertToInteractive(key) {\n",
              "        const element = document.querySelector('#df-6c9c27e2-1090-46a0-bd4b-a001ec743ebb');\n",
              "        const dataTable =\n",
              "          await google.colab.kernel.invokeFunction('convertToInteractive',\n",
              "                                                    [key], {});\n",
              "        if (!dataTable) return;\n",
              "\n",
              "        const docLinkHtml = 'Like what you see? Visit the ' +\n",
              "          '<a target=\"_blank\" href=https://colab.research.google.com/notebooks/data_table.ipynb>data table notebook</a>'\n",
              "          + ' to learn more about interactive tables.';\n",
              "        element.innerHTML = '';\n",
              "        dataTable['output_type'] = 'display_data';\n",
              "        await google.colab.output.renderOutput(dataTable, element);\n",
              "        const docLink = document.createElement('div');\n",
              "        docLink.innerHTML = docLinkHtml;\n",
              "        element.appendChild(docLink);\n",
              "      }\n",
              "    </script>\n",
              "  </div>\n",
              "\n",
              "\n",
              "    <div id=\"df-a1afccbe-53e3-4252-92a5-b3867c9bb3cc\">\n",
              "      <button class=\"colab-df-quickchart\" onclick=\"quickchart('df-a1afccbe-53e3-4252-92a5-b3867c9bb3cc')\"\n",
              "                title=\"Suggest charts\"\n",
              "                style=\"display:none;\">\n",
              "\n",
              "<svg xmlns=\"http://www.w3.org/2000/svg\" height=\"24px\"viewBox=\"0 0 24 24\"\n",
              "     width=\"24px\">\n",
              "    <g>\n",
              "        <path d=\"M19 3H5c-1.1 0-2 .9-2 2v14c0 1.1.9 2 2 2h14c1.1 0 2-.9 2-2V5c0-1.1-.9-2-2-2zM9 17H7v-7h2v7zm4 0h-2V7h2v10zm4 0h-2v-4h2v4z\"/>\n",
              "    </g>\n",
              "</svg>\n",
              "      </button>\n",
              "\n",
              "<style>\n",
              "  .colab-df-quickchart {\n",
              "      --bg-color: #E8F0FE;\n",
              "      --fill-color: #1967D2;\n",
              "      --hover-bg-color: #E2EBFA;\n",
              "      --hover-fill-color: #174EA6;\n",
              "      --disabled-fill-color: #AAA;\n",
              "      --disabled-bg-color: #DDD;\n",
              "  }\n",
              "\n",
              "  [theme=dark] .colab-df-quickchart {\n",
              "      --bg-color: #3B4455;\n",
              "      --fill-color: #D2E3FC;\n",
              "      --hover-bg-color: #434B5C;\n",
              "      --hover-fill-color: #FFFFFF;\n",
              "      --disabled-bg-color: #3B4455;\n",
              "      --disabled-fill-color: #666;\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart {\n",
              "    background-color: var(--bg-color);\n",
              "    border: none;\n",
              "    border-radius: 50%;\n",
              "    cursor: pointer;\n",
              "    display: none;\n",
              "    fill: var(--fill-color);\n",
              "    height: 32px;\n",
              "    padding: 0;\n",
              "    width: 32px;\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart:hover {\n",
              "    background-color: var(--hover-bg-color);\n",
              "    box-shadow: 0 1px 2px rgba(60, 64, 67, 0.3), 0 1px 3px 1px rgba(60, 64, 67, 0.15);\n",
              "    fill: var(--button-hover-fill-color);\n",
              "  }\n",
              "\n",
              "  .colab-df-quickchart-complete:disabled,\n",
              "  .colab-df-quickchart-complete:disabled:hover {\n",
              "    background-color: var(--disabled-bg-color);\n",
              "    fill: var(--disabled-fill-color);\n",
              "    box-shadow: none;\n",
              "  }\n",
              "\n",
              "  .colab-df-spinner {\n",
              "    border: 2px solid var(--fill-color);\n",
              "    border-color: transparent;\n",
              "    border-bottom-color: var(--fill-color);\n",
              "    animation:\n",
              "      spin 1s steps(1) infinite;\n",
              "  }\n",
              "\n",
              "  @keyframes spin {\n",
              "    0% {\n",
              "      border-color: transparent;\n",
              "      border-bottom-color: var(--fill-color);\n",
              "      border-left-color: var(--fill-color);\n",
              "    }\n",
              "    20% {\n",
              "      border-color: transparent;\n",
              "      border-left-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "    }\n",
              "    30% {\n",
              "      border-color: transparent;\n",
              "      border-left-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "      border-right-color: var(--fill-color);\n",
              "    }\n",
              "    40% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "      border-top-color: var(--fill-color);\n",
              "    }\n",
              "    60% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "    }\n",
              "    80% {\n",
              "      border-color: transparent;\n",
              "      border-right-color: var(--fill-color);\n",
              "      border-bottom-color: var(--fill-color);\n",
              "    }\n",
              "    90% {\n",
              "      border-color: transparent;\n",
              "      border-bottom-color: var(--fill-color);\n",
              "    }\n",
              "  }\n",
              "</style>\n",
              "\n",
              "      <script>\n",
              "        async function quickchart(key) {\n",
              "          const quickchartButtonEl =\n",
              "            document.querySelector('#' + key + ' button');\n",
              "          quickchartButtonEl.disabled = true;  // To prevent multiple clicks.\n",
              "          quickchartButtonEl.classList.add('colab-df-spinner');\n",
              "          try {\n",
              "            const charts = await google.colab.kernel.invokeFunction(\n",
              "                'suggestCharts', [key], {});\n",
              "          } catch (error) {\n",
              "            console.error('Error during call to suggestCharts:', error);\n",
              "          }\n",
              "          quickchartButtonEl.classList.remove('colab-df-spinner');\n",
              "          quickchartButtonEl.classList.add('colab-df-quickchart-complete');\n",
              "        }\n",
              "        (() => {\n",
              "          let quickchartButtonEl =\n",
              "            document.querySelector('#df-a1afccbe-53e3-4252-92a5-b3867c9bb3cc button');\n",
              "          quickchartButtonEl.style.display =\n",
              "            google.colab.kernel.accessAllowed ? 'block' : 'none';\n",
              "        })();\n",
              "      </script>\n",
              "    </div>\n",
              "\n",
              "    </div>\n",
              "  </div>\n"
            ],
            "application/vnd.google.colaboratory.intrinsic+json": {
              "type": "dataframe",
              "summary": "{\n  \"name\": \"plt\",\n  \"rows\": 10,\n  \"fields\": [\n    {\n      \"column\": \"Feature\",\n      \"properties\": {\n        \"dtype\": \"string\",\n        \"num_unique_values\": 10,\n        \"samples\": [\n          \"time_diff_minutes\",\n          \"country_changed\",\n          \"is_weekend\"\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    },\n    {\n      \"column\": \"Importance\",\n      \"properties\": {\n        \"dtype\": \"float32\",\n        \"num_unique_values\": 10,\n        \"samples\": [\n          0.0011516057420521975,\n          0.12665997445583344,\n          0.002052182564511895\n        ],\n        \"semantic_type\": \"\",\n        \"description\": \"\"\n      }\n    }\n  ]\n}"
            }
          },
          "metadata": {}
        },
        {
          "output_type": "display_data",
          "data": {
            "text/plain": [
              "<Figure size 1000x500 with 1 Axes>"
            ],
            "image/png": "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47BhwzKs2kiXXvd20ttYvHhxpj+s2dk9OP+v9e2utfGfB3sCsD4Pzv8SAQDwCClZsqR++uknXbp0yWI2P33Zc8mSJS3qp88g3uqPP/6Qs7NzjpZq30m1atW0detWpaWlWcwi79y5U87OzuakNrfKlCmj6OhoNW7c+I5Llr/99lulpKRozZo1FrORty7nT3e7dtJnLS9evGjx/vH/zoJnFW9SUtIdZ73T2dvbq1WrVmrVqpXS0tLUr18/zZs3T2PHjs0ycfyvkiVLat++fRnG4XbfjfshIiJC586d06pVq9SgQQNz+dGjRzPU9fT01MWLFzOUHz9+XKVLlzZvlyxZUr///rsMw7AYx9jY2DyJefHixZJkTujTz12gQIEsx/R236syZcpIuvnDzp3aSD/X77//nqvzZKZkyZKZXpv8/F4AeDBxTz4AAPmgefPmSk1N1QcffGBR/t5778lkMqlZs2YW5Tt27NBvv/1m3j558qS++eYbPfPMM3k2O9q+fXudOnVKq1atMpedPXtWK1asUKtWre76vd0dOnTQ33//rQULFmTYd+XKFV2+fFnS/81A3jrjmJCQoLCwsAzHFSxYMNOEMj0Zu/U1g5cvX87RzHqHDh20Y8cObdiwIcO+ixcv6saNG5JuPo3+VjY2NqpSpYok5ep1cM2bN9e///6rL774wlx248YNvf/++3JxcVFwcHCO27xbmY3JtWvX9NFHH2WoW6ZMGf3yyy+6du2auey7777TyZMnLeo1b95c//zzj1auXGkuS05O1vz58+863k2bNmny5MkqVaqUunTpIulmYh4SEqJ58+ZlumLlzJkz5r8LFiwoSRm+W6GhoXJzc9OUKVN0/fr127bh5eWlBg0a6LPPPtOJEycs6tx6DW93nsw0b95cv/76q3bs2GEuu3z5subPn6+AgABVrFgxyzYAPBqYyQcAIB+0atVKDRs21BtvvKFjx46patWq+uGHH/TNN99o0KBB5iQ13WOPPabQ0FCLV+hJ0sSJE7M817fffqvo6GhJN1+ftm/fPr355puSpGeffdackLZv315PPvmkevTooYMHD6pIkSL66KOPlJqamq3zZOWll17Sl19+qVdeeUWbN29WvXr1lJqaqkOHDunLL780v6f+mWeeMc+Mv/zyy0pKStKCBQvk7e2dITmrWbOm5s6dqzfffFNly5aVt7e3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9zly65TT8Uh37NgxDRs2TPXr179PkVq/3IyFm5ub4uPjzZ/jx4/fx4itW07H49q1a3r66ad17NgxrVy5UrGxsVqwYIGKFSt2nyO3Pjkdi1WrVln8u/j9999la2ur559//j5HbkUMAPnmiSeeMF577TXzdmpqquHn52dMnTo10/odOnQwWrRoYVFWu3Zt4+WXX76ncT4KcjoWtypZsqTx3nvv3cPoHj13Mx6GYRg3btwwXF1djYULF96rEB8ZdzsWhmEY1atXN8aMGXMvwnvk5GY8bty4YdStW9f45JNPjG7duhmtW7e+D5Fav5yORVhYmOHu7n6fonv05HQ85s6da5QuXdq4du3a/QrxkXG3/9147733DFdXVyMpKelehWj1mMkH8sm1a9e0Z88eNWnSxFxmY2OjJk2aaMeOHZkes2PHDov6khQaGnrb+sie3IwF7p28GI/k5GRdv35dhQoVuldhPhLudiwMw9DGjRsVGxurBg0a3MtQHwm5HY9JkybJ29tbvXr1uh9hPhJyOxZJSUkqWbKk/P391bp1ax04cOB+hGv1cjMea9asUZ06dfTaa6/Jx8dHjz32mKZMmaLU1NT7FbZVyov/hn/66afq1KmTChYseK/CtHok+UA+OXv2rFJTU+Xj42NR7uPjo3///TfTY/79998c1Uf25GYscO/kxXiMHDlSfn5+GX4UQ87kdiwSEhLk4uIie3t7tWjRQu+//76efvrpex2u1cvNeGzbtk2ffvqpFixYcD9CfGTkZizKly+vzz77TN98840+//xzpaWlqW7duvrrr7/uR8hWLTfj8eeff2rlypVKTU3VunXrNHbsWM2cOVNvvvnm/QjZat3tf8N//fVX/f777+rdu/e9CvGRYJffAQAAkJemTZum5cuXKyIigoda5RNXV1dFRUUpKSlJGzdu1JAhQ1S6dGmFhITkd2iPlEuXLumll17SggULVKRIkfwO55FXp04d1alTx7xdt25dBQUFad68eZo8eXI+RvZoSktLk7e3t+bPny9bW1vVrFlTf//9t9555x2NHz8+v8N7ZH366aeqXLmynnjiifwO5aFGkg/kkyJFisjW1lanTp2yKD916tRtH+RWtGjRHNVH9uRmLHDv3M14zJgxQ9OmTdNPP/2kKlWq3MswHwm5HQsbGxuVLVtWklStWjXFxMRo6tSpJPl3KafjERcXp2PHjqlVq1bmsrS0NEmSnZ2dYmNjVaZMmXsbtJXKi/9uFChQQNWrV9eRI0fuRYiPlNyMh6+vrwoUKCBbW1tzWVBQkP79919du3ZN9vb29zRma3U3/zYuX76s5cuXa9KkSfcyxEcCy/WBfGJvb6+aNWtq48aN5rK0tDRt3LjR4pf+W9WpU8eiviT9+OOPt62P7MnNWODeye14vP3225o8ebLWr1+vWrVq3Y9QrV5e/dtIS0tTSkrKvQjxkZLT8ahQoYL279+vqKgo8+fZZ59Vw4YNFRUVJX9///sZvlXJi38bqamp2r9/v3x9fe9VmI+M3IxHvXr1dOTIEfMPX5L0xx9/yNfXlwT/LtzNv40VK1YoJSVFL7744r0O0/rl95P/gEfZ8uXLDQcHByM8PNw4ePCg0bdvX8PDw8P4999/DcMwjJdeeskYNWqUuX5kZKRhZ2dnzJgxw4iJiTHGjx9vFChQwNi/f39+dcFq5HQsUlJSjL179xp79+41fH19jWHDhhl79+41Dh8+nF9dsCo5HY9p06YZ9vb2xsqVK434+Hjz59KlS/nVBauR07GYMmWK8cMPPxhxcXHGwYMHjRkzZhh2dnbGggUL8qsLViWn4/FfPF0/7+R0LCZOnGhs2LDBiIuLM/bs2WN06tTJcHR0NA4cOJBfXbAqOR2PEydOGK6urkb//v2N2NhY47vvvjO8vb2NN998M7+6YDVy+79TTz31lNGxY8f7Ha5VIskH8tn7779vlChRwrC3tzeeeOIJ45dffjHvCw7+f+3deUhU/R7H8c/Q4lg5Y2kOhWQ+OJpYlraa2NgfhYjRAqkhpa0QlEVRUGblFv1Rf7QQhEZaFBK0UFghIUFZorlQiS20UER7RmrQYuf5o9tw57bcfB4f9Z77fsHAcM7v/Pye80PkM98zR5eRnp7uMf7YsWNGaGio0bdvXyMiIsIoKyvr4orNqyNr8eDBA0PSdy+Xy9X1hZtUR9YjKCjoh+uxZcuWri/chDqyFllZWUZISIhhtVqNgQMHGjExMUZpaWk3VG1eHf278e8I+Z2rI2uxevVq91iHw2EkJiYadXV13VC1eXX0d+PKlSvGxIkTDS8vL+OPP/4wCgoKjM+fP3dx1ebU0bW4deuWIckoLy/v4krNyWIYhtFNNxEAAAAAAIBOxHfyAQAAAAAwCUI+AAAAAAAmQcgHAAAAAMAkCPkAAAAAAJgEIR8AAAAAAJMg5AMAAAAAYBKEfAAAAAAATIKQDwAAAACASRDyAQAAAAAwCUI+AADAv2RkZGjWrFndXcYPPXz4UBaLRQ0NDd1dCgCgByPkAwAA9HAfP37s7hIAAP8jCPkAAAA/EB8fr5UrV2r16tUaOHCgHA6HCgsL1dbWpoULF8rHx0chISE6d+6c+5iLFy/KYrGorKxMkZGRslqtmjRpkm7evOkx9/HjxxURESEvLy8NHz5cO3fu9Ng/fPhw5eXlacGCBbLZbFq2bJmCg4MlSVFRUbJYLIqPj5ck1dTUaNq0afL395fdbpfL5VJdXZ3HfBaLRUVFRZo9e7b69esnp9Op06dPe4xpbGxUUlKSbDabfHx8FBcXp3v37rn3FxUVKTw8XFarVSNGjNC+ffv+9jUGAHQ+Qj4AAMBPlJSUyN/fX9XV1Vq5cqWWL1+uuXPnavLkyaqrq9P06dM1f/58vX//3uO4devWaefOnaqpqdHgwYM1Y8YMffr0SZJUW1ur5ORkpaam6saNG9q6dauys7NVXFzsMceOHTs0evRo1dfXKzs7W9XV1ZKkCxcu6OnTpzpx4oQkqaWlRenp6bp8+bKqqqrkdDqVmJiolpYWj/lycnKUnJys69evKzExUWlpaXrz5o0k6cmTJ5oyZYq8vLxUUVGh2tpaLVq0SJ8/f5YkHTlyRJs3b1ZBQYGampq0bds2ZWdnq6SkpNOvOQDg77EYhmF0dxEAAAA9QUZGht6+fatTp04pPj5e7e3tunTpkiSpvb1ddrtdc+bM0aFDhyRJz54905AhQ3T16lVNmjRJFy9e1NSpU1VaWqqUlBRJ0ps3bxQYGKji4mIlJycrLS1NL1++VHl5ufvnrl+/XmVlZWpsbJT0tZMfFRWlkydPusc8fPhQwcHBqq+v15gxY356Dl++fJGvr6+OHj2qpKQkSV87+Zs2bVJeXp4kqa2tTQMGDNC5c+eUkJCgjRs3qrS0VLdv31afPn2+mzMkJER5eXmaN2+ee1t+fr7Onj2rK1eu/JVLDQD4h9DJBwAA+InIyEj3+169esnPz0+jRo1yb3M4HJKkFy9eeBwXExPjfj9o0CCFhYWpqalJktTU1KTY2FiP8bGxsbp7967a29vd28aNG/dbNT5//lxLly6V0+mU3W6XzWZTa2urHj169NNz6d+/v2w2m7vuhoYGxcXF/TDgt7W16d69e1q8eLEGDBjgfuXn53vczg8A6Bl6d3cBAAAAPdV/hl6LxeKxzWKxSPraPe9s/fv3/61x6enpev36tXbt2qWgoCB5eXkpJibmu4f1/ehcvtXt7e390/lbW1slSYWFhZo4caLHvl69ev1WjQCArkPIBwAA6GRVVVUaNmyYJKm5uVl37txReHi4JCk8PFyVlZUe4ysrKxUaGvrL0Ny3b19J8uj2fzt23759SkxMlCQ9fvxYr1696lC9kZGRKikp0adPn777MMDhcGjo0KG6f/++0tLSOjQvAKDrEfIBAAA6WW5urvz8/ORwOJSVlSV/f3/NmjVLkrR27VqNHz9eeXl5SklJ0dWrV7V3797/+rT6gIAAeXt76/z58woMDJTVapXdbpfT6dThw4c1btw4vXv3TuvWrftlZ/5HVqxYoT179ig1NVUbNmyQ3W5XVVWVJkyYoLCwMOXk5CgzM1N2u10JCQn68OGDrl27pubmZq1Zs+avXiYAwD+A7+QDAAB0su3bt2vVqlUaO3asnj17pjNnzrg78dHR0Tp27JhKS0s1cuRIbd68Wbm5ucrIyPjlnL1799bu3bu1f/9+DR06VDNnzpQkHThwQM3NzYqOjtb8+fOVmZmpgICADtXr5+eniooKtba2yuVyaezYsSosLHR39ZcsWaKioiIdPHhQo0aNksvlUnFxsfvf+gEAeg6erg8AANBJvj1dv7m5Wb6+vt1dDgDg/xCdfAAAAAAATIKQDwAAAACASXC7PgAAAAAAJkEnHwAAAAAAkyDkAwAAAABgEoR8AAAAAABMgpAPAAAAAIBJEPIBAAAAADAJQj4AAAAAACZByAcAAAAAwCQI+QAAAAAAmMSfrhxtiREA7wYAAAAASUVORK5CYII=\n"
          },
          "metadata": {}
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 🧠 Retrain or Extend the Model\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Retrain XGBoost or extend the pipeline for additional experiments or tuning.\n"
      ],
      "metadata": {
        "id": "O-G9MK11XMbX"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "import pandas as pd\n",
        "from sklearn.model_selection import train_test_split\n",
        "from sklearn.preprocessing import LabelEncoder\n",
        "import xgboost as xgb\n",
        "\n",
        "# Dataset load\n",
        "df = pd.read_csv(\"aml_transactions_full_100k.csv\")\n",
        "\n",
        "# Fill missing categorical values\n",
        "df['previous_receiver_country'].fillna('Unknown', inplace=True)\n",
        "\n",
        "# Features and target\n",
        "cols_to_drop = ['transaction_id','account_id','receiver_id','transaction_time','previous_transaction_time']\n",
        "X = df.drop(columns=cols_to_drop + ['is_fraud'])\n",
        "y = df['is_fraud']\n",
        "\n",
        "# Label Encoding for categorical features\n",
        "cat_cols = ['sender_country','receiver_country','channel','account_type','previous_receiver_country']\n",
        "for col in cat_cols:\n",
        "    X[col] = LabelEncoder().fit_transform(X[col])\n",
        "\n",
        "# Train-test split\n",
        "X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.2, stratify=y, random_state=42)\n"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "7GP7iiQ4T9vI",
        "outputId": "4e3cb0e0-27e6-4ec1-dd46-9e0ff52122d0"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stderr",
          "text": [
            "/tmp/ipython-input-1442677634.py:10: FutureWarning: A value is trying to be set on a copy of a DataFrame or Series through chained assignment using an inplace method.\n",
            "The behavior will change in pandas 3.0. This inplace method will never work because the intermediate object on which we are setting values always behaves as a copy.\n",
            "\n",
            "For example, when doing 'df[col].method(value, inplace=True)', try using 'df.method({col: value}, inplace=True)' or df[col] = df[col].method(value) instead, to perform the operation inplace on the original object.\n",
            "\n",
            "\n",
            "  df['previous_receiver_country'].fillna('Unknown', inplace=True)\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 💾 Train Final Model\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Train the final model version with refined parameters for production use.\n"
      ],
      "metadata": {
        "id": "iY26r-GJZcGC"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "# Train model\n",
        "model = xgb.XGBClassifier(n_estimators=200, max_depth=10, use_label_encoder=False, eval_metric='logloss')\n",
        "model.fit(X_train, y_train)\n",
        "\n",
        "# Test model accuracy\n",
        "y_pred = model.predict(X_test)\n",
        "accuracy = (y_pred == y_test).mean()\n",
        "print(f\"✅ Model trained. Test Accuracy: {accuracy:.2f}\")"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "gpR9Wlu9UC9s",
        "outputId": "1513b02d-0474-4f54-e2a6-b899e259b490"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stderr",
          "text": [
            "/usr/local/lib/python3.12/dist-packages/xgboost/training.py:183: UserWarning: [13:33:40] WARNING: /workspace/src/learner.cc:738: \n",
            "Parameters: { \"use_label_encoder\" } are not used.\n",
            "\n",
            "  bst.update(dtrain, iteration=i, fobj=obj)\n"
          ]
        },
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "✅ Model trained. Test Accuracy: 0.93\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 📦 Save the Trained Model\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Save the trained XGBoost model using `joblib` for reuse in Streamlit deployment.\n"
      ],
      "metadata": {
        "id": "xgE6SKYwZnCp"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "import joblib\n",
        "\n",
        "# Save trained model\n",
        "joblib.dump(model, \"xgb_model.pkl\")\n",
        "print(\"✅ Model saved as xgb_model.pkl\")"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "lMVhY5-UUG-g",
        "outputId": "61359444-7fa9-408e-bab3-03e11e1d5550"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "✅ Model saved as xgb_model.pkl\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "##🌐 Install Streamlit and Pyngrok\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Install necessary tools for creating and sharing a live web app in Google Colab.\n"
      ],
      "metadata": {
        "id": "rNf2y8hSZqE8"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "!pip install streamlit pyngrok --quiet"
      ],
      "metadata": {
        "id": "yGMGZxzQUKNr"
      },
      "execution_count": null,
      "outputs": []
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 🔑 Set Ngrok Authentication\n",
        "Authenticate Ngrok using your token to allow tunneling of Streamlit web apps.\n"
      ],
      "metadata": {
        "id": "rprfeWuwZy0I"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "from pyngrok import ngrok\n",
        "\n",
        "# Set your ngrok auth token\n",
        "ngrok.set_auth_token(\"2pZDIIAf2OdOXIWnidHUELiHb0p_7eETgLTrU9gLPr7mZiSRX\")"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "Nm_qReOwUO4T",
        "outputId": "a794686a-9804-4e86-bf28-85b48361e0a3"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": []
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 🧩 Import Dependencies for Streamlit App\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Load essential modules and helper functions required by the deployed app.\n"
      ],
      "metadata": {
        "id": "b7ItfqRJZ3Fv"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "import pandas as pd\n",
        "import numpy as np\n",
        "import joblib\n",
        "from sklearn.ensemble import RandomForestClassifier\n",
        "from sklearn.preprocessing import LabelEncoder\n",
        "\n",
        "# LabelEncoder\n",
        "le_sender = LabelEncoder()\n",
        "le_sender.fit(['India', 'USA', 'UK', 'Singapore', 'UAE', 'Germany', 'Nigeria', 'China'])\n",
        "\n",
        "# Dummy training data\n",
        "X_train = pd.DataFrame({\n",
        "    'amount': [1000, 500000, 75000, 250000, 900000],\n",
        "    'sender_country': le_sender.transform(['India', 'USA', 'UK', 'Singapore', 'UAE'])\n",
        "})\n",
        "y_train = [0,1,0,1,1]  # Example labels\n",
        "\n",
        "# Random Forest train\n",
        "rf_model = RandomForestClassifier(n_estimators=200, max_depth=5, class_weight='balanced', random_state=42)\n",
        "rf_model.fit(X_train, y_train)\n",
        "\n",
        "# Save model to current directory\n",
        "joblib.dump(rf_model, \"rf_two_feature_model.pkl\")\n",
        "print(\"✅ Model saved as rf_two_feature_model.pkl\")"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "MBZPKY1GUf_f",
        "outputId": "d5ac1f91-c07d-4a8b-a7e4-f8d4e60d5b0d"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "✅ Model saved as rf_two_feature_model.pkl\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 🖥️ Build Streamlit Application\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Create a Streamlit UI (`app.py`) that allows users to input transaction data and predict if it’s suspicious.\n"
      ],
      "metadata": {
        "id": "rm-MBJ1PaA-g"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "%%writefile app.py\n",
        "import streamlit as st\n",
        "import pandas as pd\n",
        "import joblib\n",
        "from sklearn.preprocessing import LabelEncoder\n",
        "\n",
        "st.title(\"💰 AML Fraud Prediction (Manual Input)\")\n",
        "\n",
        "# Load trained model\n",
        "model = joblib.load(\"xgb_model.pkl\")\n",
        "\n",
        "# User Input Form\n",
        "amount = st.number_input(\"Transaction Amount\", min_value=1.0, max_value=1000000.0, value=1000.0)\n",
        "sender_country = st.selectbox(\"Sender Country\", ['India','USA','UK','Singapore','UAE','Germany','Nigeria','China'])\n",
        "receiver_country = st.selectbox(\"Receiver Country\", ['India','USA','UK','Singapore','UAE','Germany','Nigeria','China'])\n",
        "channel = st.selectbox(\"Transaction Channel\", ['Online', 'ATM', 'Bank Transfer', 'Mobile App'])\n",
        "account_type = st.selectbox(\"Account Type\", ['Savings','Current','Business','Offshore'])\n",
        "previous_receiver_country = st.selectbox(\"Previous Receiver Country\", ['India','USA','UK','Singapore','UAE','Germany','Nigeria','China','Unknown'])\n",
        "time_diff_minutes = st.number_input(\"Time Diff Since Last Transaction (minutes)\", min_value=0, max_value=100000, value=60)\n",
        "\n",
        "# Prepare input for model (column order same as training)\n",
        "user_input = pd.DataFrame({\n",
        "    'amount':[amount],\n",
        "    'sender_country':[sender_country],\n",
        "    'receiver_country':[receiver_country],\n",
        "    'channel':[channel],\n",
        "    'account_type':[account_type],\n",
        "    'is_weekend':[0],\n",
        "    'high_value':[1 if amount>500000 else 0],\n",
        "    'international':[1 if sender_country != receiver_country else 0],\n",
        "    'time_diff_minutes':[time_diff_minutes],\n",
        "    'previous_receiver_country':[previous_receiver_country],\n",
        "    'country_changed':[1 if previous_receiver_country != receiver_country else 0],\n",
        "    'same_account_multiple_txn':[1 if time_diff_minutes <=120 else 0]\n",
        "})\n",
        "\n",
        "# Label Encoding\n",
        "cat_cols = ['sender_country','receiver_country','channel','account_type','previous_receiver_country']\n",
        "for col in cat_cols:\n",
        "    le = LabelEncoder()\n",
        "    user_input[col] = le.fit_transform(user_input[col])\n",
        "\n",
        "# Predict button\n",
        "if st.button(\"Predict Fraud\"):\n",
        "    pred = model.predict(user_input)[0]\n",
        "    prob = model.predict_proba(user_input)[0][1]\n",
        "    if pred==1:\n",
        "        st.error(f\"⚠️ ALERT: This transaction is FRAUD! (Probability: {prob:.2f})\")\n",
        "    else:\n",
        "        st.success(f\"✅ This transaction seems safe. (Probability of Fraud: {prob:.2f})\")\n"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "tipFxA_GbKzz",
        "outputId": "7c6ffc2c-9f05-4c44-ec1b-5a2a018c52ff"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Writing app.py\n"
          ]
        }
      ]
    },
    {
      "cell_type": "markdown",
      "source": [
        "## 🚀 Launch Streamlit App via Ngrok\n",
        "\n",
        "---\n",
        "\n",
        "\n",
        "Run the Streamlit server in Colab and expose it using Ngrok for public access.\n"
      ],
      "metadata": {
        "id": "TjbnvalGaE0h"
      }
    },
    {
      "cell_type": "code",
      "source": [
        "!pip install pyngrok\n",
        "from pyngrok import ngrok\n",
        "\n",
        "!ngrok authtoken 2pZDIIAf2OdOXIWnidHUELiHb0p_7eETgLTrU9gLPr7mZiSRX\n",
        "get_ipython().system_raw('streamlit run app.py &')\n",
        "url = ngrok.connect(8501)\n",
        "print(\"🌐 Open your Streamlit dashboard here:\", url)\n"
      ],
      "metadata": {
        "colab": {
          "base_uri": "https://localhost:8080/"
        },
        "id": "7vhLAuW2bMA4",
        "outputId": "cd0bd8a6-a00d-4528-ed14-107520da2fca"
      },
      "execution_count": null,
      "outputs": [
        {
          "output_type": "stream",
          "name": "stdout",
          "text": [
            "Requirement already satisfied: pyngrok in /usr/local/lib/python3.12/dist-packages (7.4.0)\n",
            "Requirement already satisfied: PyYAML>=5.1 in /usr/local/lib/python3.12/dist-packages (from pyngrok) (6.0.3)\n",
            "Authtoken saved to configuration file: /root/.config/ngrok/ngrok.yml\n",
            "🌐 Open your Streamlit dashboard here: NgrokTunnel: \"https://6e59a7a3941d.ngrok-free.app\" -> \"http://localhost:8501\"\n"
          ]
        }
      ]
    }
  ]
}

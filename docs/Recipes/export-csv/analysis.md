---
title: "Analysis"
slug: "export-csv/analysis"
excerpt: "Analyze CSV data with pandas."
hidden: false
---

## Load and Analyze

```python
import pandas as pd

# Load data
df = pd.read_csv("logs/aircraft_2024-01-15.csv")

# Basic statistics
print(f"Total sightings: {len(df)}")
print(f"Unique aircraft: {df['hex'].nunique()}")
print(f"Military: {df['military'].sum()}")
print(f"Emergencies: {df['emergency'].sum()}")

# Most common aircraft types
print("\nTop Aircraft Types:")
print(df['type'].value_counts().head(10))

# Distance distribution
print(f"\nDistance Stats:")
print(df['distance'].describe())

# Altitude distribution
print(f"\nAltitude Stats:")
print(df['alt'].describe())
```

## Visualization

```python
import matplotlib.pyplot as plt

# Altitude histogram
df['alt'].hist(bins=50)
plt.xlabel('Altitude (ft)')
plt.ylabel('Count')
plt.title('Altitude Distribution')
plt.savefig('altitude_dist.png')

# Aircraft by hour
df['timestamp'] = pd.to_datetime(df['timestamp'])
df['hour'] = df['timestamp'].dt.hour
df.groupby('hour')['hex'].count().plot(kind='bar')
plt.xlabel('Hour')
plt.ylabel('Sightings')
plt.title('Traffic by Hour')
plt.savefig('traffic_by_hour.png')
```

## Filter Military

```python
military_df = df[df['military'] == True]
print(f"Military aircraft: {len(military_df)}")
print(military_df[['timestamp', 'hex', 'flight', 'type', 'alt']])
```

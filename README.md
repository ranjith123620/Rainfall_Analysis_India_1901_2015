# Rainfall_Analysis_India_1901_2015
# rainfall_analysis_india_fixed.py
import pandas as pd
import matplotlib.pyplot as plt
import numpy as np
import os

#  Preprocessing 
def preprocess_rainfall(input_file, output_file):
    df = pd.read_csv(input_file)

    # Print initial info
    print("Initial columns:", df.columns.tolist())

    # Normalize column names (lowercase, no spaces)
    df.columns = df.columns.str.strip().str.lower()

    # Check for 'year' column
    if "year" not in df.columns:
        # Try to infer the first column as Year
        df.rename(columns={df.columns[0]: "year"}, inplace=True)

    # Fill missing numeric values
    df.fillna(df.mean(numeric_only=True), inplace=True)

    # Save cleaned data
    df.to_csv(output_file, index=False)
    print(f"✅ Processed file saved as {output_file}")
    print("Shape after processing:", df.shape)

#  Visualization 
def visualize_rainfall(processed_file, output_dir="plots"):
    os.makedirs(output_dir, exist_ok=True)
    df = pd.read_csv(processed_file)

    # Identify month columns automatically
    month_cols = [
        col for col in df.columns
        if col not in ["year", "annual", "subdivision", "region"] and df[col].dtype != 'O'
    ]

    # Melt into long format
    rainfall_long = df.melt(
        id_vars=["year"],
        value_vars=month_cols,
        var_name="month",
        value_name="rainfall_mm"
    )

    # Fill missing rainfall
    rainfall_long["rainfall_mm"].fillna(0, inplace=True)

    # Order months if available
    month_order = ["jan","feb","mar","apr","may","jun","jul","aug","sep","oct","nov","dec"]
    rainfall_long["month"] = rainfall_long["month"].str.lower()
    rainfall_long["month"] = pd.Categorical(rainfall_long["month"], categories=month_order, ordered=True)

    #  1. Total Rainfall by Year 
    yearly = rainfall_long.groupby("year")["rainfall_mm"].sum()
    plt.figure(figsize=(10,5))
    plt.bar(yearly.index, yearly.values, color="skyblue")
    plt.title("Total Rainfall in India (1901–2015)")
    plt.xlabel("Year")
    plt.ylabel("Rainfall (mm)")
    plt.grid(axis="y", linestyle="--", alpha=0.6)
    plt.tight_layout()
    plt.savefig(f"{output_dir}/total_rainfall_by_year.png")
    plt.show()

    #  2. Monthly Average Rainfall
    monthly_avg = rainfall_long.groupby("month")["rainfall_mm"].mean()
    plt.figure(figsize=(8,5))
    plt.bar(monthly_avg.index, monthly_avg.values, color="orange")
    plt.title("Average Monthly Rainfall (1901–2015)")
    plt.xlabel("Month")
    plt.ylabel("Average Rainfall (mm)")
    plt.grid(axis="y", linestyle="--", alpha=0.6)
    plt.tight_layout()
    plt.savefig(f"{output_dir}/average_monthly_rainfall.png")
    plt.show()

    # 3. Line Chart: Yearly Trend
    plt.figure(figsize=(10,5))
    plt.plot(yearly.index, yearly.values, marker="o", color="green", linewidth=2)
    plt.title("Rainfall Trend Over the Years (1901–2015)")
    plt.xlabel("Year")
    plt.ylabel("Rainfall (mm)")
    plt.grid(True, linestyle="--", alpha=0.6)
    plt.tight_layout()
    plt.savefig(f"{output_dir}/rainfall_trend.png")
    plt.show()



#  Main 
if __name__ == "__main__":
    input_path = "rainfall in india 1901-2015.csv"
    processed_path = "processed_rainfall.csv"

    preprocess_rainfall(input_path, processed_path)
    visualize_rainfall(processed_path)

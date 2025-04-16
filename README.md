# Python-Sales-Monitoring-Data
# Laporan Proyek Machine Learning - Akmal Muhammad Naim

## Domain Proyek

Proyek ini dilakukan oleh permintaan klien saya yang dimana klien ingin membuat sebuah data master sales monitoring yang diambil dari basis data penjualan mereka yang terkumpul dari tahun 2019

### Mengapa masalah ini harus diselesaikan:

- Klien ingin melihat behavior dari pembeli mereka jadi klien meminta agar data penjualan diubah menjadi data master monitoring yang sesuai format keinginan mereka
- data yang terkumpul cukup signifikan yaitu diatas 100k baris jadi untuk mengolahnya tidak memungkinkan menggunakan aplikasi excel dibutuhkan penggunaan bahasa pemrograman python terutama pada tools panda


## Business Understanding

### Problem Statements

1. Bagaimana cara mentransformasikan data penjualan yang tidak terstandar menjadi standar?
2. Bagaimana data yang telah diolah diubah menjadi data master sales monitoring sesuai dengan format keinginan klien?


### Goals

1. Membuat data master sales monitoring 


### Solution Statements

- Menggunakan tools library pandas pada platform jupyter notebook dengan bahasa pemrograman python

## Data Understanding

Dataset yang digunakan dalam proyek ini berasal dari file dump excel dari website jurnal.id, yang bernama sales_by_customer_01-01-2019_31-12-2019.xlsx,sales_by_customer_01-01-2020_30-11-2024.xlsx.

- **Jumlah Baris Data**: 125813 baris
- **Kondisi Data**: dapat dilihat gambar dibawah ini 

Dataset tidak standar pada umumnya karena itu adalah hasil export data dari jurnal.id



### Kolom/Fitur Data:
Dari data yang sudah ada gambaran kolom fitur data yang dibersihkan dan di standarkan nanti akan diolah adalah sebagai berikut :

1. **Transaction**:
   - Kolom ini berisi label nama transaksi yaitu Sales Invoice 
     
2. **No**:
   - Kolom ini berisi nomor invoice yang nantinya akan digunakan untuk mentotalkan harga/amount pada data sales monitoring
  
3. **Product**:
   - Kolom ini berisi teks nama product yang telah dibeli oleh customer.

4. **Qty**:
   - Kolom ini berisi jumlah kuantitas barang.
  
5. **Unit**:
   - Kolom ini berisih besaran pada barang  

6. **Unit Price**:
   - Kolom ini berisi harga per unit
     
7. **Amount**:
   - Kolom ini merupakan hasil dari Qty * Unit Price
     
8. **Total**:
   - Kolom ini berisi total penjumlahan amount berdasarkan kolom No
     
9. **Customer**:
   - Merupakan Nama Customer

10. **Date**:
   - Merupakan Tanggal Transaksi 


Berikut kondisi awal data mentah : 
![Image](https://github.com/user-attachments/assets/842d69cd-3e06-4767-8a4d-91c884f3de66)


**_Data Combining_**

Menggabungkan beberapa sumber data menjadi satu dataframe 

Berikut ini adalah Data Combining yang dilakukan :

- ```python
  import pandas as pd

  # === Configuration ===
  FILES = [
    "sales_by_customer_01-01-2019_31-12-2019.xlsx",
    "sales_by_customer_01-01-2020_30-11-2024.xlsx"
  ]
  HEADER_ROW_INDEX = 5  # Header is located on the 6th row (index 5)
  SKIP_ROWS = 6         # Number of rows to skip for files after the first

  # === Load and Combine Data ===
  dataframes = []

  # Load the first Excel file with headers
  df_first = pd.read_excel(FILES[0], header=HEADER_ROW_INDEX)
  dataframes.append(df_first)

  # Load the rest of the files without headers and apply the same columns
  for file in FILES[1:]:
      df = pd.read_excel(file, skiprows=SKIP_ROWS, header=None)
      df.columns = df_first.columns  # Use header from the first file
      dataframes.append(df)
  
  # Concatenate all DataFrames into one
  combined_df = pd.concat(dataframes, ignore_index=True)
  ```

  Kode diatas memiliki output:
  
![Image](https://github.com/user-attachments/assets/770cf1f3-fb62-4181-943c-7fa7a9201e39)

terlihat bahwa data telah digabungkan namun terlihat bahwa data tidak bersih. maka dilakukan proses berikutnya

**_Data Cleaning and Transforming_**

Membersihkan kolom-kolom dan baris-baris yang tidak digunakan dan mengubah data sesuai dengan format standar 

Berikut ini adalah Data Cleaning and Transforming yang dilakukan :

- ```python
  

  # === Clean and Transform Data ===
  
  # Mark rows that contain customer names instead of dates
  combined_df['IsCustomer'] = ~combined_df['Customer / Date'].str.contains(r'\d{2}/\d{2}/\d{4}', na=False)
  
  # Fill down customer names
  combined_df['Customer'] = combined_df['Customer / Date'].where(combined_df['IsCustomer']).ffill()
  
  # Extract dates into a new column
  combined_df['Date'] = combined_df['Customer / Date'].where(~combined_df['IsCustomer'])
  
  # Drop unnecessary columns
  combined_df.drop(columns=['Customer / Date', 'IsCustomer','Description'], inplace=True)
  
  # Remove rows where all values are NaN
  combined_df.dropna(how='all', inplace=True)
  
  # Remove rows where 'Transaction' column is NaN
  combined_df.dropna(subset=['Transaction'], inplace=True)
  
  # === Output Result ===
  
  # Display the cleaned DataFrame
  combined_df
  
  # Optional: Save to Excel
  combined_df.to_excel("cleaned_sales_data.xlsx", index=False)

  ```

  Kode diatas memiliki output:
  
![Image](https://github.com/user-attachments/assets/e318bc4e-7c7a-4547-a213-b4c2b27e84ed)

Data sudah Bersih kemudian proses berikutnya adalah transformasi menjadi sales monitoring
  
**Data Sales Monitoring Transform**

Klien menginginkan susunan kolom-kolom sebagai berikut :

1. No :
   - angka incremental dimulai dari 1
  
2. transaction_[year]
   - merupakan kolom jumlah pada transaksi pada tahun tertentu
   - klien menginginkan jika ada pergantian tahun maka kolom otomatis bertambah

3. amount_[year]
   - merupakan kolom total amount pada tiap tahun
  
4. Contact:
   - merupakan Nomor Pribadi pembeli
  
5. total_transaction
   - merupakan total transaksi dari awal tahun hingga tahun saat ini
  
6. year_count
   - merupakan sebarapa banyak tahun customer aktif
  
7. total_amount
   - merupakan jumlah harga total pada seluruh transaksi yang dilakukan oleh pembeli
  
8. average_amount_per_trx
   - jumlah rata-ata harga pada tiap transaksi
  
9. last_trx
   - tanggal terakhir transaksi yang dilakukan oleh pembeli
   
Berikut ini adalah Data Cleaning and Transforming yang dilakukan :

- ```python
  # === Ensure 'Date' column is in datetime format ===
  df['Date'] = pd.to_datetime(df['Date'], dayfirst=True, errors='coerce')
  
  
  # Drop rows with invalid dates if any
  df.dropna(subset=['Date'], inplace=True)
  
  # Create 'Formatted_Date' column (optional, just a cleaned datetime)
  df['Formatted_Date'] = df['Date']
  
  # Extract year for grouping
  df['Year'] = df['Formatted_Date'].dt.year
  
  # Calculate the last transaction date per customer
  df['last_transaction'] = df.groupby('Customer')['Formatted_Date'].transform('max')
  
  # === Aggregate by Customer and Year ===
  grouped = df.groupby(['Customer', 'Year']).agg(
      transaction_count=('No', 'nunique'),
      total_amount=('Amount', 'sum')
  ).reset_index()
  
  # === Pivot to Wide Format ===
  pivot_transaction = grouped.pivot(index='Customer', columns='Year', values='transaction_count').fillna(0).astype(int)
  pivot_amount = grouped.pivot(index='Customer', columns='Year', values='total_amount').fillna(0).astype(int)
  
  # Rename columns
  pivot_transaction.columns = [f"transaction_{year}" for year in pivot_transaction.columns]
  pivot_amount.columns = [f"amount_{year}" for year in pivot_amount.columns]
  
  # Combine transaction & amount pivots
  result = pd.concat([pivot_transaction, pivot_amount], axis=1)
  
  # === Final Summary Columns ===
  result['total_transaction'] = result.filter(like='transaction_').sum(axis=1)
  result['total_amount'] = result.filter(like='amount_').sum(axis=1)
  result['last_trx'] = df.groupby('Customer')['last_transaction'].first()
  result['Contact'] = ''
  
  # Count how many years the customer was active
  year_columns = [col for col in result.columns if col.startswith('transaction_')]
  result['year_count'] = result[year_columns].gt(0).sum(axis=1)
  
  # Calculate average amount per transaction
  result['average_amount_per_trx'] = (result['total_amount'] / result['total_transaction']).fillna(0).astype(int)
  
  result.reset_index(inplace=True)
  
  
  # === Reorder Columns ===
  ordered_cols = (
      ['Customer'] +
      year_columns +
      [col.replace('transaction_', 'amount_') for col in year_columns] +
      ['Contact', 'total_transaction', 'year_count', 'total_amount', 'average_amount_per_trx', 'last_trx']
  )
  result = result[ordered_cols]
  result
  ```

  Kode diatas memiliki output:
  
  ![Image](https://github.com/user-attachments/assets/b5024ff3-b6e8-49bc-a1b2-b271e3768efb)
  ![Image](https://github.com/user-attachments/assets/2e0249d8-6c23-4a22-8cd4-c641a5814aaf)
  ![image](https://github.com/user-attachments/assets/36c14cc6-74ce-4e12-b2c5-ac73743a248a)


**Data Contact Merge**
  Klien menginginkna data monitoring ini ditambahkan contact yang sesuai dengan nama customer 


  

  Berikut ini adalah Data contact merge yang dilakukan :
  - ```python
     import pandas as pd
    
    # === Load contact file ===
    contact_df = pd.read_excel("CARA_FLORIST_ContactExport_12_04_2025.xlsx")
    
    # Select only the necessary columns and rename them for consistency
    contact_df = contact_df[["*DisplayName", "Mobile", "Phone"]].copy()  # Select relevant columns
    contact_df.rename(columns={"*DisplayName": "ContactName"}, inplace=True)  # Rename column for consistency
    
    # Function to combine Mobile and Phone into a single 'Contact' column
    def combine_contacts(row):
        # Convert Mobile and Phone to strings if not null
        mobile = str(row['Mobile']) if pd.notna(row['Mobile']) else ''
        phone = str(row['Phone']) if pd.notna(row['Phone']) else ''
        
        # If both Mobile and Phone exist, combine them with a '/' separator
        if mobile and phone:
            return f"{mobile}/{phone}"
        # If only Mobile exists, return Mobile
        elif mobile:
            return mobile
        # If only Phone exists, return Phone
        elif phone:
            return phone
        # If neither exist, return an empty string
        else:
            return ''
    
    # Apply the function to create a new 'Contact' column
    contact_df['Contact'] = contact_df.apply(combine_contacts, axis=1)
    
    # Drop the Mobile and Phone columns as they are no longer needed
    contact_df.drop(columns=['Mobile', 'Phone'], inplace=True)

    # Perform a merge to get the 'Contact' column from contact_df
    merged_contact = pd.merge(
        result[['Customer']],  # Only take the 'Customer' column to avoid disrupting the structure
        contact_df,            # 'contact_df' already contains the 'Contact' column with combined mobile/phone
        left_on="Customer",    # Merge on the 'Customer' column from 'result'
        right_on="ContactName",  # Merge on the 'ContactName' column from 'contact_df'
        how="left"             # 'left' join ensures all rows from 'result' are kept
    )
    
    # Select only the 'Customer' and 'Contact' columns
    merged_contact = merged_contact[['Customer', 'Contact']]
    
    # Remove the existing 'Contact' column in the 'result' DataFrame to avoid duplication
    result.drop(columns=['Contact'], inplace=True)
    
    # Merge the updated 'merged_contact' back into 'result', overwriting the previous 'Contact' column
    result = pd.merge(result, merged_contact, on="Customer", how="left")
    
    # Define the desired column order
    ordered_cols = (
        ['Customer'] +  # Start with the 'Customer' column
        [col for col in result.columns if col.startswith('transaction_')] +  # Include all columns starting with 'transaction_'
        [col for col in result.columns if col.startswith('amount_')] +  # Include all columns starting with 'qty_'
        ['Contact'] +  # Place the 'Contact' column after the 'amount_2024' column
        ['total_transaction', 'year_count', 'total_amount', 'average_amount_per_trx', 'last_trx']  # Add summary columns at the end
    )
    
    # Reorder the DataFrame based on the defined column order
    df = result[ordered_cols]
    
    # === Add 'No' Column ===
    df['No'] = range(1, len(df) + 1)  # Sequential numbering starting from 1
    
    # Reorder the columns to place 'No' on the left
    ordered_columns = ['No'] + [col for col in df.columns if col != 'No']
    df = df[ordered_columns]

    ```

berikut outputnya :

![image](https://github.com/user-attachments/assets/4704e96a-5921-49d9-9e65-310d81465902)
![image](https://github.com/user-attachments/assets/c0a7d634-bd6b-46d9-8c43-d579b27f55f5)


## Kesimpulan

Proyek ini menunjukkan pentingnya standarisasi dan membersihkan data agar proses transformasi mudah dilakukan, jika ada update dari klien maka klien hanya perlu menjalankan kembali code ini dengan sedikt penambahan yaitu pengisian nama file baru pada list di awal source code.

---



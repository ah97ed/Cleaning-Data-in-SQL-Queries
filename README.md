# Nashville Housing Data Cleaning Project
This project involves cleaning and transforming the Nashville Housing dataset using SQL. The goal is to standardize the dataset for analysis, improve data quality, and ensure consistency across all records.

## Project Overview
The Nashville Housing Data Cleaning project is designed to clean and transform raw housing data into a more structured format suitable for analysis. The dataset includes various columns related to property addresses, sales information, and owner details. The project involves the following key tasks:

- Standardizing date formats
- Populating missing property address data
- Breaking out addresses into separate columns
- Normalizing categorical fields
- Removing duplicate records
- Deleting unused columns

## Data Source
The dataset used in this project is housed in the PortfolioProject.dbo.NashvilleHousing table. The data includes information such as property addresses, sale dates, sale prices, owner addresses, and other related fields.

## SQL Code Details

### 1. Standardize Date Format
Convert the SaleDate column to a standard date format.

``` sql
ALTER TABLE NashvilleHousing
ADD SaleDateConverted Date;

UPDATE NashvilleHousing
SET SaleDateConverted = CONVERT(Date, SaleDate);

```

### 2. Populate Property Address Data
Fill in missing property addresses using existing data from the same ParcelID.

``` sql
UPDATE a
SET PropertyAddress = ISNULL(a.PropertyAddress, b.PropertyAddress)
FROM PortfolioProject.dbo.NashvilleHousing a
JOIN PortfolioProject.dbo.NashvilleHousing b
ON a.ParcelID = b.ParcelID
AND a.[UniqueID] <> b.[UniqueID]
WHERE a.PropertyAddress IS NULL;
```
### 3. Break Out Address into Individual Columns
Separate the PropertyAddress into Address and City.
```sql
ALTER TABLE NashvilleHousing
ADD PropertySplitAddress NVARCHAR(255),
    PropertySplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',', PropertyAddress) - 1),
    PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',', PropertyAddress) + 1, LEN(PropertyAddress));
```
### 4. Normalize Categorical Fields
Change 'Y' and 'N' in the SoldAsVacant field to 'Yes' and 'No'.
```sql
UPDATE NashvilleHousing
SET SoldAsVacant = CASE 
    WHEN SoldAsVacant = 'Y' THEN 'Yes'
    WHEN SoldAsVacant = 'N' THEN 'No'
    ELSE SoldAsVacant
END;
```
### 5. Remove Duplicates
Remove duplicate records based on specific columns.
``` sql
WITH RowNumCTE AS (
    SELECT *,
           ROW_NUMBER() OVER (
               PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
               ORDER BY UniqueID
           ) row_num
    FROM PortfolioProject.dbo.NashvilleHousing
)
DELETE FROM RowNumCTE
WHERE row_num > 1;
```
### 6. Delete Unused Columns
Remove columns that are no longer needed.
``` sql
ALTER TABLE PortfolioProject.dbo.NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate;
```
## Usage Instructions
To execute the data cleaning process:

1. Ensure the database connection: Make sure you have access to the PortfolioProject database and the NashvilleHousing table.
2. Run the SQL scripts: Execute the SQL commands in the order provided to perform the cleaning tasks.
3. Verify the results: After executing the scripts, check the table to ensure that data is clean and structured as expected.

## Conclusion
This project demonstrates the application of SQL techniques to clean and transform a housing dataset, making it ready for analysis. By standardizing formats, normalizing data, and removing duplicates, the dataset is now more consistent and reliable for any analytical tasks.







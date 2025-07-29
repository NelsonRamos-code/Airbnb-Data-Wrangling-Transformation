<h1>Airbnb Data Wrangling & Transformation Project</h1>



<h2>Description</h2>
This project focuses on cleaning and transforming real-world Airbnb listing data for Nashville, TN. The dataset contains property details, locations, pricing, and other key metrics—but like most scraped data, it had missing values, inconsistencies, duplicates, and formatting errors. Some objectives were: Standardization of data formats (dates, currencies, categorical values), correction of missing & invalid entries (imputation, corrections), splitting combined columns (e.g., address into city/state/ZIP) and remove duplicates & unused columns for preparing data for analysis.
<br />


<h2>Languages and Utilities Used</h2>

- <b>SQL</b> 

<h2>Environments Used </h2>

- <b>SQL Server</b> 

<h2>Program walk-through:</h2>

```sql
SELECT *
FROM DataCleaningProject.dbo.NashvilleHousing

-------------------------------------------------------------------------------------------

--Standarize Date Format

SELECT SaleDateConverted, CAST(SaleDate AS DATE) AS date1
FROM DataCleaningProject.dbo.NashvilleHousing

UPDATE NashvilleHousing
SET SaleDate = CAST(SaleDate AS DATE)

ALTER TABLE NashvilleHousing
ADD SaleDateConverted Date;

UPDATE NashvilleHousing
SET SaleDateConverted = CAST(SaleDate AS DATE)

--Populate Property Address Data

SELECT *
FROM DataCleaningProject.dbo.NashvilleHousing
ORDER BY ParcelID

SELECT a.ParcelID, a.PropertyAddress, b.ParcelID, b.PropertyAddress, COALESCE(a.PropertyAddress, b.PropertyAddress)
FROM DataCleaningProject.dbo.NashvilleHousing a
JOIN DataCleaningProject.dbo.NashvilleHousing b
	on a.ParcelID = b.ParcelID
	AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress IS NULL

UPDATE a
SET PropertyAddress = COALESCE(a.PropertyAddress, b.PropertyAddress)
FROM DataCleaningProject.dbo.NashvilleHousing a
JOIN DataCleaningProject.dbo.NashvilleHousing b
	on a.ParcelID = b.ParcelID
	AND a.UniqueID <> b.UniqueID
WHERE a.PropertyAddress IS NULL

--Breaking out Address Info Individual Columns (Address, City, State)

SELECT PropertyAddress
FROM DataCleaningProject.dbo.NashvilleHousing 

SELECT 
	SUBSTRING(PropertyAddress, 1, CHARINDEX(',',PropertyAddress)-1) AS Address,
	SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1, LEN(PropertyAddress)) AS Address
FROM DataCleaningProject.dbo.NashvilleHousing 

ALTER TABLE NashvilleHousing
ADD PropertySplitAddress NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitAddress = SUBSTRING(PropertyAddress, 1, CHARINDEX(',',PropertyAddress)-1)

ALTER TABLE NashvilleHousing
ADD PropertySplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET PropertySplitCity = SUBSTRING(PropertyAddress, CHARINDEX(',',PropertyAddress)+1, LEN(PropertyAddress))

select *
FROM DataCleaningProject.dbo.NashvilleHousing 


select OwnerAddress
FROM DataCleaningProject.dbo.NashvilleHousing 


SELECT 
	PARSENAME(REPLACE(OwnerAddress, ',','.'), 1),
	PARSENAME(REPLACE(OwnerAddress, ',','.'), 2),
	PARSENAME(REPLACE(OwnerAddress, ',','.'), 3)
FROM DataCleaningProject.dbo.NashvilleHousing 

ALTER TABLE NashvilleHousing
ADD OwnerSplitAddress NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitAddress = PARSENAME(REPLACE(OwnerAddress, ',','.'), 3)

ALTER TABLE NashvilleHousing
ADD OwnerSplitCity NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitCity = PARSENAME(REPLACE(OwnerAddress, ',','.'), 2)

ALTER TABLE NashvilleHousing
ADD OwnerSplitState NVARCHAR(255);

UPDATE NashvilleHousing
SET OwnerSplitState = PARSENAME(REPLACE(OwnerAddress, ',','.'), 1)


-- Change Y and N to Yes and No in ''Solid as Vacant'' field

SELECT COUNT(SoldAsVacant), SoldAsVacant
FROM DataCleaningProject.dbo.NashvilleHousing
GROUP BY SoldAsVacant


SELECT SoldAsVacant,
	CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
		 WHEN SoldAsVacant = 'N' THEN 'No'
		 ELSE SoldAsVacant
		 END
FROM DataCleaningProject.dbo.NashvilleHousing

UPDATE NashvilleHousing
SET SoldAsVacant = CASE WHEN SoldAsVacant = 'Y' THEN 'Yes'
		 WHEN SoldAsVacant = 'N' THEN 'No'
		 ELSE SoldAsVacant
		 END

SELECT SoldAsVacant, COUNT(SoldAsVacant)
FROM DataCleaningProject.dbo.NashvilleHousing
GROUP BY SoldAsVacant

-- Remove Duplicates

WITH RowNumCTE AS (
SELECT *, 
	   ROW_NUMBER() OVER(PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference ORDER BY UniqueID) AS row_num
FROM DataCleaningProject.dbo.NashvilleHousing
)

SELECT *
FROM RowNumCTE
WHERE row_num > 1


--ORDER BY PropertyAddress



-- Delete Unused Columns


SELECT *
FROM DataCleaningProject.dbo.NashvilleHousing

ALTER TABLE DataCleaningProject.dbo.NashvilleHousing
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress


```

<h2>📊 Results</h2>

- <b>Improved data consistency (e.g., standardized pricing, addresses).</b> 
- <b>New structured columns for deeper analysis</b>
- <b>Fewer missing values, no duplicates</b> 

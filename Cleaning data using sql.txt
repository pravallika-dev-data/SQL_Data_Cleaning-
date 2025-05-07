--Cleaning data using sql 
select * from [project 110]..[Nashville Housing Data for Data Cleaning]

--------------------------------------------------------------------------------------------------------------------

--when we look saledate it has time which has no purpose here so standardize date

select saledateconverted from [project 110]..[Nashville Housing Data for Data Cleaning]

select saledate, cast(saledate as date) from [project 110]..[Nashville Housing Data for Data Cleaning]

update [Nashville Housing Data for Data Cleaning]
set SaleDate=convert(date,saledate)--just want to show we can convert in two ways cast and convert


--update method is working but it is not replacing the original column
alter table [Nashville Housing Data for Data Cleaning] add saledateconverted date;

update [Nashville Housing Data for Data Cleaning]
set SaleDateconverted=convert(date,saledate) --now we can remove the saledate from the table



--------------------------------------------------------------------------------------------------

--populate propert address data , selfjoining 

select * from [project 110]..[Nashville Housing Data for Data Cleaning]
order by ParcelID

select a.ParcelID,a.PropertyAddress,b.ParcelID,b.PropertyAddress,isnull(a.PropertyAddress,b.PropertyAddress)
from [project 110]..[Nashville Housing Data for Data Cleaning] a
join 
[project 110]..[Nashville Housing Data for Data Cleaning] b
on 
a.ParcelID = b.ParcelID
and 
a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null

update a 
set propertyaddress = isnull(a.PropertyAddress,b.PropertyAddress)
from [project 110]..[Nashville Housing Data for Data Cleaning] a
join 
[project 110]..[Nashville Housing Data for Data Cleaning] b
on 
a.ParcelID = b.ParcelID
and 
a.[UniqueID ] <> b.[UniqueID ]
where a.PropertyAddress is null




----------------------------------------------------------------------------------------------------------------

--breaking out address into individual columns(address,city,state)
 
 --property address

 select SUBSTRING(propertyaddress,1,CHARINDEX(',',propertyaddress) -1),
 SUBSTRING(propertyaddress,CHARINDEX(',',propertyaddress)+1,len(propertyaddress))
 from [project 110]..[Nashville Housing Data for Data Cleaning]

 alter table [Nashville Housing Data for Data Cleaning] add address nvarchar(255);

 update [Nashville Housing Data for Data Cleaning] 
 set address = SUBSTRING(propertyaddress,1,CHARINDEX(',',propertyaddress) -1)

  alter table [Nashville Housing Data for Data Cleaning] add city nvarchar(255);

  update [Nashville Housing Data for Data Cleaning] 
 set city = SUBSTRING(propertyaddress,CHARINDEX(',',propertyaddress)+1,len(propertyaddress))


  select * from [project 110]..[Nashville Housing Data for Data Cleaning];

  --owner address
  --we have seen oneway in the above step which is using substring and charindex 
  -- we can achieve same thing using parsenam

  select PARSENAME(replace(owneraddress,',','.'),3), 
  PARSENAME(replace(owneraddress,',','.'),2),
  PARSENAME(replace(owneraddress,',','.'),1)
  from [project 110]..[Nashville Housing Data for Data Cleaning]

  alter table [Nashville Housing Data for Data Cleaning] add oaddress nvarchar(255);

  update [Nashville Housing Data for Data Cleaning] 
 set oaddress = PARSENAME(replace(owneraddress,',','.'),3)

 alter table [Nashville Housing Data for Data Cleaning] add ocity nvarchar(255);

  update [Nashville Housing Data for Data Cleaning] 
 set ocity = PARSENAME(replace(owneraddress,',','.'),2)


 alter table [Nashville Housing Data for Data Cleaning] add ostate nvarchar(255);

  update [Nashville Housing Data for Data Cleaning] 
 set ostate = PARSENAME(replace(owneraddress,',','.'),1)


   select * from [project 110]..[Nashville Housing Data for Data Cleaning];



---------------------------------------------------------------------------------------------------------

--change y and n to yes and no in soldasvacant 

select soldasvacant,count(soldasvacant) from [project 110]..[Nashville Housing Data for Data Cleaning]
group by soldasvacant
order by 2

select soldasvacant, 
case when
soldasvacant='Y' then 'Yes'
when soldasvacant='N' then 'No'
ELSE soldasvacant
end 
from [project 110]..[Nashville Housing Data for Data Cleaning]

update [Nashville Housing Data for Data Cleaning]
set SoldAsVacant = case when
soldasvacant='Y' then 'Yes'
when soldasvacant='N' then 'No'
ELSE soldasvacant
end 





------------------------------------------------------------------------------------------------------


--removing duplicates
select * from [project 110]..[Nashville Housing Data for Data Cleaning]
GO


SELECT *,
        ROW_NUMBER() OVER (
            PARTITION BY ParcelID, PropertyAddress, SalePrice, SaleDate, LegalReference
            ORDER BY UniqueID
        ) AS row_num
    FROM [project 110]..[Nashville Housing Data for Data Cleaning]

	--- I DON'T KNOW WHY IT IS NOT WORKING BUT WE NEED TO STORE IT IN CTE TABLE USING WITH 
	--THEN DELETE ALL THE DUPLICATE ELEMENTS

WITH RowNumCTE AS(
Select *,
	ROW_NUMBER() OVER (
	PARTITION BY ParcelID,
				 PropertyAddress,
				 SalePrice,
				 SaleDate,
				 LegalReference
				 ORDER BY
					UniqueID
					) row_num

From [project 110]..[Nashville Housing Data for Data Cleaning]
--order by ParcelID
)

DELETE 
From RowNumCTE
Where row_num > 1

Select *
From RowNumCTE
Where row_num > 1
Order by PropertyAddress

---THAT SHOULD DELETE ALL THE DUPLICATE ELEMENTS



----------------------------------------------

--deleting unused columns


Select *
From [project 110]..[Nashville Housing Data for Data Cleaning]


ALTER TABLE [project 110]..[Nashville Housing Data for Data Cleaning]
DROP COLUMN OwnerAddress, TaxDistrict, PropertyAddress, SaleDate
# ABCD_medications
An r-markdown pipeline for processing medication data from the ABCD Study (https://abcdstudy.org/).

## ABCD Raw Files ##
This project uses variables from the following ABCD instruments [version 5.1]:

- ph_p_meds

- su_y_plus

## Outline of Pipeline ##
1. The project follows a directory structure like:

```
├── My_Project/
│   ├── raw/abcd-data-release-5.1/
│   │   └── *.csv
│   ├── docs/abcd_var_lists/
│   │   └── *.csv
```

- abcd-data-release-5.1/ should contain the raw, unprocessed download of ABCD data
  
- abcd\_var\_lists/ should contain a .csv data dictionary (which can be obtained here: https://data-dict.abcdstudy.org/) that will be used to inform the pipeline of which ABCD files and variables to pull information from and merge together.
	
2. The data dictionary file should contain the following columns: 
```
"Analysis_Role"	"Analysis_Keep"	"FileName"	"ElementName"	"DataType"	"var_name"	"var_label"	"notes	condition"	"table_name_nda"	"table_name".
```

- Analysis_Role and Analysis_Keep are prepended to the default columns and manually labelled by the researcher.
- Analysis_Role should have the value "meta" set for variables with ElementNames ("src_subject_id"	"interview_date"	"eventname") and the corresponding Analysis_Keep column should have the value "1" set for these meta items. These elements are common across different ABCD data files and will be used for merging variables across files. Any variables that should not be included can either be removed from the dictionary file or set to "0" or left as NA in Analysis_Keep. Here's an example meta item row:

```
"meta"	"1"	"abcd_y_lt"	"src_subject_id"	"String"	"src_subject_id"	"Subject ID how it's defined in lab/project"	"NA"	"NA"	"acspsw03"	"NA"
```

- Here's an example medication item row:

```
"1"	"ph_p_meds"	"med1_rxnorm_p"	"String"	"med1_rxnorm_p"	"Medication Name (please choose the formulation that most closely matches the medication taken by your child; no need to include dosage)  Nombre del Medicamento (por favor, elija la formulaciue mse aproxime a la medicaciomada por su ni); no hay necesidad de incluir la dosis)"	"This variable may include all variants available in the Bioportal RxNorm database (https://bioportal.bioontology.org). Please refer to this database for additional information on trade/generic names and formulations."	"brought_medications == 0"	"medsy01"	"ph_p_meds"
```

3. Follow Setup steps and instantiate all helper functions.

	- Make sure all availble timepoints are accounted for in the abcd_datevents function.

	- Assess whether abcd_setmissing is reasonable for how you want to handle specific values.
	
4. The remaining steps involve loading the data, and using the helper functions to merge everything together, add some additional meta variables, and handling special values.

5. Cleaning. This will be covered in more detail below.

## Cleaning Steps ##

1. Medication names are processed by their name group ("pls*_med*" or "med*_rxnorm_p"), which depends on the file from where they are derived.
	- Removing any duplicate medication items (i.e., within med name group, subject, and visit date)
	- The "med*_rxnorm_p" names contain a name and identifier number, so we split these values to create 2 new columns

3. Merge medication values across name groups and create new columns
	- Create a new list of column names to receive the transformed medication values compliled from both default name groups. 
	- These are named with the new prefix "medNums_***" with the last 3 digits corresponding to 1:n original medication columns.
	- The transformation is just left-shifting medication values in a matrix, forcing any reported values to occur in the leftmost columns instead of being distributed across the original columns inconsistently. This will allow dropping some columns that will become all NA and make it a little easier to manage the data.
	- Removeing any duplicate medication items (i.e., across med name group, within subject and visit date)

## Medication Dictionary and Subject-level Medication Group Assignment ##

1. Narrow down to unique pairs of a) medication numerical identifier and b) medication name. Essentially, creating a key that ties together any possible identifier with a text name.
	- This will output "medication_pairs" which can be saved off to a .csv and researcher can use this to derive a list of medication names to form medication groups of interest.

2. In this example, we create 3 groups: adhd medications, vitamins, and over the counter. The former is of interest, and the latter two for convenience. These may not be exhaustive for these categories and each researcher should assess based on their needs.

3. Determine which subjects are taking any of the medications in the designated medication group of interest (e.g., adhd meds).

## Plotting ##

1. This workbook also includes code to plot the overall frequency that each medication is reported in the dataset.
	- Its done by matching an originally reported medication item against the corresponding number in the dictionary and then pulling the corresponding text.
	- This involves tokenizing the medication names, then using the researcher-defined medication keywords to quantify how often keywords appear as tokens

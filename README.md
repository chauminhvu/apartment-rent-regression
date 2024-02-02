# Apartment Rental Prediction

This project aims to develop machine learning models for predicting rental property prices using both structural and text data. The dataset used for this project contains rental property listings from a German real estate platform, encompassing various features such as living area size, rent, location, and more.

## Repository Structure

The repository is organized as follows:

- `docs`: contains documentation and reports for the project
- `figures`: contains plots and visualizations generated by the models
- `models`: contains trained models and model evaluation results
- `notebooks`: contains Jupyter notebooks for data exploration, preprocessing, and modeling
- `src`: contains Python scripts for data processing, feature engineering, and model training
- `tests`: contains unit tests for the src modules
- `.gitignore`: specifies the files and directories to be ignored by Git
- `README.md`: provides an overview of the project and the repository
- `processing.sh`: runs the data processing pipeline using the src scripts
- `requirements-cuda.txt`: lists the Python packages required for running the project on a GPU
- `requirements.txt`: lists the Python packages required for running the project on a CPU

<a name="setup"></a>
## Setup

Clone this repository:

```bash
git clone https://github.com/chauminhvu/apartment-rent-regression.git
```

Install the dependencies:

```bash
pip install -r requirements.txt # for CPU
pip install -r requirements-cuda.txt # for GPU
```

Download the dataset from [here](https://www.kaggle.com/corrieaar/apartment-rental-offers-in-germany) and place it in the `data` folder.


## Part 1: Predicting Rent with Structural Data

In this part, only the structural features of the rental properties, such as service charge, living area size, number of rooms, location, etc., to predict the rent.

### 1.1 **Data Processing**:
The detailed process and detail explanation are provided in the notebook `src/notebooks/01-data-processing.ipynb`. Highlight:
- *Data Cleaning*: The notebook performs various data cleaning steps, such as removing duplicated columns, and dropping all colmns that has missing ppercentages **35%**.
- *Outlier detection:* Applying the median absolute deviation (MAD) criteria by Leys et al. (2013). The columns are cleaned: `['baseRent', 'totalRent','serviceCharge','livingSpace', 'noRooms', 'floor', 'yearConstruct']`.
- *Missing value inputation*: Separate into 2 parts: 
   - The first part is inputing values base on the insides of the dataset, such as `'totalRent' = 'baseRent' + 'serviceCharge' + residue`. Methods to keep the statistical properties of the dataset are performed.
   - The remaining missed values colmns are filled by `MissForest` algorithm by Stekhoven and Bühlmann (2012).

- *Features engineering:* Creation of `distance` columns are done by matching zipcodes of each listing. The results reduced 6 columns related to location.
   - The distance to a specific location is measurable, numeric, and distinct. Stuttgart is chosen for a reference point.
   - The data for coordinates is from: https://gist.github.com/iteufel/af379872bbc3bf5261e2fd09b681ff7e.\
Then, it is converted to csv using function `sql2csv` in `src/data_processing.py`
- *Data Visualization*: The notebook also creates some plots to explore the distribution and correlation of the features and the target variable `totalRent`.
- *Data Export*: The notebook saves the processed data into a CSV file for further analysis and modeling.
- *Unit tests:* `tests/` stores Unit test files.

*Bibliography*\
Leys, C., Ley, C., Klein, O., Bernard, P., & Licata, L. (2013). Detecting outliers: Do not use standard deviation around the mean, use absolute deviation around the median. Journal of Experimental Social Psychology, 49(4), 764–766. https://doi.org/10.1016/j.jesp.2013.03.013

Stekhoven, D. J., & Bühlmann, P. (2012). MissForest—Non-parametric missing value imputation for mixed-type data. Bioinformatics, 28(1), 112–118. https://doi.org/10.1093/bioinformatics/btr597


### 1.2 **Structural data for price prediction using Deep Neural Networks and XGBoost**:
- Trained models are saved in `src/models`
- Inputs: 
   - Drop `baseRent` and `picturecount`. Total rent = base rent + service charge + err (details in section filling missing data of `01-data-processing.ipynb`), so, it makes no sense to keep `base rent`, there will be no need for machine learning models.
   - Category columns are encoded using one-hot encoding.
- Output: `totalRent`

- *Multilayer perceptron (MLP):* defined using `Pytorch`, hyperparameters are tuned using `Optuna` train using `Pytorch-lighting`

- *XGBRegressor* defined using `xgboost`, hyperparameters are tuned using `Optuna` train using `xgboost`
- Performance: $R^2 (MLP)= 0.8$; $R^2 (XGB)= 0.9$

## Part 2: Predicting Rent with Structural and Text Data
- Download folder `multi_models` and place it in the main folder `apartment-rent-regression`. It is about 500 MB, therefore, I have problems with git.
- The download link: https://drive.google.com/drive/folders/1r6HNT2szh_ImIUsA2d3uMljEuyZAA9Cj?usp=sharing
- Trained models are saved in `src/multimodal`
- Inputs: included textual data: `description`, `facilities`
- Output: `totalRent`

- A Transformer based model: `Multimodal-Net` is employed, training is conducted using `autogluon`
- Performance: $R^2 (multimodal)= 0.84$

# Furture:
- Investigating other methods for cleaning textual data, i.e 
   - Using LLMs to convert German texts to English, since most of the pre-trained models currently perform better on English.
   - Clean original texts
- Exploiting information from texts to confirm the values of structural columns, i.e. filling NaN of other columns using information from `description`, veryfing the `bool` columns.

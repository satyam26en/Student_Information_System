```markdown
# Student Information System

A Streamlit application for managing and displaying student information. This application allows users to view and download the details of students, including their core section and professional electives. The data is sourced from CSV files hosted on GitHub.

## Objective

The objective of this project is to provide an interactive and user-friendly web interface for managing and viewing student information. By leveraging Streamlit, this application aims to simplify the process of accessing and analyzing student data, ensuring that users can easily retrieve and download information about students' core sections and professional electives.

## Features

- View roll numbers and elective details for specific core sections.
- View roll numbers and elective details for specific professional elective sections.
- View details of individual students by roll number.
- View details of students within a range of roll numbers.
- Download the displayed data as a CSV file.
- Interactive and user-friendly web interface using Streamlit.

## Installation

### Prerequisites

- Python 3.7 or higher

### Clone the Repository

```sh
git clone https://github.com/satyam26en/student-information-system.git
cd student-information-system
```

### Install the Required Packages

Create and activate a virtual environment:

```sh
python -m venv venv
source venv/bin/activate   # On Windows, use `venv\Scripts\activate`
```

Install the required packages:

```sh
pip install -r requirements.txt
```

## Usage

### Running the Application Locally

1. Ensure you are in the project directory:

   ```sh
   cd student-information-system
   ```

2. Run the Streamlit app:

   ```sh
   streamlit run app.py
   ```

3. Open your web browser and navigate to the provided URL (usually `http://localhost:8501`).

### Deploying the Application on Streamlit Cloud

1. Commit and push all your changes to your GitHub repository.
2. Go to [Streamlit Cloud](https://streamlit.io/cloud) and log in.
3. Click on "New app".
4. Connect your GitHub account and select the repository.
5. Select the branch (usually `main` or `master`) and the `app.py` file.
6. Click on "Deploy".

You can view the deployed app at [this link](https://gkswyb65o6xma3pdnxk7gy.streamlit.app/).

## File Structure

```
student-information-system/
├── app.py                    # Main application file
├── data/
│   ├── elective_sheet1.csv   # Elective section data
│   ├── core_section_sheet1.csv # Core section data
├── notebooks/
│   ├── data_preprocessing.ipynb # Jupyter notebook for data preprocessing (if needed)
├── requirements.txt          # List of dependencies
└── README.md                 # Project README file
```

## Data Sources

The application uses the following CSV files hosted on GitHub:

- [Elective Section Data](https://raw.githubusercontent.com/satyam26en/student-information-system/main/data/elective_sheet1.csv)
- [Core Section Data](https://raw.githubusercontent.com/satyam26en/student-information-system/main/data/core_section_sheet1.csv)

## How to Use the Application

1. **Select a Section**: Use the sidebar to navigate through the options.
   - **View by Section**: Choose whether to view core sections or professional electives.
   - **View by Roll Number**: Enter a specific roll number or a range of roll numbers.

2. **View Data**: The application will display the relevant data based on your selection.

3. **Download Data**: Click the "Download results as CSV" button to download the displayed data.

## Screenshots

![App Screenshot 1](https://raw.githubusercontent.com/satyam26en/student-information-system/main/images/1.png)
![App Screenshot 2](https://raw.githubusercontent.com/satyam26en/student-information-system/main/images/2.png)
![App Screenshot 3](https://raw.githubusercontent.com/satyam26en/student-information-system/main/images/3.png)

## Contributing

Contributions are welcome! Please fork this repository and submit a pull request for any improvements.

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contact

For any inquiries or support, please contact [your-email@example.com].

## Repository

[GitHub Repository](https://github.com/satyam26en/student-information-system)

---

### Example `requirements.txt`

```plaintext
streamlit==1.18.1
pandas==2.0.3
natsort==8.3.1
```

### `app.py`

Ensure your `app.py` uses the correct paths:

```python
import pandas as pd
from natsort import natsorted
import streamlit as st

# Load the CSV files from local data directory
@st.cache_data
def load_data():
    elective_df = pd.read_csv('data/elective_sheet1.csv')
    core_df = pd.read_csv('data/core_section_sheet1.csv')

    # Rename columns for better clarity
    elective_df.columns = elective_df.iloc[0]
    elective_df = elective_df[1:]

    core_df.columns = core_df.iloc[0]
    core_df = core_df[1:]

    # Merge the dataframes on 'Roll No.'
    merged_df = pd.merge(elective_df, core_df, on='Roll No.', how='outer')

    return merged_df

# Function to filter and process the dataframe
def filter_and_process_data(merged_df):
    # Split the elective sections into two columns
    merged_df['Professional Elective 1'] = merged_df.groupby('Roll No.')['Elective section'].transform(lambda x: x.iloc[0])
    merged_df['Professional Elective 2'] = merged_df.groupby('Roll No.')['Elective section'].transform(lambda x: x.iloc[1] if len(x) > 1 else None)
    
    # Drop duplicates and unnecessary columns
    merged_df = merged_df[['Roll No.', 'Professional Elective 1', 'Professional Elective 2', 'Core Section']].drop_duplicates()
    
    # Function to rearrange elective sections
    def rearrange_electives(row):
        if row['Professional Elective 1'] and row['Professional Elective 1'].startswith(('HPC', 'DOS')):
            row['Professional Elective 1'], row['Professional Elective 2'] = row['Professional Elective 2'], row['Professional Elective 1']
        return row

    # Apply the function to each row in the dataframe
    merged_df = merged_df.apply(rearrange_electives, axis=1)

    # Sort the dataframe by 'Roll No.' and return
    return merged_df.sort_values(by='Roll No.').reset_index(drop=True)

# Function to get details for a specific section
def get_details_by_section(processed_df, section, section_type='core'):
    if section_type == 'core':
        result_df = processed_df[processed_df['Core Section'] == section]
    else:
        result_df = processed_df[(processed_df['Professional Elective 1'] == section) | (processed_df['Professional Elective 2'] == section)]
    return result_df[['Roll No.', 'Core Section', 'Professional Elective 1', 'Professional Elective 2']]

# Function to get details of a specific roll number
def get_details_by_roll_number(processed_df, roll_number):
    result_df = processed_df[processed_df['Roll No.'].astype(int) == roll_number]
    return result_df[['Roll No.', 'Core Section', 'Professional Elective 1', 'Professional Elective 2']]

# Main function
def main():
    # Load and process data
    merged_df = load_data()
    processed_df = filter_and_process_data(merged_df)

    # Get unique section lists and sort them naturally
    core_sections = natsorted(processed_df['Core Section'].dropna().unique())
    prof_elective_1_sections = natsorted(processed_df['Professional Elective 1'].dropna().unique())
    prof_elective_2_sections = natsorted(processed_df['Professional Elective 2'].dropna().unique())

    # Streamlit UI
    st.sidebar.title("Student Information System")
    st.sidebar.markdown("### Navigate through the options to explore student information")

    query_section = st.sidebar.radio("Do you want to know the roll list of any particular section?", ('Yes', 'No'))
    
    st.title("Student Information System Dashboard")
    st.markdown("""
        <style>
        .main {
            background-color: #f5f5f5;
        }
        </style>
        """, unsafe_allow_html=True)
    
    if query_section == 'Yes':
        section_type = st.sidebar.radio("Select the type of section:", ('Core Section', 'Professional Elective'))
        
        if section_type == 'Core Section':
            section_name = st.sidebar.selectbox("Select a core section:", core_sections)
        else:
            elective_type = st.sidebar.radio("Select the type of professional elective:", ('Professional Elective 1', 'Professional Elective 2'))
            if elective_type == 'Professional Elective 1':
                section_name = st.sidebar.selectbox("Select a professional elective 1 section:", prof_elective_1_sections)
            else:
                section_name = st.sidebar.selectbox("Select a professional elective 2 section:", prof_elective_2_sections)
            section_type = elective_type
        
        section_result_df = get_details_by_section(processed_df

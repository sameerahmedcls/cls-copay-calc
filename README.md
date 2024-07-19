# Patient Responsibility Calculator

This Streamlit app allows users to calculate the total patient responsibility based on the entered CPT codes and their corresponding patient responsibility percentages. The app uses a CSV file (`mdfs.csv`) containing the 'Non Facility Fee' for each CPT code to perform the calculations.

## Features

- Input fields to enter CPT codes and patient responsibility percentages.
- Display of entered CPT codes and responsibilities along with their corresponding 'Non Facility Fee' from the CSV file.
- Calculation of total patient responsibility.
- Option to remove all entered CPT codes and responsibilities.
- User-friendly interface with an explanation dropdown for guidance.

## How to Use

1. **Enter CPT Code**: Input the CPT code you see from a billing note.
2. **Enter % Patient Responsibility**: Input the patient's responsibility percentage.
3. **Add CPT Code**: Click the 'Add CPT Code' button to add the CPT code and responsibility to the list.
4. **Remove All Rows**: If needed, click the 'Remove All Rows' button to clear all entered CPT codes and responsibilities.
5. **View Entered Data**: The entered data will be displayed on the right side, along with the corresponding 'Non Facility Fee' from the MDFS data.
6. **Calculate Total Responsibility**: Click the 'Calculate Total Responsibility' button to see the total patient responsibility amount.

## Installation

1. Clone the repository:
    ```bash
    git clone https://github.com/sameerahmedcls/cls-copay-cal.git
    cd cls-copay-cal
    ```

2. Install the required dependencies:
    ```bash
    pip install -r requirements.txt
    ```

3. Ensure you have the `mdfs.csv` file in the same directory as the app.

## Running the App

To run the app, use the following command:
```bash
streamlit run app.py
```

This will start the Streamlit app, and you can access it in your web browser.

## Code Explanation

```python
import streamlit as st
import pandas as pd

# Load the CSV file into a DataFrame
@st.cache_data
def load_data():
    return pd.read_csv('mdfs.csv')

data = load_data()

# Initialize session state for storing CPT codes and patient responsibilities
if 'cpt_codes' not in st.session_state:
    st.session_state.cpt_codes = []
if 'responsibilities' not in st.session_state:
    st.session_state.responsibilities = []

st.title('Patient Responsibility Calculator')

# Explanation dropdown
with st.expander("How to use this tool"):
    st.write("""
        1. Enter the CPT code you see from a billing note in the 'Enter CPT Code' field.
        2. Enter the patient's responsibility percentage in the 'Enter % Patient Responsibility' field.
        3. Click 'Add CPT Code' to add the CPT code and responsibility to the list.
        4. If needed, you can remove all entered CPT codes and responsibilities by clicking 'Remove All Rows'.
        5. The entered data will be displayed on the right side, along with the corresponding 'Non Facility Fee' from the MDFS data.
        6. Click 'Calculate Total Responsibility' to see the total patient responsibility amount.
    """)

# Layout the app with input fields on the left and data on the right
col1, col2 = st.columns(2)

with col1:
    # Input fields for CPT code and patient responsibility
    cpt_code = st.text_input('Enter CPT Code')
    patient_responsibility = st.number_input('Enter % Patient Responsibility', min_value=0, max_value=100, step=1)

    # Button to add the CPT code and responsibility to the list
    col1_1, col1_2 = st.columns(2)
    with col1_1:
        if st.button('Add CPT Code'):
            if cpt_code and patient_responsibility is not None:
                st.session_state.cpt_codes.append(cpt_code)
                st.session_state.responsibilities.append(patient_responsibility)

    with col1_2:
        if st.session_state.cpt_codes:
            if st.button('Remove All Rows'):
                st.session_state.cpt_codes = []
                st.session_state.responsibilities = []

with col2:
    # Display the entered CPT codes and responsibilities
    if st.session_state.cpt_codes:
        input_data = pd.DataFrame({
            'CPT Code': st.session_state.cpt_codes,
            '% Patient Responsibility': st.session_state.responsibilities
        })
        
        # Merge the input data with the MDFS data to include 'Non Facility Fee'
        merged_input_data = input_data.merge(data[['CPT', 'Non Facility Fee']], left_on='CPT Code', right_on='CPT', how='left')
        merged_input_data['% pts'] = merged_input_data['% Patient Responsibility'].astype(str) + '%'
        merged_input_data.set_index('CPT Code', inplace=True)
        st.write(merged_input_data[['% pts', 'Non Facility Fee']])

        merged_data = input_data.merge(data, left_on='CPT Code', right_on='CPT')

        # Calculate the total patient responsibility
        merged_data['Patient Responsibility'] = (merged_data['% Patient Responsibility'] / 100) * merged_data['Non Facility Fee']
        total_responsibility = merged_data['Patient Responsibility'].sum()

        if st.button('Calculate Total Responsibility'):
            st.write(f'### Total Patient Responsibility: ${total_responsibility:.2f}')

if st.session_state.cpt_codes:
    # Display the MDFS data for the entered CPT codes
    merged_data = input_data.merge(data, left_on='CPT Code', right_on='CPT')
    merged_data.set_index('CPT Code', inplace=True)

    st.write('### MDFS Data:')
    st.write(merged_data)
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

# Job Skills Analysis for Immigrant Career Placement

## Overview

This project analyzes job market data to identify high-growth career opportunities for immigrants and maps them to required skills using Bureau of Labor Statistics (BLS) employment data and O*NET skills database. The analysis supports the development of targeted job skills workshops for adult learners.

## Academic Foundation

This research was conducted under **Prof. Prakash Shrivastava, PhD**, Clinical Professor at the Naveen Jindal School of Management, University of Texas at Dallas. The project developed a Python-based skills-matching application to facilitate immigrant job placement using data analysis algorithms.

## Project Structure

```
├── JobAnalysis.ipynb          # Main analysis notebook
├── occupation.xlsx            # BLS employment data
├── Skills.xlsx               # O*NET skills database
├── filtered_jobs.csv         # Output: filtered high-growth jobs
├── job-workshop/             # Workshop website files
└── README.md                # This documentation
```

## Data Sources

1. **BLS Employment Data** (`occupation.xlsx`): Bureau of Labor Statistics employment projections 2023-2033
2. **O*NET Skills Data** (`Skills.xlsx`): Occupational skills importance ratings from O*NET database

## Code Walkthrough

### Cell 1: Job Filtering and Data Cleaning

```python
import pandas as pd
import numpy as np

# Load the Excel file
file_path = "occupation.xlsx"
df = pd.read_excel(file_path)
```

**What's happening:**
- Imports required libraries for data manipulation
- Loads BLS employment data from Excel file
- **Note:** Requires `openpyxl` library for Excel file reading

#### Data Cleaning Steps

```python
# Step 1: Replace '—' and other non-numeric entries with NaN
wage_col = 'Median annual wage, dollars, 2024[1]'
df[wage_col] = df[wage_col].replace('—', np.nan)

# Step 2: Remove dollar signs and commas
df[wage_col] = df[wage_col].replace('[\\$,]', '', regex=True)

# Step 3: Convert to float
df[wage_col] = pd.to_numeric(df[wage_col], errors='coerce')
```

**Purpose:**
- Cleans wage data by removing non-numeric characters
- Converts wage column to numeric format for filtering
- Handles missing data (represented as '—') by converting to NaN

#### Job Filtering Criteria

```python
# Step 4: Filter in-demand jobs
filtered = df[
    (df['Employment change, percent, 2023-33'] > 3) &
    (df['Occupational openings, 2023-33 annual average'] >= 900) &
    (df[wage_col] > 30000)
]
```

**Filtering Logic:**
- **Growth Rate:** Employment change > 3% (faster than average growth)
- **Job Availability:** ≥900 annual job openings (sufficient opportunities)
- **Wage Threshold:** >$30,000 annual salary (living wage consideration)

#### Output Generation

```python
# Step 5: Select relevant columns
result = filtered[[
    '2023 National Employment Matrix title',
    '2023 National Employment Matrix code',
    'Employment change, percent, 2023-33',
    'Occupational openings, 2023-33 annual average',
    wage_col
]]

# Export results
result.to_csv('filtered_jobs.csv', index=False)
```

**Output:** Creates `filtered_jobs.csv` with high-growth, well-paying jobs suitable for immigrant job seekers.

---

### Cell 2: Basic Skills Matching

```python
# Load filtered jobs CSV
jobs_df = pd.read_csv('filtered_jobs.csv')

# Extract SOC codes as strings and strip
target_soc_codes = jobs_df['2023 National Employment Matrix code'].astype(str).tolist()
target_soc_codes = [code.strip() for code in target_soc_codes]
```

**What's happening:**
- Loads the filtered jobs from previous step
- Extracts Standard Occupational Classification (SOC) codes
- Cleans whitespace from codes

#### SOC Code Formatting

```python
# Add decimal suffix if missing to match O*NET format
def format_soc_code(code):
    if '.' not in code:
        return code + '.00'
    return code

target_soc_codes = [format_soc_code(code) for code in target_soc_codes]
```

**Purpose:** Standardizes SOC codes to match O*NET database format (e.g., "11-1011" → "11-1011.00")

#### Skills Data Integration

```python
# Load O*NET skills data
skills_df = pd.read_excel('Skills.xlsx')

# Filter for Importance ratings only
importance_df = skills_df[skills_df['Scale ID'] == 'IM']

# Filter for your occupations with formatted SOC codes
importance_df = importance_df[importance_df['O*NET-SOC Code'].isin(target_soc_codes)]

# Filter for skills with Importance > 2
important_skills = importance_df[importance_df['Data Value'] > 2]
```

**Process:**
1. Loads O*NET skills database
2. Filters for "Importance" scale ratings (vs. Level ratings)
3. Matches skills to our filtered job codes
4. Keeps only skills with importance rating > 2 (moderately important or higher)

---

### Cell 3: Advanced Skills Mapping with Code Expansion

```python
# Get all unique SOC codes in the O*NET dataset
all_onet_codes = skills_df['O*NET-SOC Code'].unique().tolist()

# Function to expand broad SOC codes to detailed codes
def get_detailed_codes(general_code, all_codes):
    # Remove trailing '-0000.00' to get prefix (first two digits + '-')
    prefix = general_code.split('-')[0] + '-'
    # Select detailed codes that start with prefix and do NOT end with '0000.00'
    detailed_codes = [code for code in all_codes if code.startswith(prefix) and not code.endswith('0000.00')]
    return detailed_codes
```

**Advanced Feature:** 
- Handles broad occupational categories (e.g., "11-0000.00" = All Management Occupations)
- Expands them to specific job titles (e.g., "11-1011.00" = Chief Executives)
- Increases analysis coverage from 11 broad categories to 181 specific occupations

#### Code Expansion Logic

```python
# Expand broad SOC codes to detailed SOC codes
expanded_soc_codes = []
for soc in target_soc_codes:
    if soc.endswith('0000.00'):  # Broad category code
        detailed_list = get_detailed_codes(soc, all_onet_codes)
        if detailed_list:
            expanded_soc_codes.extend(detailed_list)
        else:
            expanded_soc_codes.append(soc)  # Keep original if no details found
    else:
        expanded_soc_codes.append(soc)  # Keep specific codes as-is

# Remove duplicates
expanded_soc_codes = list(set(expanded_soc_codes))
```

**Result:** Expands analysis from 11 job categories to 181 specific occupations for more granular skills mapping.

---

### Cell 4: Skills Output and Formatting

```python
# Group skills by detailed SOC code
skills_by_job = important_skills.groupby('O*NET-SOC Code')['Element Name'].apply(list)

# Format output for readability
for soc_code, skills_list in skills_by_job.items():
    print(f"{soc_code}:")
    for skill in skills_list:
        print(f" - {skill}")
    print()  # Empty line between jobs
```

**Output Format:**
```
11-1011.00:
 - Reading Comprehension
 - Active Listening
 - Writing
 - Speaking
 - Mathematics
 - Critical Thinking
 [... more skills]
```

**Purpose:** Creates human-readable output showing required skills for each occupation, informing workshop curriculum development.

## Key Insights from Analysis

### Common Skills Across High-Growth Jobs
- **Communication:** Reading Comprehension, Active Listening, Writing, Speaking
- **Cognitive:** Critical Thinking, Active Learning, Mathematics
- **Interpersonal:** Social Perceptiveness, Coordination, Persuasion
- **Management:** Time Management, Resource Management

### Workshop Curriculum Implications
1. **Core Skills Focus:** Communication and critical thinking appear in most occupations
2. **Transferable Skills:** Many skills apply across multiple job categories
3. **Skill Prioritization:** Importance ratings help prioritize workshop content

## Installation Requirements

```bash
# Required Python packages
pip install pandas numpy openpyxl

# Or using conda
conda install pandas numpy openpyxl
```

## Usage Instructions

1. **Prepare Data Files:**
   - Place `occupation.xlsx` (BLS data) in project directory
   - Place `Skills.xlsx` (O*NET data) in project directory

2. **Run Analysis:**
   ```bash
   jupyter notebook JobAnalysis.ipynb
   ```

3. **Execute Cells Sequentially:**
   - Cell 1: Filter high-growth jobs
   - Cell 2: Basic skills matching
   - Cell 3: Advanced code expansion
   - Cell 4: Format and display results

## Output Files

- **`filtered_jobs.csv`**: High-growth jobs meeting criteria
- **Console Output**: Skills lists for each occupation
- **Workshop Materials**: Informed by skills analysis results

## Applications

### Job Skills Workshop Development
- **Curriculum Design:** Focus on most important transferable skills
- **Resource Creation:** Develop materials for high-demand skills
- **Career Guidance:** Help immigrants identify skill development priorities

### Career Counseling
- **Skills Gap Analysis:** Compare immigrant backgrounds to job requirements
- **Training Recommendations:** Prioritize skill development based on importance ratings
- **Job Matching:** Connect skills to specific high-growth opportunities

## Research Methodology

1. **Data Integration:** Combines employment projections with skills requirements
2. **Filtering Algorithm:** Identifies jobs with growth, availability, and wage criteria
3. **Skills Mapping:** Links occupations to required competencies
4. **Importance Weighting:** Prioritizes skills based on O*NET importance ratings

## Future Enhancements

- **Geographic Analysis:** Add location-specific job market data
- **Skills Gap Scoring:** Quantify skill development needs
- **Interactive Dashboard:** Web-based tool for career exploration
- **Multilingual Support:** Translate skills descriptions for diverse learners

## Contact

For questions about this analysis or the associated workshop program:
- **Email:** contact.aspire2@gmail.com
- **Research Repository:** [GitHub Project](https://github.com/nanditashetty857/jobs-research/)

## Citation

If using this analysis in research or educational materials, please cite:
```
Job Skills Analysis for Immigrant Career Placement
Conducted under Prof. Prakash Shrivastava, PhD
Naveen Jindal School of Management, University of Texas at Dallas
```

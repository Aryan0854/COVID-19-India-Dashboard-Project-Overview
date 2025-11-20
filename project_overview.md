# COVID-19 India Dashboard Project Overview

## Project Summary

This is a Streamlit-based web application that displays real-time COVID-19 data for India and its states. The application fetches data from the publicly available COVID19-India API and presents it through an interactive dashboard with various visualizations.

## Team Members
- Aryan Mishra (20221CSG0011)
- Cepha G (20221CSG0006)
- Affan (20221CSG0020)
- Karthik MP (20221CSG0059)

## Technical Architecture

### Core Technologies
1. **Streamlit** - Web framework for creating data applications
2. **Python** - Primary programming language
3. **Requests** - HTTP library for API calls
4. **Pandas** - Data manipulation and analysis
5. **Plotly Express** - Interactive data visualization
6. **COVID19-India API** - Data source for COVID-19 statistics

### Key Components

#### 1. Data Fetching Layer
- `fetch_india_covid_data()` - Retrieves latest national-level COVID data
- `fetch_india_historical_data()` - Retrieves historical COVID trends
- `get_state_data()` - Extracts specific state information from aggregated data

#### 2. User Interface Layer
- Interactive input for selecting up to 3 Indian states
- Responsive dashboard layout with metrics display
- Tabbed interface for different data visualizations
- Expandable sections for raw data inspection

#### 3. Visualization Components
- State-wise comparison charts (confirmed, recovered, deaths)
- Combined metrics visualization
- Historical trend analysis for India
- Interactive Plotly charts with hover information

## Features and Functionality

### Main Dashboard Sections

1. **India Overview**
   - Total confirmed cases
   - Total recovered cases
   - Total deaths
   - Daily new cases
   - Last updated timestamp

2. **State-wise Data**
   - Up to 3 states comparison side-by-side
   - Confirmed, recovered, and death counts
   - Active cases tracking
   - Recovery rate calculation
   - Last updated timestamps

3. **Interactive Charts**
   - Tabbed interface for different metrics
   - Bar charts comparing states
   - Grouped bar chart for combined metrics
   - Line chart for historical trends

4. **Historical Trends**
   - 30-day trend visualization
   - Confirmed, recovered, and deceased trends
   - Interactive time-series chart

### Technical Features

1. **Caching Mechanism**
   - 120-second TTL for API responses
   - Reduces server load and improves response times
   - Balances data freshness with performance

2. **Error Handling**
   - Graceful API failure handling
   - User-friendly error messages
   - Warning displays for missing data

3. **Data Processing**
   - State name normalization and matching
   - Numeric formatting with thousands separators
   - Percentage calculations for recovery rates
   - Date parsing and formatting

## Data Sources

### Primary API
- **Endpoint**: https://data.covid19india.org/data.json
- **Data Types**: 
  - National cumulative statistics
  - State-wise current data
  - Historical time-series data
- **Update Frequency**: Daily updates from official sources

### Data Fields
- Total confirmed cases
- Total recovered cases
- Total deceased cases
- Daily case counts
- State-level breakdowns
- Timestamps and last update information

## Implementation Details

### File Structure
- Single file implementation: `Proj.py`
- Self-contained application with all components
- Clear section separation with comments

### Code Organization
1. **Header Section** - Imports and configuration
2. **User Input** - State selection interface
3. **Helper Functions** - API data fetching and processing
4. **Data Processing** - Aggregation and transformation
5. **Display Logic** - UI rendering and visualization
6. **Footer** - Informational content and metadata

### Performance Optimizations
- API response caching with TTL
- Efficient data filtering and processing
- Lazy loading of visualization components
- Minimal redundant data fetching

## Usage Instructions

### Running the Application
1. Install required dependencies:
   ```
   pip install streamlit requests pandas plotly
   ```
2. Execute the application:
   ```
   python -m streamlit run Proj.py
   ```

### User Interaction
1. Enter up to 3 Indian state names (comma-separated)
2. View national and state-level statistics
3. Explore interactive charts and visualizations
4. Examine raw data through expandable sections

## Project Strengths

1. **Real-time Data**: Uses live data from official sources
2. **Interactive Visualizations**: Rich, engaging charts with Plotly
3. **Responsive Design**: Adapts to different screen sizes
4. **Performance Optimization**: Caching reduces API load
5. **Error Resilience**: Handles API failures gracefully
6. **User Experience**: Intuitive interface with clear information hierarchy

## Potential Improvements

1. **Data Validation**: Enhanced input validation for state names
2. **Extended History**: Longer historical data visualization
3. **Export Features**: Data export capabilities (CSV, PDF)
4. **Additional Metrics**: Vaccination data integration
5. **Mobile Optimization**: Enhanced mobile experience
6. **Internationalization**: Multi-language support

## Dependencies

### Required Packages
- streamlit>=1.0.0
- requests>=2.25.0
- pandas>=1.3.0
- plotly>=5.0.0
- datetime (standard library)

### Installation Command
```bash
pip install streamlit requests pandas plotly
```

## Deployment Considerations

### Local Development
- Runs directly with Streamlit's development server
- No additional infrastructure required
- Easy setup and iteration

### Production Deployment
- Can be deployed to Streamlit Cloud
- Compatible with Docker containerization
- Suitable for cloud platforms (AWS, GCP, Azure)
- Lightweight resource requirements

## Conclusion

This COVID-19 dashboard demonstrates effective use of Python data science libraries to create an informative, interactive web application. The project successfully combines real-time data fetching, thoughtful data processing, and engaging visualizations to present complex information in an accessible format. The modular code structure and clear documentation make it maintainable and extensible for future enhancements.
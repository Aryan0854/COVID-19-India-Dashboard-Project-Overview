#Name:-
#Aryan Mishra  20221CSG0011
#Cepha G           20221CSG0006
#Affan                20221CSG0020
#Karthik MP      20221CSG0059
#python -m streamlit run Proj.py

import streamlit as st
import requests
import pandas as pd
import plotly.express as px
from datetime import datetime

st.set_page_config(page_title="COVID-19 India Dashboard", layout="wide")
st.title("🦠 COVID-19 Dashboard — India (COVID19-India API)")
st.write(
    "Enter up to three Indian states. The app shows COVID-19 data from the COVID19-India API "
    "which provides state-level data for Indian states. Data is updated daily."
)

# --- User input ---
default_states = "Maharashtra, Kerala, Karnataka"
states_input = st.text_input("Enter up to 3 Indian state names (comma-separated)", default_states)
state_list = [s.strip() for s in states_input.split(",") if s.strip()][:3]

# --- Helpers: COVID19-India API queries ---
@st.cache_data(ttl=120)
def fetch_india_covid_data():
    """Fetch India-level COVID data from COVID19-India API"""
    try:
        # Fetch latest data for India
        url = "https://data.covid19india.org/data.json"
        r = requests.get(url, timeout=10)
        
        if r.status_code == 200:
            data = r.json()
            if 'cases_time_series' in data and len(data['cases_time_series']) > 0:
                # Get the latest entry
                latest_data = data['cases_time_series'][-1]
                return {"level": "country", "data": latest_data, "url": r.url, "statewise": data.get('statewise', [])}
        else:
            st.warning(f"API returned status {r.status_code} for India data")
    except Exception as e:
        st.error(f"Error fetching India data: {str(e)}")
        return None
    return None

@st.cache_data(ttl=120)
def fetch_india_historical_data():
    """Fetch historical COVID data for India from COVID19-India API"""
    try:
        # Fetch historical data for India
        url = "https://data.covid19india.org/data.json"
        r = requests.get(url, timeout=10)
        
        if r.status_code == 200:
            data = r.json()
            if 'cases_time_series' in data:
                return data['cases_time_series']
        else:
            st.warning(f"API returned status {r.status_code} for historical India data")
    except Exception as e:
        st.error(f"Error fetching historical India data: {str(e)}")
        return None
    return None

def get_state_data(state_name, statewise_data):
    """Extract data for a specific state from statewise data"""
    if not statewise_data or not state_name:
        return None
    
    # Normalize state names for matching
    state_name_lower = state_name.lower().strip()
    
    for state in statewise_data:
        state_name_in_data = state.get('state', '').lower().strip()
        state_code = state.get('statecode', '').lower().strip()
        
        # Match by state name or state code
        if (state_name_lower == state_name_in_data or 
            state_name_lower == state_code or
            state_name_lower in state_name_in_data or
            state_name_in_data in state_name_lower):
            return state
    
    return None

# --- Process data ---
india_data = fetch_india_covid_data()
historical_data = fetch_india_historical_data()
statewise_data = india_data.get('statewise', []) if india_data else []

# --- Display ---
if not state_list:
    st.error("Please enter at least one state name.")
else:
    # Display India-level data
    st.subheader("🇮🇳 India Overview")
    if india_data and india_data.get("data"):
        india_latest = india_data["data"]
        col1, col2, col3, col4 = st.columns(4)
        col1.metric("Total Cases", f"{int(india_latest.get('totalconfirmed', 0)):,}" if india_latest.get('totalconfirmed') else "N/A")
        col2.metric("Total Recovered", f"{int(india_latest.get('totalrecovered', 0)):,}" if india_latest.get('totalrecovered') else "N/A")
        col3.metric("Total Deaths", f"{int(india_latest.get('totaldeceased', 0)):,}" if india_latest.get('totaldeceased') else "N/A")
        col4.metric("Daily Cases", f"{int(india_latest.get('dailyconfirmed', 0)):,}" if india_latest.get('dailyconfirmed') else "N/A")
        
        st.write(f"Data as of: {india_latest.get('date', 'N/A')}")
    else:
        st.warning("No India-level data found.")
    
    # Display state-level data
    st.subheader("📍 State-wise Data")
    cols = st.columns(len(state_list))
    results = []
    
    for col, state in zip(cols, state_list):
        with col:
            st.subheader(state)
            state_data = get_state_data(state, statewise_data)
            
            if state_data:
                # Display state metrics
                col1, col2, col3 = st.columns(3)
                col1.metric("Confirmed", f"{int(state_data.get('confirmed', 0)):,}" if state_data.get('confirmed') else "N/A")
                col2.metric("Recovered", f"{int(state_data.get('recovered', 0)):,}" if state_data.get('recovered') else "N/A")
                col3.metric("Deaths", f"{int(state_data.get('deaths', 0)):,}" if state_data.get('deaths') else "N/A")
                
                # Additional metrics
                st.write("---")
                col4, col5, col6 = st.columns(3)
                col4.metric("Active", f"{int(state_data.get('active', 0)):,}" if state_data.get('active') else "N/A")
                col5.metric("Last Updated", state_data.get('lastupdatedtime', 'N/A').split(' ')[0] if state_data.get('lastupdatedtime') else "N/A")
                
                # Calculate recovery rate if possible
                confirmed = state_data.get('confirmed')
                recovered = state_data.get('recovered')
                if confirmed and recovered and int(confirmed) > 0:
                    recovery_rate = (int(recovered) / int(confirmed)) * 100
                    col6.metric("Recovery Rate", f"{recovery_rate:.1f}%")
                else:
                    col6.metric("Recovery Rate", "N/A")
                
                # Show raw data for debugging
                with st.expander("Raw Data"):
                    st.json(state_data)
                
                results.append({"state": state, "data": state_data})
            else:
                st.warning("No data found for this state.")
                results.append({"state": state, "data": None})
    
    # Create interactive charts
    st.subheader("📈 Interactive Charts")
    
    # Filter out states with no data
    valid_results = [r for r in results if r.get("data")]
    
    if valid_results:
        # Tabbed interface for different charts
        tab1, tab2, tab3 = st.tabs(["Cases Comparison", "Recovery Comparison", "Death Comparison"])
        
        with tab1:
            # Bar chart for confirmed cases comparison
            chart_data = []
            for result in valid_results:
                state_name = result["state"]
                state_data = result["data"]
                confirmed = int(state_data.get('confirmed', 0)) if state_data.get('confirmed') else 0
                chart_data.append({"State": state_name, "Confirmed Cases": confirmed})
            
            if chart_data:
                df_confirmed = pd.DataFrame(chart_data)
                fig_confirmed = px.bar(df_confirmed, x="State", y="Confirmed Cases", 
                                     title="Confirmed Cases by State",
                                     labels={'Confirmed Cases': 'Number of Cases', 'State': 'State'})
                st.plotly_chart(fig_confirmed, use_container_width=True)
        
        with tab2:
            # Bar chart for recovered cases comparison
            chart_data = []
            for result in valid_results:
                state_name = result["state"]
                state_data = result["data"]
                recovered = int(state_data.get('recovered', 0)) if state_data.get('recovered') else 0
                chart_data.append({"State": state_name, "Recovered Cases": recovered})
            
            if chart_data:
                df_recovered = pd.DataFrame(chart_data)
                fig_recovered = px.bar(df_recovered, x="State", y="Recovered Cases", 
                                     title="Recovered Cases by State",
                                     labels={'Recovered Cases': 'Number of Cases', 'State': 'State'})
                st.plotly_chart(fig_recovered, use_container_width=True)
        
        with tab3:
            # Bar chart for deaths comparison
            chart_data = []
            for result in valid_results:
                state_name = result["state"]
                state_data = result["data"]
                deaths = int(state_data.get('deaths', 0)) if state_data.get('deaths') else 0
                chart_data.append({"State": state_name, "Deaths": deaths})
            
            if chart_data:
                df_deaths = pd.DataFrame(chart_data)
                fig_deaths = px.bar(df_deaths, x="State", y="Deaths", 
                                  title="Deaths by State",
                                  labels={'Deaths': 'Number of Deaths', 'State': 'State'})
                st.plotly_chart(fig_deaths, use_container_width=True)
        
        # Combined metrics visualization
        st.subheader("📊 Combined Metrics Comparison")
        combined_data = []
        for result in valid_results:
            state_name = result["state"]
            state_data = result["data"]
            combined_data.append({
                'State': state_name,
                'Confirmed': int(state_data.get('confirmed', 0)) if state_data.get('confirmed') else 0,
                'Recovered': int(state_data.get('recovered', 0)) if state_data.get('recovered') else 0,
                'Deaths': int(state_data.get('deaths', 0)) if state_data.get('deaths') else 0,
            })
        
        if combined_data:
            df_combined = pd.DataFrame(combined_data)
            # Melt the dataframe for better visualization
            df_melted = df_combined.melt(id_vars=['State'], 
                                       value_vars=['Confirmed', 'Recovered', 'Deaths'],
                                       var_name='Metric', value_name='Value')
            
            fig_combined = px.bar(df_melted, x="State", y="Value", color="Metric", 
                                title="Comparison of All Metrics by State",
                                barmode='group')
            st.plotly_chart(fig_combined, use_container_width=True)
    
    # Historical trend chart for India
    if historical_data:
        st.subheader("📈 India Historical Trends")
        try:
            # Prepare data for plotting
            dates = []
            confirmed = []
            recovered = []
            deceased = []
            
            # Take last 30 days of data
            recent_data = historical_data[-30:] if len(historical_data) > 30 else historical_data
            
            for entry in recent_data:
                dates.append(entry.get('dateymd', ''))
                confirmed.append(int(entry.get('totalconfirmed', 0)) if entry.get('totalconfirmed') else 0)
                recovered.append(int(entry.get('totalrecovered', 0)) if entry.get('totalrecovered') else 0)
                deceased.append(int(entry.get('totaldeceased', 0)) if entry.get('totaldeceased') else 0)
            
            if dates and (confirmed or recovered or deceased):
                # Create dataframe for plotting
                df_hist = pd.DataFrame({
                    'Date': dates,
                    'Confirmed': confirmed,
                    'Recovered': recovered,
                    'Deceased': deceased
                })
                
                # Convert date column to datetime
                df_hist['Date'] = pd.to_datetime(df_hist['Date'])
                
                # Melt for plotting
                df_hist_melted = df_hist.melt(id_vars=['Date'], 
                                            value_vars=['Confirmed', 'Recovered', 'Deceased'],
                                            var_name='Metric', value_name='Count')
                
                fig_hist = px.line(df_hist_melted, x="Date", y="Count", color="Metric", 
                                 title="India COVID-19 Trends (Last 30 Days)")
                st.plotly_chart(fig_hist, use_container_width=True)
        except Exception as e:
            st.warning(f"Could not process historical data: {str(e)}")

    st.caption(f"Last updated: {datetime.now().strftime('%Y-%m-%d %H:%M:%S')}")
    st.markdown("***")
    st.info(
        "- This app uses the COVID19-India API (https://data.covid19india.org/) which provides Indian state-level data.\n"
        "- Data is sourced from official Indian government sources and updated daily.\n"
        "- Metrics include confirmed cases, recoveries, deaths, and active cases.\n"
        "- Caching TTLs are short (120s) for fresher results — increase if you want fewer API calls."
    )

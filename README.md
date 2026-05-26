import streamlit as st
import pandas as pd
import numpy as np
import plotly.express as px
import re

# Page Configuration
st.set_page_config(page_title="Nagaland State Lottery Forensics", layout="wide", page_icon="📊")

# --- CUSTOM CSS FOR CLEAN LOOK ---
st.markdown("""
    <style>
    .block-container {padding-top: 2rem; padding-bottom: 2rem;}
    .stAlert p {font-weight: 500;}
    </style>
""", unsafe_allow_html=True)

# Title Structure
st.title("📊 Nagaland State Lottery Forensics Tracker")
st.caption("Custom Analytics Engine for Daily Dear Morning / Day / Evening Results Data Sheets")
st.markdown("---")

# --- SIDEBAR: DATA INGESTION ---
with st.sidebar:
    st.header("📁 Data Ingestion")
    uploaded_files = st.file_uploader(
        "Upload Past Results (Official PDF or Text Charts)", 
        type=["pdf", "txt"], 
        accept_multiple_files=True
    )
    
    st.info("💡 **Mode:** Demonstration Mock Data Active. Upload official 1 PM, 6 PM, or 8 PM PDF results to parse live patterns.")

# --- MOCK DATA ENGINE (Tailored for Nagaland Format) ---
@st.cache_data
def generate_nagaland_mock_data():
    # Simulate extraction of 5th prize numbers (100 numbers per draw, across 10 draws)
    np.random.seed(42)
    mock_5th_prizes = [f"{np.random.randint(0, 10000):04d}" for _ in range(1000)]
    
    # Extract ending double digits (00-99) for pattern recognition
    last_two_digits = [num[-2:] for num in mock_5th_prizes]
    
    # Simulate high prize alphabet series counts
    alphabets = ['A', 'B', 'C', 'D', 'E', 'G', 'H', 'J', 'K', 'L']
    alphabet_counts = {letter: np.random.randint(15, 60) for letter in alphabets}
    
    return pd.DataFrame({"4_digit": mock_5th_prizes, "last_2": last_two_digits}), alphabet_counts

df_prizes, dict_alphabets = generate_nagaland_mock_data()

# --- MAIN LAYOUT CONTEXT ---
with st.expander("👁️ View Processed Result Dataset Matrix", expanded=False):
    col_view1, col_view2 = st.columns(2)
    with col_view1:
        st.subheader("Extracted 4-Digit Winning Sequences")
        st.dataframe(df_prizes, use_container_width=True, height=250)
    with col_view2:
        st.subheader("Letter Series Frequency Matrix")
        st.json(dict_alphabets)

# --- NAVIGATION TABS ---
tab1, tab2, tab3 = st.tabs([
    "📊 4-Digit Ending Analysis", 
    "❄️ Alphabet & Series Metrics", 
    "🎲 Dear Ticket Generator"
])

# TAB 1: 4-DIGIT ENDING ANALYSIS
with tab1:
    st.subheader("Distribution Map of Drawn Numbers (Last 2 Digits)")
    st.markdown("Analyzing the structural frequency of the final matching digits in the **5th Prize Pool (₹120)**.")
    
    # Calculate frequencies for the ending combinations
    freq_series = df_prizes['last_2'].value_counts().sort_index().reset_index()
    freq_series.columns = ['Last Two Digits (00-99)', 'Total Times Drawn']
    
    # Plotly Bar Chart configuration to match original design profile
    fig_bars = px.bar(
        freq_series, 
        x='Last Two Digits (00-99)', 
        y='Total Times Drawn',
        color='Total Times Drawn',
        color_continuous_scale='viridis',
    )
    fig_bars.update_layout(xaxis_tickangle=-90, height=450, margin=dict(t=10, b=10, l=0, r=0))
    st.plotly_chart(fig_bars, use_container_width=True)

# TAB 2: HOT & COLD METRICS (ALPHABETS & SERIES)
with tab2:
    st.subheader("Alphabet Series Distribution Trends")
    st.markdown("Identifies variations across the middle sequence letters tied to top prize tiers.")
    
    df_alpha = pd.DataFrame(list(dict_alphabets.items()), columns=['Series Letter', 'Draw Count']).sort_values(by='Draw Count', ascending=False)
    
    col_g1, col_g2 = st.columns()
    with col_g1:
        fig_alpha = px.bar(df_alpha, x='Series Letter', y='Draw Count', color='Draw Count', color_continuous_scale='plasma')
        fig_alpha.update_layout(height=350)
        st.plotly_chart(fig_alpha, use_container_width=True)
    with col_g2:
        st.markdown("### 📈 Summary Metrics")
        st.metric(label="Hottest Series Letter", value=df_alpha.iloc['Series Letter'], delta=f"Count: {df_alpha.iloc['Draw Count']}")
        st.metric(label="Coldest Series Letter", value=df_alpha.iloc[-1]['Series Letter'], delta=f"Count: {df_alpha.iloc[-1]['Draw Count']}", delta_color="inverse")

# TAB 3: SMART TICKET GENERATOR
with tab3:
    st.subheader("Smart Combination Assembler")
    st.markdown("Generates full-sequence mock ticket structures weighted against historical metrics.")
    
    col_input1, col_input2 = st.columns(2)
    with col_input1:
        select_method = st.selectbox("Strategy Bias Profiles", ["High-Frequency Hot Numbers", "Under-Drawn Cold Numbers", "Pure Random Distribution"])
    with col_input2:
        ticket_count = st.slider("Number of tickets to generate", 1, 5, 3)
        
    st.markdown("### Generated Ticket Series Recommendations")
    
    # Code to build combinations matching Nagaland's format: [2-digit Series] + [Letter] + [5-digit Number]
    for i in range(ticket_count):
        mock_prefix = np.random.randint(50, 100)
        mock_letter = np.random.choice(list(dict_alphabets.keys()))
        
        if select_method == "High-Frequency Hot Numbers":
            # Pick a hot ending 2-digit sequence
            hot_ending = df_prizes['last_2'].value_counts().index[i]
            prefix_3_digits = f"{np.random.randint(0, 1000):03d}"
            mock_five_digit = f"{prefix_3_digits}{hot_ending}"
        else:
            mock_five_digit = f"{np.random.randint(0, 100000):05d}"
            
        st.code(f"🎟️ Ticket #{i+1}: {mock_prefix}{mock_letter} {mock_five_digit}", language="text")

# --- FORENSIC DISCLAIMER FOOTNOTE ---
st.markdown("---")
st.warning(
    "⚠️ **Forensic Disclaimer:** This tool processes descriptive statistics for historical data tracking. "
    "It does not predict future independent events or modify underlying lottery house edge profiles. "
    "Conducted for structural pattern research purposes only."
)

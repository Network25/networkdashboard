import streamlit as st
import plotly.express as px
import pandas as pd
import os
import warnings

warnings.filterwarnings('ignore')

st.set_page_config(page_title="Operation Analysis", page_icon=":bar_chart:", layout="wide")

st.title(":bar_chart: Rider/Driver Performance")
st.markdown('<style>div.block-container{padding-top:2rem;}</style>', unsafe_allow_html=True)

# File uploader for multiple files
fls = st.file_uploader(":file_folder: Upload a file", type=["csv", "txt", "xlsx", "xls"], accept_multiple_files=True)

# Initialize an empty list to store the dataframes
dfs = []

# Process each uploaded file
if fls:
    for fl in fls:
        if fl.name.endswith(".csv"):
            # Read CSV file
            df = pd.read_csv(fl, encoding="ISO-8859-1")
        elif fl.name.endswith(".xlsx") or fl.name.endswith(".xls"):
            # Read Excel file
            df = pd.read_excel(fl)
        else:
            st.error(f"Unsupported file type for {fl.name}!")
            continue
        
        # Append the dataframe to the list
        dfs.append(df)

    # Combine all dataframes into one
    if dfs:
        combined_df = pd.concat(dfs, ignore_index=True)

else:
    # Fallback to a default file if no file is uploaded
    os.chdir(r"D:\STREAMLIT")
    df = pd.read_csv("rptRiderJobDtlAll.csv", encoding="ISO-8859-1")
    combined_df = df  # Use the fallback data in case no file is uploaded
    st.write("Using default file")

# Add date column and filter by date
combined_df["Date"] = pd.to_datetime(combined_df["Date"])

startDate = combined_df["Date"].min()
endDate = combined_df["Date"].max()

col1, col2 = st.columns((2, 2))
with col1:
    date1 = st.date_input("Start Date", startDate)
with col2:
    date2 = st.date_input("End Date", endDate)

combined_df = combined_df[(combined_df["Date"] >= pd.to_datetime(date1)) & (combined_df["Date"] <= pd.to_datetime(date2))].copy()

# Sidebar filters
st.sidebar.header("Choose your filter:")
filtered_df = combined_df.copy()  # Make sure to filter the combined dataframe

# Sidebar Service Type Filter with Checkboxes (Bike/Van)
col1, col2 = st.sidebar.columns(2)
with col1:
    bike_selected = st.checkbox("Bike", value=False)

with col2:
    van_selected = st.checkbox("Van", value=False)

# Filter the data based on the selected checkboxes
if bike_selected and van_selected:
    filtered_df = combined_df.copy()  # Show both Bike and Van
elif bike_selected:
    filtered_df = combined_df[combined_df["Service Type"] == "Bike"]
elif van_selected:
    filtered_df = combined_df[combined_df["Service Type"] == "Van"]
else:
    filtered_df = combined_df.copy()  # Show both Bike and Van by default if neither is selected

# Remove the data table display here
# st.dataframe(filtered_df)  # This line is removed to prevent data from being displayed

# Delivery Type Filter
deltype = st.sidebar.multiselect("Select Delivery Type", combined_df["Job Type"].unique())
if deltype:
    filtered_df = filtered_df[filtered_df["Job Type"].isin(deltype)]

# Delivery Status Filter
status = st.sidebar.multiselect("Select Delivery Status", combined_df["Status"].unique())
if status:
    filtered_df = filtered_df[filtered_df["Status"].isin(status)]

# Employment Type Filter
emptype = st.sidebar.multiselect("Select Employment Type", combined_df["type"].unique())
if emptype:
    filtered_df = filtered_df[filtered_df["type"].isin(emptype)]

# Company Name Filter
company = st.sidebar.multiselect("Select Company Name", combined_df["Bill To Company"].unique())
if company:
    filtered_df = filtered_df[filtered_df["Bill To Company"].isin(company)]

# Rider Name Filter
name = st.sidebar.multiselect("Select Rider/Driver Name", combined_df["Short Name"].unique())
if name:
    filtered_df = filtered_df[filtered_df["Short Name"].isin(name)]

# Group data for visualization
jobs_by_rider = filtered_df.groupby("Short Name").size().reset_index(name="Number of Jobs")
jobs_by_rider = jobs_by_rider.sort_values(by="Number of Jobs", ascending=False)  # Sort in descending order by Number of Jobs

# Plotting Number of Jobs by Rider Name
with st.container():
    st.subheader("Number of Jobs by Field Staff")
    if not jobs_by_rider.empty:
        fig = px.bar(
            jobs_by_rider,
            x="Short Name",
            y="Number of Jobs",
            text="Number of Jobs",
            color="Short Name",
            title="Jobs per Rider",
            template="plotly_white",
        )
        fig.update_traces(textposition="outside", textfont_size=14) 
        fig.update_layout(
            height=600, 
            width=700, 
            xaxis_title="Rider Name", 
            yaxis_title="Number of Jobs", 
            font=dict(size=14),  # Increase axis labels font size
        )
        st.plotly_chart(fig, use_container_width=True)
    else:
        st.write("No data available for the selected filters.")

# Pie chart for Delivery Type distribution
delivery_type_distribution = filtered_df.groupby("Job Type").size().reset_index(name="Number of Jobs")
delivery_type_distribution = delivery_type_distribution.sort_values(by="Number of Jobs", ascending=False)  # Sort by value

with st.container():
    st.subheader("Total Jobs by Delivery Type")
    fig2 = px.pie(delivery_type_distribution, names="Job Type", values="Number of Jobs", title="Jobs Distribution by Delivery Type")
    fig2.update_layout(font=dict(family="Arial Black", size=18),height=600)  # Increase font size for labels
    st.plotly_chart(fig2, use_container_width=True)

# Pie chart for Delivery Status distribution
status_distribution = filtered_df.groupby("Status").size().reset_index(name="Number of Jobs")
status_distribution = status_distribution.sort_values(by="Number of Jobs", ascending=False)  # Sort by value

# Pie chart for Service Type distribution
service_type_distribution = filtered_df.groupby("Service Type").size().reset_index(name="Number of Jobs")
service_type_distribution = service_type_distribution.sort_values(by="Number of Jobs", ascending=False)  # Sort by value

# Display both pie charts in the same row
with st.container():
    st.subheader("Jobs by Delivery Status and Service Type")
    
    # Create two columns
    col1, col2 = st.columns(2)

    with col1:
        # Jobs by Delivery Status (Pie Chart)
        fig3 = px.pie(
            status_distribution,
            names="Status",
            values="Number of Jobs",
            title="Jobs by Delivery Status",
            template="plotly_white",
        )
        fig3.update_layout(font=dict(family="Arial Black", size=18),height=600)
        st.plotly_chart(fig3, use_container_width=True)

    with col2:
        # Jobs by Service Type (Pie Chart)
        fig4 = px.pie(
            service_type_distribution,
            names="Service Type",
            values="Number of Jobs",
            title="Jobs by Service Type",
            template="plotly_white",
        )
        fig4.update_layout(font=dict(family="Arial Black", size=18),height=600)
        st.plotly_chart(fig4, use_container_width=True)

# Additional pie charts (Jobs by Company and Employment Type)
company_distribution = filtered_df.groupby("Bill To Company").size().reset_index(name="Number of Jobs")
company_distribution = company_distribution.sort_values(by="Number of Jobs", ascending=False)

employment_type_distribution = filtered_df.groupby("type").size().reset_index(name="Number of Jobs")
employment_type_distribution = employment_type_distribution.sort_values(by="Number of Jobs", ascending=False)

# Display both charts in the same row
with st.container():
    st.subheader("Jobs by Company and Employment Type")
    
    # Create two columns
    col1, col2 = st.columns(2)

    with col1:
        # Jobs by Company Name (Pie Chart)
        fig_company = px.pie(
            company_distribution,
            names="Bill To Company",
            values="Number of Jobs",
            title="Jobs by Company Name",
            template="plotly_white",
        )
        fig_company.update_layout(font=dict(family="Arial Black", size=18),height=600)
        st.plotly_chart(fig_company, use_container_width=True)

    with col2:
        # Jobs by Employment Type (Pie Chart)
        fig_employment_type = px.pie(
            employment_type_distribution,
            names="type",
            values="Number of Jobs",
            title="Jobs by Employment Type",
            template="plotly_white",
        )
        fig_employment_type.update_layout(font=dict(family="Arial Black", size=18),height=600)
        st.plotly_chart(fig_employment_type, use_container_width=True)

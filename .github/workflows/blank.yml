import streamlit as st
import pandas as pd
from datetime import datetime, timedelta

# Define your substitution rules here
SUBSTITUTIONS = {
    'CFAR': ['IFAR','SFAR','FFAR'],
    'IFAR': ['SFAR','FFAR'],
    'CDMR': ['IDMR','SDMR','FDMR'],
    'ILAR': ['SLAR','FLAR'],
    # add other ACRISS groups as needed...
}

st.title("ACRISS Booking & Inventory Matcher")

# Upload CSVs
b_file = st.file_uploader("Upload Bookings CSV", type="csv")
i_file = st.file_uploader("Upload Inventory CSV", type="csv")

if st.button("Match") and b_file and i_file:
    bookings = pd.read_csv(b_file, names=["time","acriss"], parse_dates=[0])
    inv = pd.read_csv(i_file, names=["acriss","reg","model","return_time"], parse_dates=[3], keep_default_na=False)
    inv["return_time"] = inv["return_time"].replace({0: datetime.now()})
    used = set()
    results = {}

    bookings['time'] = bookings['time'].dt.strftime("%H:%M")
    for time_block, grp in bookings.groupby("time"):
        lines = []
        deadline = datetime.strptime(time_block, "%H:%M") - timedelta(hours=1)
        avail = inv[inv.return_time <= deadline].copy()
        for _, row in grp.iterrows():
            found = False
            for cand in [row.acriss] + SUBSTITUTIONS.get(row.acriss, []):
                sel = avail[(avail.acriss==cand) & (~avail.reg.isin(used))]
                if not sel.empty:
                    car = sel.iloc[0]
                    used.add(car.reg)
                    lines.append(f"{row.acriss} – {car.reg} – {car.model}")
                    found = True
                    break
            if not found:
                lines.append(f"{row.acriss} – 🟥 NO CAR")
        results[time_block] = lines

    for t, lines in results.items():
        st.markdown(f"**{t}**")
        for l in lines:
            st.text(l)

if st.button("Reset"):
    st.experimental_rerun()

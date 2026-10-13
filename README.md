# ai-work-agent
Personal AI work dashboard for finding opportunities, preparing tasks, tracking approvals and payments.
import streamlit as st
import pandas as pd
from datetime import datetime

st.set_page_config(
    page_title="AI Work Agent",
    page_icon="🤖",
    layout="wide"
)

if "jobs" not in st.session_state:
    st.session_state.jobs = []

if "activities" not in st.session_state:
    st.session_state.activities = [
        "Dashboard initialized successfully."
    ]

st.title("🤖 AI Work Agent")
st.caption("Your personal workspace for online work opportunities")

st.sidebar.title("Navigation")

page = st.sidebar.radio(
    "Open section",
    [
        "Overview",
        "Opportunities",
        "AI Activity",
        "Approvals",
        "Files & Earnings"
    ]
)

jobs = st.session_state.jobs

if page == "Overview":
    st.header("📊 Overview")

    col1, col2, col3 = st.columns(3)

    col1.metric("Total Opportunities", len(jobs))
    col2.metric(
        "Pending Approvals",
        sum(j["Status"] == "Needs Approval" for j in jobs)
    )

    earnings = sum(
        float(j["Payment"])
        for j in jobs
        if j["Status"] == "Paid"
    )

    col3.metric("Recorded Earnings", f"₹{earnings:.2f}")

    st.info(
        "This is your first prototype. "
        "Automatic job discovery and notifications "
        "will be added in later stages."
    )

    st.subheader("Recent Opportunities")

    if jobs:
        st.dataframe(pd.DataFrame(jobs), use_container_width=True)
    else:
        st.write("No opportunities recorded yet.")

elif page == "Opportunities":
    st.header("🔎 Opportunities")
    st.write("Record an opportunity you find online.")

    with st.form("add_job"):
        title = st.text_input("Job or task title")
        category = st.selectbox(
            "Category",
            [
                "Spreadsheet / Data Entry",
                "PowerPoint / Documents",
                "Online Research",
                "Data Labeling",
                "Other"
            ]
        )
        source = st.text_input("Platform or source")
        payment = st.number_input(
            "Expected payment (₹)",
            min_value=0.0,
            step=50.0
        )

        submitted = st.form_submit_button("Add Opportunity")

        if submitted:
            if title.strip():
                st.session_state.jobs.append({
                    "Title": title.strip(),
                    "Category": category,
                    "Source": source.strip(),
                    "Payment": payment,
                    "Status": "New",
                    "Added": datetime.now().strftime("%Y-%m-%d %H:%M")
                })

                st.session_state.activities.append(
                    f"Added opportunity: {title.strip()}"
                )

                st.success("Opportunity added.")
                st.rerun()
            else:
                st.warning("Enter a task title first.")

    if jobs:
        st.subheader("Your Opportunities")
        st.dataframe(pd.DataFrame(jobs), use_container_width=True)

elif page == "AI Activity":
    st.header("⚙️ AI Activity Log")
    st.write("A record of activity in this prototype.")

    for activity in reversed(st.session_state.activities):
        st.write("•", activity)

    st.caption(
        "The agent does not run background searches yet."
    )

elif page == "Approvals":
    st.header("✅ Approval Center")
    st.write("Review recorded opportunities and update their status.")

    for index, job in enumerate(jobs):
        with st.container(border=True):
            st.subheader(job["Title"])
            st.write("Category:", job["Category"])
            st.write("Expected payment: ₹", job["Payment"])
            st.write("Current status:", job["Status"])

            options = [
                "New",
                "Needs Approval",
                "Approved",
                "Submitted",
                "Paid",
                "Rejected"
            ]

            current = (
                job["Status"]
                if job["Status"] in options
                else "New"
            )

            new_status = st.selectbox(
                "Update status",
                options,
                index=options.index(current),
                key=f"status_{index}"
            )

            if st.button("Save Status", key=f"save_{index}"):
                st.session_state.jobs[index]["Status"] = new_status
                st.session_state.activities.append(
                    f"Updated '{job['Title']}' to {new_status}."
                )
                st.success("Status updated.")
                st.rerun()

elif page == "Files & Earnings":
    st.header("💰 Files & Earnings")

    if jobs:
        df = pd.DataFrame(jobs)

        st.subheader("Earnings Summary")

        paid = sum(
            float(j["Payment"])
            for j in jobs
            if j["Status"] == "Paid"
        )

        pending = sum(
            float(j["Payment"])
            for j in jobs
            if j["Status"] != "Paid"
        )

        col1, col2 = st.columns(2)
        col1.metric("Recorded as Paid", f"₹{paid:.2f}")
        col2.metric("Not Yet Paid", f"₹{pending:.2f}")

        st.download_button(
            "Download Job Tracker (CSV)",
            data=df.to_csv(index=False).encode("utf-8"),
            file_name="ai_work_agent_jobs.csv",
            mime="text/csv"
        )

        st. 
        caption(
            "These figures are manually recorded estimates, "
            "not verified payments."
        )
    else:
        st.write("Your job tracker is empty.")

st.divider()
st.caption("AI Work Agent • Prototype v1")

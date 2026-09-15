import math
import re
from io import BytesIO

import numpy as np
import pandas as pd
import streamlit as st
import plotly.express as px

# ============================================================
# PHARMAGUARD AI
# Drug Safety Signal Detector & CTD Submission Checker
# ============================================================

st.set_page_config(
    page_title="PharmaGuard AI",
    page_icon="💊",
    layout="wide",
    initial_sidebar_state="expanded",
)


# ============================================================
# CUSTOM CSS
# ============================================================

st.markdown(
    """
    <style>
    /* =========================================================
       PHARMAGUARD AI — PREMIUM UI
       ========================================================= */

    @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');

    .stApp {
        background:
            radial-gradient(circle at 90% 0%, rgba(37, 99, 235, 0.08), transparent 28%),
            radial-gradient(circle at 0% 30%, rgba(14, 165, 233, 0.06), transparent 25%),
            #f5f8fc;
        font-family: 'Inter', sans-serif;
    }

    /* Hide Streamlit's default chrome */
    #MainMenu {visibility: hidden;}
    footer {visibility: hidden;}
    header {background: transparent !important;}

    .block-container {
        padding-top: 2rem;
        padding-bottom: 3rem;
        max-width: 1450px;
    }

    /* Sidebar */
    [data-testid="stSidebar"] {
        background: linear-gradient(180deg, #071d33 0%, #0d2d4b 100%);
        border-right: 1px solid rgba(255,255,255,0.08);
    }

    [data-testid="stSidebar"] * {
        color: #eaf3fb !important;
    }

    [data-testid="stSidebar"] .stRadio label {
        padding: 10px 12px;
        border-radius: 10px;
        transition: 0.2s ease;
    }

    [data-testid="stSidebar"] .stRadio label:hover {
        background: rgba(255,255,255,0.08);
    }

    /* Main hero */
    .hero {
        position: relative;
        overflow: hidden;
        padding: 30px 34px;
        border-radius: 24px;
        margin-bottom: 24px;
        color: white;
        background: linear-gradient(135deg, #08213a 0%, #0b4f71 55%, #087f8c 100%);
        box-shadow: 0 18px 45px rgba(8, 33, 58, 0.20);
    }

    .hero:after {
        content: "✚";
        position: absolute;
        right: 38px;
        top: 18px;
        font-size: 120px;
        opacity: 0.08;
        transform: rotate(15deg);
    }

    .hero h1 {
        margin: 0;
        font-size: 42px;
        font-weight: 800;
        letter-spacing: -1.5px;
    }

    .hero p {
        margin: 8px 0 0;
        color: #d9f5fa;
        font-size: 17px;
        max-width: 760px;
    }

    .eyebrow {
        text-transform: uppercase;
        letter-spacing: 2px;
        font-size: 12px;
        font-weight: 700;
        color: #7ee7e9;
        margin-bottom: 8px;
    }

    /* Cards */
    .card {
        background: rgba(255,255,255,0.88);
        padding: 22px;
        border-radius: 18px;
        border: 1px solid #e2e8f0;
        margin-bottom: 16px;
        box-shadow: 0 8px 25px rgba(15, 23, 42, 0.06);
        transition: transform .2s ease, box-shadow .2s ease;
    }

    .card:hover {
        transform: translateY(-2px);
        box-shadow: 0 14px 32px rgba(15, 23, 42, 0.10);
    }

    .feature-card {
        min-height: 190px;
        background: linear-gradient(145deg, #ffffff, #f8fbff);
    }

    .feature-icon {
        font-size: 30px;
        margin-bottom: 8px;
    }

    .feature-title {
        color: #0b2942;
        font-size: 20px;
        font-weight: 800;
        margin-bottom: 8px;
    }

    .feature-text {
        color: #64748b;
        line-height: 1.6;
    }

    .pipeline {
        margin-top: 16px;
        padding: 13px 16px;
        border-radius: 12px;
        background: #eef7ff;
        border: 1px solid #d7eafb;
        color: #17476b;
        font-weight: 600;
        font-size: 14px;
    }

    /* Metric cards */
    [data-testid="stMetric"] {
        background: rgba(255,255,255,0.92);
        border: 1px solid #e2e8f0;
        border-radius: 16px;
        padding: 16px 18px;
        box-shadow: 0 7px 22px rgba(15, 23, 42, 0.06);
    }

    [data-testid="stMetricLabel"] {
        color: #64748b !important;
        font-weight: 600 !important;
    }

    [data-testid="stMetricValue"] {
        color: #0b2942 !important;
        font-weight: 800 !important;
    }

    /* Buttons */
    .stButton > button,
    .stDownloadButton > button {
        border-radius: 11px;
        min-height: 44px;
        font-weight: 700;
        border: 1px solid #d7e1eb;
        transition: all .2s ease;
    }

    .stButton > button:hover,
    .stDownloadButton > button:hover {
        transform: translateY(-1px);
        box-shadow: 0 8px 18px rgba(15, 23, 42, 0.10);
    }

    /* Inputs */
    .stSelectbox > div > div,
    .stNumberInput > div > div,
    .stTextArea textarea,
    [data-testid="stFileUploader"] {
        border-radius: 12px !important;
    }

    /* Upload area */
    [data-testid="stFileUploader"] {
        background: #ffffff;
        border: 1.5px dashed #9db8cc;
        padding: 8px;
    }

    /* Status cards */
    .critical {
        background: linear-gradient(135deg, #fff1f0, #fff8f7);
        border-left: 6px solid #dc2626;
        padding: 18px;
        border-radius: 14px;
        margin: 10px 0;
        box-shadow: 0 6px 18px rgba(220,38,38,.08);
    }

    .high {
        background: linear-gradient(135deg, #fff5eb, #fffaf5);
        border-left: 6px solid #ea580c;
        padding: 18px;
        border-radius: 14px;
        margin: 10px 0;
        box-shadow: 0 6px 18px rgba(234,88,12,.08);
    }

    .success-box {
        background: linear-gradient(135deg, #ecfdf5, #f7fffb);
        border-left: 6px solid #10b981;
        padding: 18px;
        border-radius: 14px;
    }

    .section-title {
        color: #0b2942;
        font-size: 24px;
        font-weight: 800;
        margin: 8px 0 4px;
    }

    .section-subtitle {
        color: #64748b;
        margin-bottom: 18px;
    }

    /* Tables */
    [data-testid="stDataFrame"] {
        border-radius: 14px;
        overflow: hidden;
        border: 1px solid #e2e8f0;
        box-shadow: 0 5px 18px rgba(15, 23, 42, 0.05);
    }

    /* Alerts */
    [data-testid="stAlert"] {
        border-radius: 12px;
    }

    hr {
        border: none;
        border-top: 1px solid #e2e8f0;
        margin: 28px 0;
    }

    @media (max-width: 900px) {
        .hero h1 { font-size: 30px; }
        .hero { padding: 24px; }
    }
    </style>
    """,
    unsafe_allow_html=True,
)


# ============================================================
# HEADER
# ============================================================

st.markdown(
    """
    <div class="hero">
        <div class="eyebrow">AI-POWERED PHARMACOVIGILANCE • REGULATORY INTELLIGENCE</div>
        <h1>💊 PharmaGuard AI</h1>
        <p>Drug Safety Signal Detection & Regulatory Submission Readiness — helping teams identify potential safety signals and discover CTD completeness gaps faster.</p>
    </div>
    """,
    unsafe_allow_html=True,
)

st.divider()


# ============================================================
# DEMO SAFETY DATA
# ============================================================

def create_demo_safety_data():

    rows = []

    report_id = 1

    # --------------------------------------------------------
    # DRUG A
    # --------------------------------------------------------

    drug_a_events = (
        ["CARDIAC EVENT"] * 25
        + ["HEADACHE"] * 8
        + ["NAUSEA"] * 6
        + ["RASH"] * 4
        + ["LIVER INJURY"] * 3
    )

    for event in drug_a_events:

        rows.append(
            {
                "report_id": report_id,
                "drug": "DRUG_A",
                "event": event,
            }
        )

        report_id += 1

    # --------------------------------------------------------
    # DRUG B
    # --------------------------------------------------------

    drug_b_events = (
        ["HEADACHE"] * 15
        + ["NAUSEA"] * 10
        + ["RASH"] * 8
        + ["CARDIAC EVENT"] * 2
        + ["LIVER INJURY"] * 2
    )

    for event in drug_b_events:

        rows.append(
            {
                "report_id": report_id,
                "drug": "DRUG_B",
                "event": event,
            }
        )

        report_id += 1

    # --------------------------------------------------------
    # DRUG C
    # --------------------------------------------------------

    drug_c_events = (
        ["HEADACHE"] * 12
        + ["NAUSEA"] * 9
        + ["RASH"] * 7
        + ["CARDIAC EVENT"] * 3
        + ["FEVER"] * 5
    )

    for event in drug_c_events:

        rows.append(
            {
                "report_id": report_id,
                "drug": "DRUG_C",
                "event": event,
            }
        )

        report_id += 1

    # --------------------------------------------------------
    # DRUG D
    # --------------------------------------------------------

    drug_d_events = (
        ["HEADACHE"] * 10
        + ["NAUSEA"] * 8
        + ["FEVER"] * 7
        + ["COUGH"] * 5
    )

    for event in drug_d_events:

        rows.append(
            {
                "report_id": report_id,
                "drug": "DRUG_D",
                "event": event,
            }
        )

        report_id += 1

    return pd.DataFrame(rows)


# ============================================================
# DEMO CTD DOSSIER
# ============================================================

DEMO_CTD = """
1.0 Regional Administrative Information
1.1 Table of Contents
1.2 Application Form
1.3 Product Information
1.4 Information About Experts

2.1 CTD Table of Contents
2.2 CTD Introduction
2.3 Quality Overall Summary
2.4 Nonclinical Overview
2.5 Clinical Overview
2.6 Nonclinical Written and Tabulated Summaries
2.7 Clinical Summary

3.1 Table of Contents
3.2 Body of Data
3.2.S Drug Substance
3.2.P Drug Product

4.1 Table of Contents
4.2 Study Reports
4.2.1 Pharmacology
4.2.2 Pharmacokinetics

5.1 Table of Contents
5.2 Tabular Listing
5.3 Clinical Study Reports
5.3.1 Reports of Biopharmaceutic Studies
5.3.2 Reports of Pharmacokinetic Studies
5.3.3 Reports of Pharmacodynamic Studies
5.3.4 Reports of Efficacy and Safety Studies
5.4 Literature References
"""


# ============================================================
# CTD REQUIREMENTS
# ============================================================

CTD_REQUIREMENTS = {

    "Module 1": [
        ("1.0", "Regional Administrative Information"),
        ("1.1", "Table of Contents"),
        ("1.2", "Application Form"),
        ("1.3", "Product Information"),
        ("1.4", "Information About Experts"),
        ("1.5", "Specific Requirements"),
    ],

    "Module 2": [
        ("2.1", "CTD Table of Contents"),
        ("2.2", "CTD Introduction"),
        ("2.3", "Quality Overall Summary"),
        ("2.4", "Nonclinical Overview"),
        ("2.5", "Clinical Overview"),
        ("2.6", "Nonclinical Written and Tabulated Summaries"),
        ("2.7", "Clinical Summary"),
    ],

    "Module 3": [
        ("3.1", "Table of Contents"),
        ("3.2", "Body of Data"),
        ("3.2.S", "Drug Substance"),
        ("3.2.P", "Drug Product"),
        ("3.2.A", "Appendices"),
        ("3.2.R", "Regional Information"),
    ],

    "Module 4": [
        ("4.1", "Table of Contents"),
        ("4.2", "Study Reports"),
        ("4.2.1", "Pharmacology"),
        ("4.2.2", "Pharmacokinetics"),
        ("4.2.3", "Toxicology"),
    ],

    "Module 5": [
        ("5.1", "Table of Contents"),
        ("5.2", "Tabular Listing"),
        ("5.3", "Clinical Study Reports"),
        ("5.3.1", "Reports of Biopharmaceutic Studies"),
        ("5.3.2", "Reports of Pharmacokinetic Studies"),
        ("5.3.3", "Reports of Pharmacodynamic Studies"),
        ("5.3.4", "Reports of Efficacy and Safety Studies"),
        ("5.3.5", "Reports of Safety Studies"),
        ("5.4", "Literature References"),
    ],
}


# ============================================================
# SAFETY FUNCTIONS
# ============================================================

def clean_safety_data(df):

    df = df.copy()

    df.columns = [
        str(column).strip().lower()
        for column in df.columns
    ]

    required_columns = {
        "report_id",
        "drug",
        "event",
    }

    missing = required_columns - set(df.columns)

    if missing:

        raise ValueError(
            "Missing required columns: "
            + ", ".join(sorted(missing))
        )

    df["report_id"] = (
        df["report_id"]
        .astype(str)
        .str.strip()
    )

    df["drug"] = (
        df["drug"]
        .fillna("UNKNOWN")
        .astype(str)
        .str.strip()
        .str.upper()
    )

    df["event"] = (
        df["event"]
        .fillna("UNKNOWN")
        .astype(str)
        .str.strip()
        .str.upper()
    )

    df = df[
        (df["report_id"] != "")
        & (df["drug"] != "")
        & (df["event"] != "")
    ]

    df = df.drop_duplicates(
        subset=[
            "report_id",
            "drug",
            "event",
        ]
    )

    return df.reset_index(drop=True)


def calculate_prr(a, b, c, d):

    drug_total = a + b
    other_drug_total = c + d

    if drug_total == 0:
        return 0.0

    if other_drug_total == 0:

        if a > 0:
            return math.inf

        return 0.0

    drug_event_rate = a / drug_total

    background_event_rate = (
        c / other_drug_total
    )

    if background_event_rate == 0:

        if drug_event_rate > 0:
            return math.inf

        return 0.0

    return drug_event_rate / background_event_rate


def get_contingency_table(
    df,
    drug,
    event,
):

    drug_mask = df["drug"] == drug
    event_mask = df["event"] == event

    a = int(
        (drug_mask & event_mask).sum()
    )

    b = int(
        (drug_mask & ~event_mask).sum()
    )

    c = int(
        (~drug_mask & event_mask).sum()
    )

    d = int(
        (~drug_mask & ~event_mask).sum()
    )

    return a, b, c, d


def classify_signal(
    cases,
    prr,
    minimum_cases,
    minimum_prr,
):

    if cases < minimum_cases:

        return False, "LOW"

    if not np.isfinite(prr):

        return True, "CRITICAL"

    if prr >= 5:

        return True, "CRITICAL"

    if prr >= 3:

        return True, "HIGH"

    if prr >= minimum_prr:

        return True, "MEDIUM"

    return False, "LOW"


def analyze_safety(
    df,
    selected_drug,
    minimum_cases,
    minimum_prr,
):

    df = clean_safety_data(df)

    drug = selected_drug.upper()

    if drug not in set(df["drug"]):

        raise ValueError(
            f"Drug '{drug}' was not found."
        )

    events = sorted(
        df["event"].unique()
    )

    results = []

    for event in events:

        a, b, c, d = get_contingency_table(
            df,
            drug,
            event,
        )

        if a == 0:
            continue

        prr = calculate_prr(
            a,
            b,
            c,
            d,
        )

        signal, priority = classify_signal(
            cases=a,
            prr=prr,
            minimum_cases=minimum_cases,
            minimum_prr=minimum_prr,
        )

        if np.isfinite(prr):

            prr_display = round(
                prr,
                3,
            )

        else:

            prr_display = "INF"

        results.append(
            {
                "Drug": drug,
                "Adverse Event": event,
                "Cases": a,
                "PRR": prr_display,
                "Signal": "YES" if signal else "NO",
                "Priority": priority,
                "a": a,
                "b": b,
                "c": c,
                "d": d,
            }
        )

    result_df = pd.DataFrame(results)

    if result_df.empty:
        return result_df

    priority_rank = {
        "CRITICAL": 4,
        "HIGH": 3,
        "MEDIUM": 2,
        "LOW": 1,
    }

    result_df["_rank"] = (
        result_df["Priority"]
        .map(priority_rank)
    )

    result_df = (
        result_df
        .sort_values(
            ["_rank", "Cases"],
            ascending=False,
        )
        .drop(columns="_rank")
        .reset_index(drop=True)
    )

    return result_df


# ============================================================
# SIMPLE EVENT CLUSTERING
# ============================================================

def cluster_events(df):

    df = clean_safety_data(df)

    events = sorted(
        df["event"].unique()
    )

    clusters = []

    cardiovascular = [
        "CARDIAC EVENT",
        "CHEST PAIN",
        "MYOCARDIAL INFARCTION",
        "ARRHYTHMIA",
        "HEART FAILURE",
    ]

    gastrointestinal = [
        "NAUSEA",
        "VOMITING",
        "DIARRHEA",
        "ABDOMINAL PAIN",
    ]

    neurological = [
        "HEADACHE",
        "DIZZINESS",
        "SEIZURE",
        "CONFUSION",
    ]

    hepatic = [
        "LIVER INJURY",
        "HEPATITIS",
        "LIVER FAILURE",
    ]

    for event in events:

        if event in cardiovascular:
            cluster = "Cardiovascular"

        elif event in gastrointestinal:
            cluster = "Gastrointestinal"

        elif event in neurological:
            cluster = "Neurological"

        elif event in hepatic:
            cluster = "Hepatic"

        else:
            cluster = "Other"

        clusters.append(
            {
                "Adverse Event": event,
                "Clinical Cluster": cluster,
            }
        )

    return pd.DataFrame(clusters)


# ============================================================
# CTD FUNCTIONS
# ============================================================

def normalize_text(text):

    text = str(text).lower()

    text = re.sub(
        r"[^a-z0-9.\s]",
        " ",
        text,
    )

    text = re.sub(
        r"\s+",
        " ",
        text,
    )

    return text.strip()


def section_exists(
    text,
    section_number,
    section_title,
):

    normalized = normalize_text(text)

    number_pattern = (
        r"(?<![0-9])"
        + re.escape(section_number)
        + r"(?![0-9])"
    )

    if re.search(
        number_pattern,
        normalized,
    ):

        return True

    title = normalize_text(
        section_title
    )

    if title in normalized:

        return True

    return False


def check_ctd(dossier_text):

    if not dossier_text.strip():

        raise ValueError(
            "Dossier content is empty."
        )

    module_results = []

    gaps = []

    for module, sections in CTD_REQUIREMENTS.items():

        found = 0

        for number, title in sections:

            exists = section_exists(
                dossier_text,
                number,
                title,
            )

            if exists:

                found += 1

            else:

                if number in {
                    "2.3",
                    "2.5",
                    "3.2",
                    "4.2",
                    "5.3",
                }:

                    severity = "CRITICAL"

                else:

                    severity = "MAJOR"

                gaps.append(
                    {
                        "Module": module,
                        "Section": number,
                        "Requirement": title,
                        "Severity": severity,
                        "Recommendation": (
                            "Verify and provide "
                            + number
                            + " "
                            + title
                            + "."
                        ),
                    }
                )

        total = len(sections)

        score = (
            found / total * 100
            if total > 0
            else 0
        )

        if score >= 90:

            status = "READY"

        elif score >= 75:

            status = "REVIEW"

        else:

            status = "ACTION REQUIRED"

        module_results.append(
            {
                "Module": module,
                "Required": total,
                "Found": found,
                "Missing": total - found,
                "Score": round(score, 1),
                "Status": status,
            }
        )

    results = pd.DataFrame(
        module_results
    )

    gaps_df = pd.DataFrame(
        gaps
    )

    overall_score = round(
        results["Score"].mean(),
        1,
    )

    return (
        results,
        gaps_df,
        overall_score,
    )


# ============================================================
# PDF EXTRACTION
# ============================================================

def extract_pdf_text(file):

    try:

        import fitz

    except ImportError:

        raise RuntimeError(
            "PyMuPDF is not installed. "
            "Run: pip install PyMuPDF"
        )

    pdf_bytes = file.read()

    document = fitz.open(
        stream=pdf_bytes,
        filetype="pdf",
    )

    pages = []

    for page in document:

        pages.append(
            page.get_text()
        )

    document.close()

    return "\n".join(pages)


# ============================================================
# SIDEBAR
# ============================================================

st.sidebar.title("💊 PharmaGuard AI")

mode = st.sidebar.radio(
    "Choose a module",
    [
        "🏠 Dashboard",
        "🔬 Safety Signal Detector",
        "📋 CTD Submission Checker",
    ],
)

st.sidebar.divider()

st.sidebar.caption(
    "AI-assisted decision support prototype"
)


# ============================================================
# DASHBOARD
# ============================================================

if mode == "🏠 Dashboard":

    st.markdown('<div class="section-title">Pharmaceutical Intelligence Dashboard</div>', unsafe_allow_html=True)
    st.markdown(
        '<div class="section-subtitle">One workspace for safety signal detection, clinical event clustering, and CTD submission readiness.</div>',
        unsafe_allow_html=True
    )

    col1, col2, col3, col4 = st.columns(4)

    col1.metric(
        "Adverse Event Reports",
        "20M+",
    )

    col2.metric(
        "CTD Modules",
        "5",
    )

    col3.metric(
        "Signal Metric",
        "PRR",
    )

    col4.metric(
        "Regulatory Framework",
        "ICH M4",
    )

    st.divider()

    left, right = st.columns(2)

    with left:

        st.markdown(
            """
            <div class="card feature-card">
                <div class="feature-icon">🔬</div>
                <div class="feature-title">Safety Signal Detection</div>
                <div class="feature-text">
                    Identify drug–event combinations with disproportionate
                    reporting patterns using PRR-based analysis.
                </div>
                <div class="pipeline">
                    DATA → CLEAN → PRR → CLUSTER → PRIORITIZE → REVIEW
                </div>
            </div>
            """,
            unsafe_allow_html=True
        )

        if st.button(
            "Open Safety Detector"
        ):

            st.info(
                "Select 'Safety Signal Detector' from the sidebar."
            )

    with right:

        st.markdown(
            """
            <div class="card feature-card">
                <div class="feature-icon">📋</div>
                <div class="feature-title">CTD Submission Readiness</div>
                <div class="feature-text">
                    Analyze a dossier outline against the five-module
                    CTD structure and surface missing sections.
                </div>
                <div class="pipeline">
                    DOSSIER → EXTRACT → MAP → CHECK → GAP REPORT
                </div>
            </div>
            """,
            unsafe_allow_html=True
        )

        if st.button(
            "Open CTD Checker"
        ):

            st.info(
                "Select 'CTD Submission Checker' from the sidebar."
            )

    st.divider()

    st.warning(
        """
        Important: This is a decision-support system.
        A statistical safety signal does not establish
        causality, and automated CTD completeness does
        not constitute regulatory approval.
        """
    )


# ============================================================
# SAFETY SIGNAL DETECTOR
# ============================================================

elif mode == "🔬 Safety Signal Detector":

    st.markdown('<div class="section-title">🔬 Drug Safety Signal Detector</div>', unsafe_allow_html=True)
    st.markdown(
        '<div class="section-subtitle">Calculate Proportional Reporting Ratio (PRR) and prioritize potential drug safety signals.</div>',
        unsafe_allow_html=True
    )

    st.subheader(
        "Upload Data"
    )

    uploaded_file = st.file_uploader(
        "Upload CSV file",
        type=["csv"],
        help=(
            "Required columns: "
            "report_id, drug, event"
        ),
    )

    if uploaded_file:

        try:

            safety_df = pd.read_csv(
                uploaded_file
            )

            st.success(
                "CSV loaded successfully."
            )

        except Exception as error:

            st.error(
                f"Could not read CSV: {error}"
            )

            st.stop()

    else:

        safety_df = create_demo_safety_data()

        st.info(
            "No CSV uploaded. Demo data is being used."
        )

    # --------------------------------------------------------
    # Validate
    # --------------------------------------------------------

    try:

        safety_df = clean_safety_data(
            safety_df
        )

    except Exception as error:

        st.error(
            str(error)
        )

        st.stop()

    st.write(
        f"**Records:** {len(safety_df):,}"
    )

    st.dataframe(
        safety_df.head(20),
        use_container_width=True,
    )

    st.divider()

    # --------------------------------------------------------
    # Controls
    # --------------------------------------------------------

    drugs = sorted(
        safety_df["drug"].unique()
    )

    selected_drug = st.selectbox(
        "Select drug",
        drugs,
    )

    col1, col2 = st.columns(2)

    with col1:

        minimum_cases = st.number_input(
            "Minimum number of cases",
            min_value=1,
            max_value=1000,
            value=3,
            step=1,
        )

    with col2:

        minimum_prr = st.number_input(
            "PRR signal threshold",
            min_value=0.1,
            max_value=100.0,
            value=2.0,
            step=0.1,
        )

    run_signal = st.button(
        "🚨 Run Safety Signal Detection",
        type="primary",
        use_container_width=True,
    )

    if run_signal:

        try:

            results = analyze_safety(
                safety_df,
                selected_drug,
                int(minimum_cases),
                float(minimum_prr),
            )

            clusters = cluster_events(
                safety_df
            )

            st.session_state[
                "safety_results"
            ] = results

            st.session_state[
                "safety_clusters"
            ] = clusters

        except Exception as error:

            st.error(
                f"Analysis failed: {error}"
            )

    # --------------------------------------------------------
    # RESULTS
    # --------------------------------------------------------

    if "safety_results" in st.session_state:

        results = st.session_state[
            "safety_results"
        ]

        clusters = st.session_state[
            "safety_clusters"
        ]

        st.divider()

        st.subheader(
            "🚨 Safety Analysis Results"
        )

        signal_results = results[
            results["Signal"] == "YES"
        ]

        high_results = results[
            results["Priority"].isin(
                ["HIGH", "CRITICAL"]
            )
        ]

        c1, c2, c3, c4 = st.columns(4)

        c1.metric(
            "Events Analyzed",
            len(results),
        )

        c2.metric(
            "Potential Signals",
            len(signal_results),
        )

        c3.metric(
            "High/Critical",
            len(high_results),
        )

        if not results.empty:

            finite_prrs = []

            for value in results["PRR"]:

                if value != "INF":

                    finite_prrs.append(
                        float(value)
                    )

            if finite_prrs:

                max_prr = max(
                    finite_prrs
                )

                c4.metric(
                    "Highest PRR",
                    max_prr,
                )

            else:

                c4.metric(
                    "Highest PRR",
                    "INF",
                )

        # ----------------------------------------------------
        # Result table
        # ----------------------------------------------------

        st.subheader(
            "Signal Table"
        )

        display_columns = [
            "Drug",
            "Adverse Event",
            "Cases",
            "PRR",
            "Signal",
            "Priority",
        ]

        st.dataframe(
            results[display_columns],
            use_container_width=True,
            hide_index=True,
        )

        # ----------------------------------------------------
        # PRR Chart
        # ----------------------------------------------------

        st.subheader(
            "📊 PRR Visualization"
        )

        chart_df = results.copy()

        chart_df["PRR Numeric"] = (
            chart_df["PRR"]
            .apply(
                lambda x:
                10
                if x == "INF"
                else float(x)
            )
        )

        chart_df = chart_df.sort_values(
            "PRR Numeric",
            ascending=False,
        )

        fig = px.bar(
            chart_df,
            x="Adverse Event",
            y="PRR Numeric",
            color="Priority",
            title=(
                "Proportional Reporting Ratio "
                f"for {selected_drug}"
            ),
            color_discrete_map={
                "CRITICAL": "#D92D20",
                "HIGH": "#E8590C",
                "MEDIUM": "#F2C94C",
                "LOW": "#12B76A",
            },
        )

        fig.add_hline(
            y=float(minimum_prr),
            line_dash="dash",
            line_color="red",
            annotation_text="PRR threshold",
        )

        st.plotly_chart(
            fig,
            use_container_width=True,
        )

        # ----------------------------------------------------
        # Clustering
        # ----------------------------------------------------

        st.subheader(
            "🧠 Adverse Event Clustering"
        )

        st.dataframe(
            clusters,
            use_container_width=True,
            hide_index=True,
        )

        cluster_counts = (
            clusters[
                "Clinical Cluster"
            ]
            .value_counts()
            .reset_index()
        )

        cluster_counts.columns = [
            "Cluster",
            "Number of Events",
        ]

        fig_cluster = px.bar(
            cluster_counts,
            x="Cluster",
            y="Number of Events",
            title="Adverse Event Clinical Clusters",
        )

        st.plotly_chart(
            fig_cluster,
            use_container_width=True,
        )

        # ----------------------------------------------------
        # Signal cards
        # ----------------------------------------------------

        if not signal_results.empty:

            st.subheader(
                "⚠️ Signals Requiring Expert Review"
            )

            for _, row in signal_results.iterrows():

                if row["Priority"] == "CRITICAL":

                    st.markdown(
                        f"""
                        <div class="critical">

                        <b>🚨 CRITICAL SIGNAL</b><br><br>

                        <b>Drug:</b>
                        {row["Drug"]}<br>

                        <b>Adverse Event:</b>
                        {row["Adverse Event"]}<br>

                        <b>Cases:</b>
                        {row["Cases"]}<br>

                        <b>PRR:</b>
                        {row["PRR"]}<br>

                        </div>
                        """,
                        unsafe_allow_html=True,
                    )

                else:

                    st.markdown(
                        f"""
                        <div class="high">

                        <b>⚠️ {row["Priority"]} PRIORITY</b><br><br>

                        <b>Drug:</b>
                        {row["Drug"]}<br>

                        <b>Adverse Event:</b>
                        {row["Adverse Event"]}<br>

                        <b>Cases:</b>
                        {row["Cases"]}<br>

                        <b>PRR:</b>
                        {row["PRR"]}<br>

                        </div>
                        """,
                        unsafe_allow_html=True,
                    )

        else:

            st.success(
                "No potential signals met the configured thresholds."
            )

        st.info(
            """
            Interpretation: PRR is a disproportionality
            measure. A statistical signal should be
            investigated by qualified pharmacovigilance
            professionals and does not by itself establish
            that a drug caused an adverse event.
            """
        )


# ============================================================
# CTD SUBMISSION CHECKER
# ============================================================

elif mode == "📋 CTD Submission Checker":

    st.markdown('<div class="section-title">📋 CTD Submission Readiness Checker</div>', unsafe_allow_html=True)
    st.markdown(
        '<div class="section-subtitle">Analyze a dossier outline against the five-module CTD structure and identify completeness gaps.</div>',
        unsafe_allow_html=True
    )

    uploaded_file = st.file_uploader(
        "Upload CTD dossier",
        type=["pdf", "txt"],
        help="Upload a text-based PDF or TXT dossier outline.",
    )

    dossier_text = ""

    if uploaded_file is None:

        dossier_text = DEMO_CTD

        st.info(
            "No dossier uploaded. Demo dossier is being used."
        )

    elif uploaded_file.name.lower().endswith(
        ".txt"
    ):

        try:

            dossier_text = (
                uploaded_file
                .read()
                .decode(
                    "utf-8",
                    errors="ignore",
                )
            )

        except Exception as error:

            st.error(
                f"Could not read TXT file: {error}"
            )

            st.stop()

    else:

        try:

            dossier_text = extract_pdf_text(
                uploaded_file
            )

            if not dossier_text.strip():

                st.error(
                    "No text could be extracted from this PDF."
                )

                st.stop()

            st.success(
                "PDF text extracted successfully."
            )

        except Exception as error:

            st.error(
                f"Could not process PDF: {error}"
            )

            st.stop()

    # --------------------------------------------------------
    # Preview
    # --------------------------------------------------------

    st.subheader(
        "Dossier Preview"
    )

    st.text_area(
        "Extracted text",
        dossier_text[:12000],
        height=250,
    )

    # --------------------------------------------------------
    # Run CTD analysis
    # --------------------------------------------------------

    if st.button(
        "📊 Analyze CTD Readiness",
        type="primary",
        use_container_width=True,
    ):

        try:

            module_results, gaps, overall_score = (
                check_ctd(dossier_text)
            )

            st.session_state[
                "ctd_module_results"
            ] = module_results

            st.session_state[
                "ctd_gaps"
            ] = gaps

            st.session_state[
                "ctd_overall"
            ] = overall_score

        except Exception as error:

            st.error(
                f"CTD analysis failed: {error}"
            )

    # --------------------------------------------------------
    # RESULTS
    # --------------------------------------------------------

    if "ctd_module_results" in st.session_state:

        module_results = st.session_state[
            "ctd_module_results"
        ]

        gaps = st.session_state[
            "ctd_gaps"
        ]

        overall_score = st.session_state[
            "ctd_overall"
        ]

        st.divider()

        st.subheader(
            "📊 Submission Readiness"
        )

        c1, c2, c3, c4 = st.columns(4)

        c1.metric(
            "Overall Score",
            f"{overall_score}%",
        )

        c2.metric(
            "Modules",
            5,
        )

        c3.metric(
            "Missing Sections",
            len(gaps),
        )

        critical_count = (
            len(
                gaps[
                    gaps["Severity"]
                    == "CRITICAL"
                ]
            )
            if not gaps.empty
            else 0
        )

        c4.metric(
            "Critical Gaps",
            critical_count,
        )

        # ----------------------------------------------------
        # Overall progress
        # ----------------------------------------------------

        st.progress(
            min(
                overall_score / 100,
                1.0,
            )
        )

        # ----------------------------------------------------
        # Module table
        # ----------------------------------------------------

        st.subheader(
            "CTD Module Assessment"
        )

        st.dataframe(
            module_results,
            use_container_width=True,
            hide_index=True,
        )

        # ----------------------------------------------------
        # Module chart
        # ----------------------------------------------------

        fig = px.bar(
            module_results,
            x="Module",
            y="Score",
            color="Status",
            text="Score",
            range_y=[0, 100],
            title="CTD Module Completeness",
            color_discrete_map={
                "READY": "#12B76A",
                "REVIEW": "#F2C94C",
                "ACTION REQUIRED": "#D92D20",
            },
        )

        fig.update_traces(
            texttemplate="%{text}%",
            textposition="outside",
        )

        st.plotly_chart(
            fig,
            use_container_width=True,
        )

        # ----------------------------------------------------
        # Gap report
        # ----------------------------------------------------

        st.subheader(
            "🚨 Regulatory Gap Report"
        )

        if gaps.empty:

            st.success(
                "No missing sections were detected."
            )

        else:

            critical = gaps[
                gaps["Severity"]
                == "CRITICAL"
            ]

            major = gaps[
                gaps["Severity"]
                == "MAJOR"
            ]

            c1, c2 = st.columns(2)

            c1.metric(
                "Critical",
                len(critical),
            )

            c2.metric(
                "Major",
                len(major),
            )

            st.dataframe(
                gaps,
                use_container_width=True,
                hide_index=True,
            )

            csv = gaps.to_csv(
                index=False
            ).encode("utf-8")

            st.download_button(
                label="⬇️ Download Gap Report",
                data=csv,
                file_name="pharmaguard_ctd_gap_report.csv",
                mime="text/csv",
                use_container_width=True,
            )

        st.warning(
            """
            Regulatory note: This automated checker
            provides a completeness aid. Module 1 is
            region-specific, and final regulatory
            submission readiness requires qualified
            regulatory review.
            """
        )

# ============================================================
# FOOTER
# ============================================================

st.markdown(
    """
    <div style="
        margin-top: 35px;
        padding: 18px;
        text-align: center;
        color: #64748b;
        font-size: 12px;
        border-top: 1px solid #e2e8f0;
    ">
        <b style="color:#17476b;">PharmaGuard AI</b>
        &nbsp;•&nbsp; AI-assisted decision support prototype
        &nbsp;•&nbsp; Human expert review remains essential
    </div>
    """,
    unsafe_allow_html=True,
)

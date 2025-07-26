import streamlit as st
import pandas as pd
import plotly.express as px
import requests
import io
import uuid

st.set_page_config(page_title="🏋️‍♂️ Your AI Gym Coach", layout="wide")

st.title("🏋️‍♂️ Your AI-Powered Personal Fitness Coach")

st.markdown("""
<style>
@keyframes pulse {
  0% { box-shadow: 0 0 0 0 rgba(255, 100, 100, 0.7); }
  70% { box-shadow: 0 0 0 20px rgba(255, 100, 100, 0); }
  100% { box-shadow: 0 0 0 0 rgba(255, 100, 100, 0); }
}
.info-box {
  background-color: #111;
  color: white;
  padding: 20px;
  border-radius: 10px;
  border-left: 5px solid #ff4d4d;
  font-size: 24px;
  animation: pulse 2s infinite;
}
</style>

#### 💪 Smarter Workouts. Stronger Body. Faster Results.
Upload your workout log and let your AI coach analyze progress, generate custom feedback, and forecast your fitness future.
""", unsafe_allow_html=True)

OPENROUTER_API_KEY = st.secrets["openrouter"]["api_key"]

def compute_fitness_metrics(df):
    df["BMI"] = df["Weight (kg)"] / ((df["Height (cm)"]/100) ** 2)
    df["Strength Index"] = df["1RM Bench Press"] + df["1RM Squat"] + df["1RM Deadlift"]
    df["Progress Score"] = df["Weight (kg)"] - df["Fat (%)"]
    return df

def ai_fitness_commentary(df):
    latest = df.iloc[-1]
    prompt = f"""You are a certified personal trainer. Analyze the latest fitness metrics:
- Weight: {latest['Weight (kg)']} kg
- Height: {latest['Height (cm)']} cm
- BMI: {latest['BMI']}
- Fat %: {latest['Fat (%)']}
- Bench Press: {latest['1RM Bench Press']} kg
- Squat: {latest['1RM Squat']} kg
- Deadlift: {latest['1RM Deadlift']} kg

Give insights, workout suggestions, and nutrition tips."""

    headers = {"Authorization": f"Bearer {OPENROUTER_API_KEY}", "Content-Type": "application/json"}
    payload = {"model": "deepseek/deepseek-r1:free", "messages": [{"role": "user", "content": prompt}]}

    try:
        response = requests.post("https://openrouter.ai/api/v1/chat/completions", headers=headers, json=payload)
        return response.json()["choices"][0]["message"]["content"]
    except Exception as e:
        return f"❌ AI error: {e}"

uploaded_file = st.file_uploader("📂 Upload Your Workout Log (CSV or Excel)", type=["xlsx", "csv"])

if uploaded_file:
    df = pd.read_excel(uploaded_file) if uploaded_file.name.endswith(".xlsx") else pd.read_csv(uploaded_file)
    df = compute_fitness_metrics(df)

    user_name = uploaded_file.name.replace(".xlsx", "").replace(".csv", "")
    st.markdown(f"<div class='info-box'>👤 Detected User: <b>{user_name}</b></div>", unsafe_allow_html=True)

    st.subheader("📅 Workout Data")
    st.dataframe(df)

    st.subheader("📊 Progress Over Time")
    st.plotly_chart(px.line(df, x="Date", y=["Weight (kg)", "Fat (%)", "BMI"], markers=True), use_container_width=True)

    st.subheader("🏋 Strength Progress")
    st.plotly_chart(px.bar(df, x="Date", y=["1RM Bench Press", "1RM Squat", "1RM Deadlift"], barmode="group"), use_container_width=True)

    st.subheader("💬 Coach Commentary")
    with st.spinner("Analyzing your progress..."):
        st.markdown(ai_fitness_commentary(df))

    st.subheader("🧐 Ask Your Fitness Coach")
    user_q = st.text_input("Ask a question about your fitness metrics or goals:")

    if user_q:
        prompt = f"User Question: {user_q}\n\nRecent Data:\n{df.tail(5).to_string(index=False)}"
        headers = {"Authorization": f"Bearer {OPENROUTER_API_KEY}", "Content-Type": "application/json"}
        payload = {"model": "deepseek/deepseek-r1:free", "messages": [{"role": "user", "content": prompt}]}
        try:
            response = requests.post("https://openrouter.ai/api/v1/chat/completions", headers=headers, json=payload)
            answer = response.json()["choices"][0]["message"]["content"]
            st.success(answer)
        except Exception as e:
            st.error(f"AI error: {e}")

    st.subheader("🧠 AI Chat Coach")
    user_id = str(uuid.uuid4())
    config_url = "https://files.bpcontent.cloud/2025/07/02/02/20250702020605-VDMFG1YB.json"
    iframe_url = f"https://cdn.botpress.cloud/webchat/v3.0/shareable.html?configUrl={config_url}&userId={user_id}"

    st.markdown(f"""<iframe src=\"{iframe_url}\" width=\"100%\" height=\"600\" style=\"border:none;\" allow=\"microphone\"></iframe>""", unsafe_allow_html=True)

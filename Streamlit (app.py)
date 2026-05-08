import streamlit as st
import pandas as pd
from openai import OpenAI
import json

# 1. Configuration & Setup
st.set_page_config(page_title="AI Test Case Generator", layout="wide")
st.title("🧪 IS Project: AI-Powered Test Case Optimizer")
st.write("นิสิต: คุณสุพรรษา พวงมณี (Digital Analytics & Data Science)")

# เชื่อมต่อ OpenAI API (ใส่ Key ของคุณที่นี่)
client = OpenAI(api_key="YOUR_OPENAI_API_KEY")

# 2. Load Data
@st.cache_data
def load_data():
    df = pd.read_csv('functional_requirements.csv')
    return df

df_f = load_data()

# 3. UI Layout - Sidebar & List View
st.sidebar.header("Filter & Settings")
st.sidebar.write(f"พบ Functional Requirements: {len(df_f)} รายการ")

# ส่วนแสดงรายการ Requirement ทั้งหมด
st.subheader("📋 Functional Requirements List")
st.dataframe(df_f[['ID', 'Type', 'Requirement']], use_container_width=True)

# 4. Input Box สำหรับเลือก ID ที่ต้องการ Generate
st.divider()
st.subheader("🤖 Generate Test Cases from Requirement")
target_id = st.number_input("ใส่เลข ID จากคอลัมน์ ID ที่ต้องการวิเคราะห์:", 
                             min_value=int(df_f['ID'].min()), 
                             max_value=int(df_f['ID'].max()), 
                             step=1)

# ดึงข้อความ Requirement จาก ID ที่เลือก
selected_req = df_f[df_f['ID'] == target_id]

if not selected_req.empty:
    req_text = selected_req['Requirement'].values[0]
    st.info(f"**Requirement ที่เลือก:** {req_text}")
    
    if st.button("Generate Test Cases ✨"):
        with st.spinner('GPT-4o กำลังวิเคราะห์และสร้าง Test Case...'):
            try:
                # 5. Prompt Engineering สำหรับสร้าง Test Case (1:n)
                prompt = f"""
                Context: You are an expert Software Tester.
                Task: Analyze the requirement below and generate 1 or more professional test cases.
                
                Requirement ID: {target_id}
                Requirement: "{req_text}"
                
                Guidelines:
                - Generate n test cases (Positive & Negative).
                - Each test case must have multiple detailed steps.
                - Provide a 'Coverage Confidence Score' (0-100%) for the created test cases.
                
                Output MUST be in JSON format:
                {{
                  "confidence_score": 00,
                  "explanation": "...",
                  "test_cases": [
                    {{
                      "name": "...",
                      "pre_condition": "...",
                      "steps": ["Step 1", "Step 2", "..."],
                      "expected_result": "..."
                    }}
                  ]
                }}
                """
                
                response = client.chat.completions.create(
                    model="gpt-4o-mini",
                    messages=[{"role": "system", "content": "You are a helpful assistant that outputs JSON."},
                              {"role": "user", "content": prompt}],
                    response_format={ "type": "json_object" }
                )
                
                result = json.loads(response.choices[0].message.content)
                
                # 6. แสดงผลบน UI
                col1, col2 = st.columns([1, 3])
                with col1:
                    st.metric("Coverage Confidence", f"{result['confidence_score']}%")
                with col2:
                    st.success(f"**เหตุผลประกอบ:** {result['explanation']}")
                
                for i, tc in enumerate(result['test_cases'], 1):
                    with st.expander(f"Test Case {i}: {tc['name']}", expanded=True):
                        st.write(f"**Pre-condition:** {tc['pre_condition']}")
                        st.write("**Steps:**")
                        for step in tc['steps']:
                            st.write(f"- {step}")
                        st.write(f"**Expected Result:** {tc['expected_result']}")
                        
            except Exception as e:
                st.error(f"เกิดข้อผิดพลาด: {e}")
else:
    st.warning("ไม่พบ ID นี้ในระบบ กรุณาตรวจสอบเลข ID อีกครั้ง")

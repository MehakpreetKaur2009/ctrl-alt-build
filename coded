import os
import streamlit as st
from google import genai
from google.genai import types
from google.genai.errors import ServerError, ClientError

# Set browser window configurations
st.set_page_config(page_title="Ctrl+Alt+Build Workspace", page_icon="🛠️", layout="wide")

# Securely fetch API key from Streamlit Secrets or Environment Variables
GOOGLE_API_KEY = st.secrets.get("GEMINI_API_KEY", os.environ.get("GEMINI_API_KEY"))

# Initialize Google Gemini Client
if not GOOGLE_API_KEY:
    st.error("⚠️ API Key missing! Please set GEMINI_API_KEY in Streamlit Secrets.")
else:
    try:
        client = genai.Client(api_key=GOOGLE_API_KEY)
    except Exception as e:
        st.error(f"API Initialization failed: {str(e)}")

# Define the precise, blunt mentor personality rules
MENTOR_PERSONALITY = (
    "You are a blunt, elite tech mentor for a brilliant Class XI student. "
    "Your job is to teach business, front-end/back-end coding, marketing, hackathon strategy, hardware engineering (drones, RC cars), and robotics. "
    "CRITICAL RULES:\n"
    "1. No fluff, no polite filler text.\n"
    "2. Be direct, direct-to-the-point, and act like a dedicated study buddy.\n"
    "3. When explaining hardware like drones or cars, always explain the underlying science, physics, and code logic behind how it actually works.\n"
    "4. Break complex concepts into clear, raw components."
)

# ----------------- SYSTEM MEMORY ENGINE -----------------
if "all_chats" not in st.session_state:
    st.session_state.all_chats = {}

if "current_chat_id" not in st.session_state:
    st.session_state.current_chat_id = "Default Thread"

if st.session_state.current_chat_id not in st.session_state.all_chats:
    st.session_state.all_chats[st.session_state.current_chat_id] = []

active_messages = st.session_state.all_chats[st.session_state.current_chat_id]


# ----------------- SIDEBAR INTERFACE LAYOUT -----------------
with st.sidebar:
    st.title("Ctrl+Alt+Build 🛠️")
    st.subheader("Navigation")
    
    if st.button("➕ New Chat Thread", use_container_width=True, type="primary"):
        new_id = f"Tech Session #{len(st.session_state.all_chats) + 1}"
        st.session_state.all_chats[new_id] = []
        st.session_state.current_chat_id = new_id
        st.rerun()
        
    st.divider()
    st.subheader("📚 Session History")
    
    if len(st.session_state.all_chats) <= 1 and len(active_messages) == 0:
        st.write("*No recent sessions stored*")
    else:
        for chat_title in list(st.session_state.all_chats.keys()):
            if st.button(f"💬 {chat_title}", key=f"btn_{chat_title}", use_container_width=True):
                st.session_state.current_chat_id = chat_title
                st.rerun()


# ----------------- MAIN SCREEN PORTAL -----------------
st.title(f"Portal: {st.session_state.current_chat_id}")
st.write("No fluff. No sugarcoating. Tell me what we are building or coding today.")

for message in active_messages:
    with st.chat_message(message["role"]):
        st.markdown(message["content"])

user_query = st.chat_input("What are we building today?")

if user_query:
    with st.chat_message("user"):
        st.markdown(user_query)
    active_messages.append({"role": "user", "content": user_query})

    if st.session_state.current_chat_id.startswith("Tech Session") and len(active_messages) == 1:
        clean_title = user_query[:20] + "..." if len(user_query) > 20 else user_query
        st.session_state.all_chats[clean_title] = st.session_state.all_chats.pop(st.session_state.current_chat_id)
        st.session_state.current_chat_id = clean_title
        st.rerun()

    with st.chat_message("assistant"):
        with st.spinner("Analyzing data vectors..."):
            try:
                response = client.models.generate_content(
                    model='gemini-2.5-flash',
                    contents=user_query,
                    config=types.GenerateContentConfig(
                        system_instruction=MENTOR_PERSONALITY
                    )
                )
                bot_reply = response.text
            except ServerError:
                bot_reply = "**[Google Server Surge Fallback]** Google's servers are experiencing high traffic. Please query again in a moment."
            except ClientError as ce:
                if "401" in str(ce):
                    bot_reply = "❌ **Authentication Failed:** Your Google API Key string is incorrect. Reset your key configuration in Streamlit Secrets."
                else:
                    bot_reply = f"❌ **Client Error Encountered:** {str(ce)}"
            except Exception as e:
                bot_reply = f"An unexpected system exception occurred: {str(e)}"
                    
            st.markdown(bot_reply)
            
    active_messages.append({"role": "assistant", "content": bot_reply})

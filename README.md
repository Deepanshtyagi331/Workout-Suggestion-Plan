import streamlit as st
import streamlit as st
import pandas as pd
import numpy as np
from datetime import datetime, timedelta, date
import plotly.express as px
import plotly.graph_objects as go
from dataclasses import dataclass
from typing import List, Dict, Optional
from enum import Enum

# Import our custom modules
from models import *
from exercise_database import exercise_db
from plan_generators import plan_generator
from ui_components import *
from beginner_features import *
from advanced_features import *

# Page configuration
st.set_page_config(
    page_title="Workout Suggestion Plan",
    page_icon="💪",
    layout="wide",
    initial_sidebar_state="expanded"
)

# Custom CSS for better styling
st.markdown("""
<style>
    .main-header {
        font-size: 3rem;
        font-weight: bold;
        text-align: center;
        color: #ff6b35;
        margin-bottom: 2rem;
    }
    .section-header {
        font-size: 1.5rem;
        font-weight: bold;
        color: #2e86ab;
        margin: 1rem 0;
    }
    .workout-card {
        background-color: #f0f2f6;
        padding: 1rem;
        border-radius: 10px;
        margin: 0.5rem 0;
        border-left: 4px solid #ff6b35;
    }
    .metric-container {
        background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
        padding: 1rem;
        border-radius: 10px;
        color: white;
        text-align: center;
    }
</style>
""", unsafe_allow_html=True)

def main():
    st.markdown('<h1 class="main-header">💪 Workout Suggestion Plan</h1>', unsafe_allow_html=True)
    st.markdown("**Your Personal Fitness Journey Starts Here!**")
    
    # Initialize session state
    if 'user_profile' not in st.session_state:
        st.session_state.user_profile = None
    if 'current_plan' not in st.session_state:
        st.session_state.current_plan = None
    if 'saved_plans' not in st.session_state:
        st.session_state.saved_plans = []
    if 'user_goals' not in st.session_state:
        st.session_state.user_goals = []
    if 'progress_entries' not in st.session_state:
        st.session_state.progress_entries = []

    # Sidebar navigation
    with st.sidebar:
        st.image("https://via.placeholder.com/200x100/ff6b35/ffffff?text=FITNESS+APP", width=200)
        
        menu_option = st.selectbox(
            "🧭 Navigation",
            ["🏠 Home", "👤 User Profile", "🌟 Beginner Guide", "📋 Daily Plans", 
             "📅 Weekly Plans", "📆 Monthly Plans", "🎯 Goal Tracking", 
             "📊 Progress Analytics", "🔧 Advanced Tools", "🍎 Nutrition Tracker", 
             "ℹ️ Exercise Guide"]
        )

    if menu_option == "🏠 Home":
        show_home_page()
    elif menu_option == "👤 User Profile":
        show_user_profile()
    elif menu_option == "🌟 Beginner Guide":
        show_beginner_guide()
    elif menu_option == "📋 Daily Plans":
        show_daily_plans()
    elif menu_option == "📅 Weekly Plans":
        show_weekly_plans()
    elif menu_option == "📆 Monthly Plans":
        show_monthly_plans()
    elif menu_option == "🎯 Goal Tracking":
        show_goal_tracking()
    elif menu_option == "📊 Progress Analytics":
        show_progress_analytics()
    elif menu_option == "🔧 Advanced Tools":
        show_advanced_tools()
    elif menu_option == "🍎 Nutrition Tracker":
        show_nutrition_tracker()
    elif menu_option == "ℹ️ Exercise Guide":
        show_exercise_guide()

def show_home_page():
    st.markdown('<h2 class="section-header">Welcome to Your Fitness Journey! 🌟</h2>', unsafe_allow_html=True)
    
    col1, col2, col3 = st.columns(3)
    
    with col1:
        st.markdown("""
        <div class="workout-card">
            <h3>🎯 Set Your Goals</h3>
            <p>Define your fitness objectives and let our AI create personalized workout plans tailored to your needs.</p>
        </div>
        """, unsafe_allow_html=True)
    
    with col2:
        st.markdown("""
        <div class="workout-card">
            <h3>📈 Track Progress</h3>
            <p>Monitor your improvements with detailed analytics and progress tracking features.</p>
        </div>
        """, unsafe_allow_html=True)
    
    with col3:
        st.markdown("""
        <div class="workout-card">
            <h3>💪 Stay Motivated</h3>
            <p>Get daily, weekly, and monthly plans that adapt to your fitness level and preferences.</p>
        </div>
        """, unsafe_allow_html=True)
    
    st.markdown("---")
    
    if st.button("🚀 Get Started with Your Workout Plan"):
        st.success("Navigate to 'User Profile' to begin your fitness journey!")

def show_user_profile():
    st.markdown('<h2 class="section-header">👤 Create Your Profile</h2>', unsafe_allow_html=True)
    
    with st.form("user_profile_form"):
        col1, col2 = st.columns(2)
        
        with col1:
            name = st.text_input("Full Name", placeholder="Enter your name")
            age = st.number_input("Age", min_value=13, max_value=100, value=25)
            weight = st.number_input("Weight (kg)", min_value=30, max_value=200, value=70)
            height = st.number_input("Height (cm)", min_value=120, max_value=220, value=170)
        
        with col2:
            fitness_level = st.selectbox(
                "Current Fitness Level",
                ["Beginner", "Intermediate", "Advanced", "Professional"]
            )
            primary_goal = st.selectbox(
                "Primary Goal",
                ["Weight Loss", "Muscle Building", "Strength Training", "Endurance", "General Fitness", "Sports Performance"]
            )
            workout_days = st.slider("Preferred Workout Days per Week", 1, 7, 4)
            session_duration = st.slider("Preferred Session Duration (minutes)", 15, 120, 45)
        
        available_equipment = st.multiselect(
            "Available Equipment",
            ["Dumbbells", "Barbell", "Resistance Bands", "Pull-up Bar", "Kettlebells", 
             "Gym Access", "Cardio Machines", "Yoga Mat", "None (Bodyweight only)"]
        )
        
        medical_conditions = st.text_area(
            "Medical Conditions or Limitations",
            placeholder="Any injuries or conditions we should consider..."
        )
        
        submitted = st.form_submit_button("💾 Save Profile")
        
        if submitted and name and age and weight and height:
            try:
                profile_data = {
                    "name": name,
                    "age": age,
                    "weight": weight,
                    "height": height,
                    "fitness_level": fitness_level,
                    "primary_goal": primary_goal,
                    "workout_days": workout_days,
                    "session_duration": session_duration,
                    "available_equipment": available_equipment,
                    "medical_conditions": medical_conditions
                }
                
                # Validate and create UserProfile object
                user_profile = validate_user_profile(profile_data)
                st.session_state.user_profile = user_profile
                st.success("✅ Profile saved successfully!")
                
                # Display profile summary
                col1, col2, col3 = st.columns(3)
                with col1:
                    st.metric("BMI", user_profile.bmi)
                with col2:
                    st.metric("BMI Category", user_profile.bmi_category)
                with col3:
                    st.metric("Fitness Level", user_profile.fitness_level.value)
                
                st.balloons()
            except ValueError as e:
                st.error(f"❌ Error saving profile: {e}")
        elif submitted:
            st.error("❌ Please fill in all required fields!")

def show_daily_plans():
    st.markdown('<h2 class="section-header">📋 Daily Workout Plans</h2>', unsafe_allow_html=True)
    
    if not st.session_state.user_profile:
        st.warning("⚠️ Please create your user profile first!")
        return
    
    # Daily plan selector
    col1, col2, col3 = st.columns(3)
    with col1:
        selected_date = st.date_input("Select Date", date.today())
    with col2:
        focus_area = st.selectbox("Focus Area", 
                                ["Full Body", "Upper Body", "Lower Body", "Core", "Cardio", "Flexibility"])
    with col3:
        intensity = st.selectbox("Intensity Level", ["Low", "Moderate", "High", "Very High"])
    
    if st.button("🎯 Generate Daily Plan"):
        try:
            daily_plan = plan_generator.generate_daily_plan(
                user_profile=st.session_state.user_profile,
                focus_area=focus_area,
                intensity=intensity,
                target_date=selected_date
            )
            display_daily_plan(daily_plan, selected_date)
        except Exception as e:
            st.error(f"❌ Error generating plan: {e}")

def show_weekly_plans():
    st.markdown('<h2 class="section-header">📅 Weekly Workout Plans</h2>', unsafe_allow_html=True)
    
    if not st.session_state.user_profile:
        st.warning("⚠️ Please create your user profile first!")
        return
    
    # Weekly plan options
    col1, col2 = st.columns(2)
    with col1:
        week_start = st.date_input("Week Starting Date", date.today())
        plan_type = st.selectbox("Weekly Plan Type", 
                               ["Balanced Training", "Strength Focus", "Cardio Focus", "Flexibility Focus"])
    with col2:
        split_type = st.selectbox("Training Split", 
                                ["Push/Pull/Legs", "Upper/Lower", "Full Body", "Body Part Split"])
        progressive = st.checkbox("Progressive Overload", value=True)
    
    if st.button("📅 Generate Weekly Plan"):
        weekly_plan = generate_weekly_plan(st.session_state.user_profile, plan_type, split_type)
        display_weekly_plan(weekly_plan, week_start)

def show_monthly_plans():
    st.markdown('<h2 class="section-header">📆 Monthly Workout Plans</h2>', unsafe_allow_html=True)
    
    if not st.session_state.user_profile:
        st.warning("⚠️ Please create your user profile first!")
        return
    
    # Monthly plan configuration
    col1, col2 = st.columns(2)
    with col1:
        month_year = st.selectbox("Select Month", 
                                ["January", "February", "March", "April", "May", "June",
                                 "July", "August", "September", "October", "November", "December"])
        year = st.number_input("Year", min_value=2024, max_value=2030, value=2024)
    
    with col2:
        program_type = st.selectbox("Program Type",
                                  ["Beginner Foundation", "Strength Building", "Fat Loss", 
                                   "Muscle Building", "Athletic Performance", "Maintenance"])
        periodization = st.checkbox("Include Periodization", value=True)
    
    if st.button("📆 Generate Monthly Plan"):
        monthly_plan = generate_monthly_plan(st.session_state.user_profile, program_type)
        display_monthly_plan(monthly_plan, month_year, year)

def show_beginner_guide():
    st.markdown('<h2 class="section-header">🌟 Beginner Fitness Guide</h2>', unsafe_allow_html=True)
    
    # Check if user has completed beginner assessment
    if not hasattr(st.session_state, 'beginner_program_active'):
        st.markdown("### 👋 Welcome to Fitness!")
        st.markdown("*New to working out? Let's create a safe and effective program just for you.*")
        
        create_beginner_onboarding()
    else:
        # Show beginner dashboard
        display_beginner_dashboard()
    
    # Always show tips in sidebar
    create_beginner_tips_sidebar()

def show_advanced_tools():
    st.markdown('<h2 class="section-header">🔧 Advanced Tools</h2>', unsafe_allow_html=True)
    
    if not st.session_state.user_profile:
        st.warning("⚠️ Please create your user profile first!")
        return
    
    # Use selectbox instead of tabs for compatibility
    tool_choice = st.selectbox(
        "Select Tool",
        ["📊 Analytics Dashboard", "⚙️ Workout Customizer"]
    )
    
    if tool_choice == "📊 Analytics Dashboard":
        create_advanced_dashboard()
    elif tool_choice == "⚙️ Workout Customizer":
        create_workout_customizer()

def show_nutrition_tracker():
    st.markdown('<h2 class="section-header">🍎 Nutrition Tracker</h2>', unsafe_allow_html=True)
    
    if not st.session_state.user_profile:
        st.warning("⚠️ Please create your user profile first!")
        return
    
    create_nutrition_tracker()
    st.markdown('<h2 class="section-header">🎯 Goal Tracking</h2>', unsafe_allow_html=True)
    
    if not st.session_state.user_profile:
        st.warning("⚠️ Please create your user profile first!")
        return
    
    create_goal_tracking_interface()

def show_goal_tracking():
    st.markdown('<h2 class="section-header">🎯 Goal Tracking</h2>', unsafe_allow_html=True)
    
    if not st.session_state.user_profile:
        st.warning("⚠️ Please create your user profile first!")
        return
    
    create_goal_tracking_interface()

def show_progress_analytics():
    st.markdown('<h2 class="section-header">📊 Progress Analytics</h2>', unsafe_allow_html=True)
    
    if not st.session_state.user_profile:
        st.warning("⚠️ Please create your user profile first!")
        return
    
    create_progress_analytics()

def show_exercise_guide():
    st.markdown('<h2 class="section-header">ℹ️ Exercise Guide</h2>', unsafe_allow_html=True)
    
    create_exercise_guide()

if __name__ == "__main__":
    main()

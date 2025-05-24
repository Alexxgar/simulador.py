import streamlit as st 
import plotly.graph_objects as go
import pandas as pd
import numpy as np
import time
from datetime import datetime

st.set_page_config(layout="wide", page_title="Simulador de Estrés")
st.title("Simulador de Estrés")

# Inicialización de estado
if "history" not in st.session_state:
    st.session_state.history = []
if "pre" not in st.session_state:
    st.session_state.pre = None
if "post" not in st.session_state:
    st.session_state.post = None
if "auto_mode" not in st.session_state:
    st.session_state.auto_mode = False
if "live_data" not in st.session_state:
    st.session_state.live_data = []

# Niveles y recomendaciones
levels = {
    "Normal": (50, 70),
    "Leve": (71, 90),
    "Moderado": (91, 110),
    "Severo": (111, 140)
}

recommendations = {
    "Normal": "¡Todo va bien! Mantén la calma.",
    "Leve": "Ligero estrés. Tómate un descanso.",
    "Moderado": "Estrés moderado. Respira profundo.",
    "Severo": "¡Alerta! Necesitas calmarte ya."
}

# Panel lateral
st.sidebar.header("Configuración")
nivel = st.sidebar.selectbox("Selecciona nivel de estrés", list(levels.keys()))
auto = st.sidebar.toggle("Modo automático", value=st.session_state.auto_mode)

# Simular frecuencia cardíaca
min_hr, max_hr = levels[nivel]
current_hr = np.random.randint(min_hr, max_hr+1)

# Registrar en historial
now = datetime.now().strftime("%Y-%m-%d %H:%M:%S")
if st.button("Registrar frecuencia"):
    st.session_state.history.append({"Hora": now, "Nivel": nivel, "BPM": current_hr})
    st.success(f"Registrado: {current_hr} BPM")

# Modo automático
if auto and not st.session_state.auto_mode:
    st.session_state.auto_mode = True
    st.session_state.live_data = []

elif not auto:
    st.session_state.auto_mode = False

# Mostrar frecuencia actual
st.metric("Frecuencia Cardíaca Actual", f"{current_hr} BPM")
st.info(recommendations[nivel])

# Gráfica en vivo
if st.session_state.auto_mode:
    st.session_state.live_data.append(current_hr)
    fig_live = go.Figure()
    fig_live.add_trace(go.Scatter(y=st.session_state.live_data[-60:], mode='lines+markers', name='BPM'))
    fig_live.update_layout(title="Frecuencia en Tiempo Real", yaxis=dict(range=[50, 150]))
    st.plotly_chart(fig_live, use_container_width=True)
    time.sleep(1)
    st.experimental_rerun()

# Tabla de historial
st.subheader("Historial de Frecuencia")
if st.session_state.history:
    df = pd.DataFrame(st.session_state.history)
    st.dataframe(df)

    col1, col2 = st.columns(2)
    with col1:
        if st.button("Exportar CSV"):
            st.download_button("Descargar", df.to_csv(index=False), file_name="historial.csv")
    with col2:
        if st.button("Exportar JSON"):
            st.download_button("Descargar", df.to_json(orient="records"), file_name="historial.json")

# Comparación pre/post relajación
st.subheader("Comparación Pre/Post Relajación")
col1, col2 = st.columns(2)
with col1:
    if st.button("Registrar Pre-Relajación"):
        st.session_state.pre = current_hr
with col2:
    if st.button("Registrar Post-Relajación"):
        st.session_state.post = current_hr

if st.session_state.pre is not None and st.session_state.post is not None:
    st.success(f"Pre: {st.session_state.pre} BPM | Post: {st.session_state.post} BPM")
    diff = st.session_state.pre - st.session_state.post
    fig_bar = go.Figure(data=[
        go.Bar(name="Pre", y=[st.session_state.pre]),
        go.Bar(name="Post", y=[st.session_state.post])
    ])
    fig_bar.update_layout(title="Comparación", barmode='group', yaxis=dict(title="BPM", range=[50, 150]))
    st.plotly_chart(fig_bar)

# Streamlit & Gradio - ML App Interfaces

## Descripción
Streamlit y Gradio: crear web apps ML sin JavaScript. Streamlit: dashboards interactivos, deployment fácil. Gradio: demos ML rápidos, Hugging Face integration. Casos: prototipos, demos clientes, MVPs. Streamlit Cloud hosting gratuito.

## Streamlit
```python
import streamlit as st
st.title("ML App")
uploaded_file = st.file_uploader("Upload image")
if st.button("Predict"):
    prediction = model.predict(data)
    st.write(f"Result: {prediction}")
```

## Gradio
```python
import gradio as gr
def predict(image):
    return model(image)
gr.Interface(fn=predict, inputs="image", outputs="label").launch()
```
---
**Tiempo**: 1 semana

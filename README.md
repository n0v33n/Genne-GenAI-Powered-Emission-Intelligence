# 🌍 GennE: Generative AI-Powered Emission Intelligence

[![Python Version](https://img.shields.io/badge/python-3.8%2B-blue)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/license-MIT-green)](LICENSE)
[![Google Gemini](https://img.shields.io/badge/LLM-Google%20Gemini-orange)](https://ai.google.dev/)
[![OpenCLIP](https://img.shields.io/badge/Vision-OpenCLIP-purple)](https://github.com/mlfoundations/open_clip)

> Environmental emissions data is often scattered across multiple sources and presented in **unstructured formats** like PDFs, making it difficult to extract, analyze, and act upon.

GennE is an innovative application leveraging generative AI to extract, analyze, and visualize environmental emission data from unstructured sources. The system seamlessly combines natural language processing, computer vision, and geospatial visualization to provide actionable insights into industrial emissions.

![Project Overview](https://www.googleapis.com/download/storage/v1/b/kaggle-forum-message-attachments/o/inbox%2F14469442%2F3c5eb5ad89393d37172063f2c9549e9b%2Fmindmap.png?generation=1744830398331864&alt=media)

## Table of Contents

- [Features](##-features)
- [System Architecture](##-system-architecture)
- [Prerequisites](##-prerequisites)
- [Installation](##-installation)
- [Usage](##-usage)
- [Key Components](##-key-components)
- [Demo](##-demo)
- [Authors](##-authors)
- [Acknowledgements](##-acknowledgements)

## Features

- **PDF Data Extraction**: Automatically extract emissions data from unstructured PDF reports
- **Structured Data Generation**: Convert unstructured text to structured JSON/CSV formats using Google Gemini
- **Geospatial Visualization**: Map emission data using coordinates lookup and interactive Folium maps
- **Visual Intelligence**: Interpret map visualizations using OpenCLIP model
- **Conversational Agent**: Query emission data using natural language via LangChain + Gemini
- **Interactive UI**: Explore emission data through a user-friendly Gradio interface

## System Architecture

The system employs a multi-stage pipeline:

1. **PDF Processing**: Extract text from emission reports
2. **Structured Data Generation**: Use Google Gemini to parse text into structured emission data
3. **Geospatial Mapping**:
   - Encode city names using SentenceTransformer
   - Find coordinates using FAISS vector search
   - Generate interactive maps with Folium
4. **Visual Analysis**: Interpret maps using OpenCLIP visual-semantic model
5. **Agent Integration**: Enable natural language queries via LangChain agent with specialized tools
6. **User Interface**: Provide access through a Gradio-based chat interface

## Prerequisites

- Python 3.8+
- Google API key with access to Gemini models
- Kaggle account (for accessing datasets and secrets)

## Installation

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/genne.git
   cd genne
   ```

2. Install dependencies:
   ```bash
   pip install -r requirements.txt
   ```

3. Set up your Google API key:
   ```python
   # Use environment variables
   export GOOGLE_API_KEY="your_api_key_here"
   
   # Or in Kaggle notebooks
   from kaggle_secrets import UserSecretsClient
   GOOGLE_API_KEY = UserSecretsClient().get_secret("GOOGLE_API_KEY")
   ```

## Usage

### Running the complete pipeline

```python
# Import necessary modules
from genne import load_pdf, extract_structured_data, generate_map, interpret_map, create_agent

# Process PDF and extract structured data
emission_data = extract_structured_data(load_pdf("path/to/emission_report.pdf"))

# Generate and visualize emission map
emission_map = generate_map(emission_data)

# Interpret map contents using CLIP
map_interpretation = interpret_map(emission_map)

# Create interactive agent for querying
agent = create_agent(emission_data, emission_map)

# Launch Gradio interface
launch_interface(agent)
```

### Using the Agent Interface

The conversational agent can answer questions about emissions data:

- "What are the CO₂ emissions in [city]?"
- "Where is [city] located?"
- "Interpret the emission map"
- "Compare emissions between [city1] and [city2]"

## Key Components

### PDF Processing
```python
# Load and process PDF
loader = PyPDFLoader("path/to/emission_report.pdf")
pages = loader.load()
```

### Structured Data Generation with Gemini
```python
response = client.models.generate_content(
    model='gemini-2.0-flash',
    config=types.GenerateContentConfig(
        temperature=0.1,
        response_mime_type="application/json",
        response_schema=emission_report,
    ),
    contents=[text_line]
)
```

### City Coordinate Lookup
```python
def search_city(query: str, top_k: int = 1):
    query_vec = model.encode([query])
    D, I = index.search(np.array(query_vec), k=top_k)
    return {
        metadata.loc[i, "name"]: [metadata.loc[i, "latitude"], metadata.loc[i, "longitude"]] 
        for i in I[0]
    }
```

### Interactive Map Generation
```python
m = folium.Map(location=[22.9734, 78.6569], zoom_start=5)
marker_cluster = MarkerCluster().add_to(m)

for _, row in data.iterrows():
    if pd.notnull(row["latitude"]) and pd.notnull(row["longitude"]):
        folium.Marker(
            location=[row["latitude"], row["longitude"]],
            popup=f"City: {row['location']}<br>Year: {row['year']}<br>Emissions: {row['emissions_mtco2e']} MtCO₂e<br>Source: {row['source']}"
        ).add_to(marker_cluster)
```

### Vision-Language Understanding
```python
model, _, preprocess = open_clip.create_model_and_transforms('ViT-B-32', pretrained='laion2b_s34b_b79k')
image_features = model.encode_image(image_tensor)
text_features = model.encode_text(text_tokens)
```

### LangChain Agent Tools
```python
tools = [
    Tool.from_function(func=get_emission_by_city, name="EmissionFactTool", 
                      description="Returns CO₂ emissions for a city."),
    Tool.from_function(func=get_coordinates, name="CityCoordinateTool", 
                      description="Returns city coordinates."),
    Tool.from_function(func=interpret_emission_map, name="MapInsightTool", 
                      description="Explains the emission map.")
]
```

## Demo

![Emission Map](https://www.kaggleusercontent.com/kf/234882243/eyJhbGciOiJkaXIiLCJlbmMiOiJBMTI4Q0JDLUhTMjU2In0..A6UBDwTG3oUp3n8aKr9mrw.29VZdW2Qp81A0_97iO0826UgYv2Kt7C55Ieam-v0qHCOt7Ndz6Xzolzm-fxcx6CXfdTnssdL7j8wFl7holeyTM5YByf2lFBUgKoz5AaYhAC_M8ncPAbLl4utfRc4fxJImZhT7jgXcOksJ7kZLJn_LfzcdBLS0-QS1Xc7XRwEaKeADtyhe-pFSqdROI-oKJvjUBzCJj-R-sbGK1BwY_ZtpBhwomGdi49cMWpogevCZbe1SSi5PMeNp7OHf-HVb1jp8fjJQVA4MoDFPQds47JhRRTY89UCB7c3dZjEExybxuROt92AJyAASgvcj0wT0X6V4EMbdnj0CGsPgAUCIUSmVFTNaxi05qYTjuONSbVSxbdJnGzR5hXjlFghE1uNysuRzN7LQuONML6Y5INhVN8K4bpXT6J33T9gcOCut8l1Macz2vnWE8AMudT4bBWS03kVV3hsiv54zn2uQZmO5cKn3RkkGiWdTtpd3zd31JghFvdZfoaMMiqD17jID1CvxHcXHtOb2pw9eSq2YBbYiMzATKQkb8dDAaNCg5sgAYIcrUpRp1A7WineJX3UZVcO1B_cu7Be_P0eO6iEh0KuLQvezmZA3El9ZfYRqfXEYqLsGTtgpkB5nc_hrDhkv3BGor0G.lc-zuDaRN1u38_1-h3BhMg/__results___files/__results___89_0.png)

The interactive demo allows users to:
1. Query emission data for specific cities
2. Visualize geographic distribution of emissions
3. Understand relationships between data points
4. Gain insights through AI-powered analysis

## Authors

- [Samudrala Dinesh Naveen Kumar](https://www.kaggle.com/dnkumars)
- [Morpho23](https://www.kaggle.com/morpho23)
- [Samudrala Hareesh](https://www.kaggle.com/samudralahareesh)

## Acknowledgements

- [Kaggle](https://www.kaggle.com/) for the competition platform
- [Google](https://ai.google.dev/) for the Gemini API
- [GeoNames](https://www.geonames.org/) for the geographic database
- [OpenCLIP](https://github.com/mlfoundations/open_clip) for vision-language models
- [LangChain](https://www.langchain.com/) for agent framework

## Resources

- 📘 **Blog Post:** [Gen AI Capstone Insights – A Deep Dive](https://medium.com/@samudraladnkumar/gen-ai-powered-emission-intelligence-system-339ceaac8fc8)
- 📺 **YouTube Video:** [Watch the Capstone Project Overview](https://youtu.be/QSTWjqR_6A4)
- 📺 **Kaggle Notebook:** [Notebook Overview](https://www.kaggle.com/code/dnkumars/genne/)

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

<div align="center">
  <p>If you find this project useful, please consider giving it a ⭐!</p>
</div>

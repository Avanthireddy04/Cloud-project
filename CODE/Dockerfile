FROM python:3.9-slim

WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt

# Download spaCy models (if not using SageMaker)
RUN python -m spacy download en_core_web_sm
RUN python -m spacy download en_core_web_md

# Copy application code
COPY . .

EXPOSE 8501

HEALTHCHECK CMD curl --fail http://localhost:8501/_stcore/health

ENTRYPOINT ["streamlit", "run", "singleResume_app.py", "--server.port=8501", "--server.address=0.0.0.0"]

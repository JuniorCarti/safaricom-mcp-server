# Use Python 3.11 slim image for compatibility
FROM python:3.11-slim

# Set working directory
WORKDIR /app

# Install system dependencies
RUN apt-get update && apt-get install -y \
    gcc \
    g++ \
    && rm -rf /var/lib/apt/lists/*

# Copy requirements and install Python dependencies
COPY requirements.txt .
RUN pip install --no-cache-dir --upgrade pip && \
    pip install --no-cache-dir -r requirements.txt

# Copy application code
COPY . .

# Create temp directory for SQLite
RUN mkdir -p /tmp

# Set environment variables
ENV PYTHONUNBUFFERED=1
ENV ADK_FORCE_LOCAL_STORAGE=1

# Expose port
EXPOSE 8000

# Run the ADK web server with UI
CMD ["adk", "web", ".", "--host", "0.0.0.0", "--port", "8000", "--session_service_uri", "sqlite:////tmp/adk_sessions.db"]
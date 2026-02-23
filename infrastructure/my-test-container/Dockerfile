# Dockerfile for evidently drift detection service
FROM python:3.13-slim

WORKDIR /app

# Install dependencies
RUN pip install pandas


# Copy your files into the container
COPY test_script.py .

ENTRYPOINT ["python", "test_script.py"]
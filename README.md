# Rental Price Prediction REST API

Airbnb Rental Price Prediction - Flask API Backend + Streamlit Frontend (Dockerized)

Authored by: Bijon Guha

## Project Overview
This project combines a machine learning-powered backend API with a Streamlit frontend user interface to predict Airbnb rental prices based on property features.

The application is designed for local Docker-based deployment using Docker Compose, making it easy to run both services together in a shared network.

## Architecture

### Backend
- Built with Python and Flask
- Exposes a REST API for rental price prediction
- Loads a serialized trained model from the file `rental_price_prediction_model_v1_0.joblib`
- Runs on port `7860` inside the container
- Listens on `0.0.0.0:7860` for container accessibility
- Supports:
  - `GET /` for a welcome message
  - `POST /v1/rental` for single-property prediction
  - `POST /v1/rentalbatch` for batch predictions from a CSV file

### Frontend
- Built with Python and Streamlit
- Provides an interactive UI for entering property details and uploading CSV files
- Sends requests to the backend API for prediction
- Runs on port `8501` inside the container
- Exposed publicly via `http://localhost:8501`
- Uses a backend URL that can be configured through environment variables, with Compose wiring to the Docker host gateway when needed

## Docker Compose Methodology
The project uses Docker Compose to orchestrate multiple services in a single deployment setup:

- `backend` service builds from the `backend` directory and runs the Flask API
- `frontend` service builds from the `frontend` directory and runs the Streamlit app
- Both services are placed on a custom bridge network named `churn-app-network`
- The backend is exposed on the host port `7860`
- The frontend is exposed on the host port `8501`
- The frontend depends on the backend and optionally waits for the backend health check before making requests

This design keeps the app modular while allowing the frontend to call the backend over the internal Docker network or via the host gateway in local Docker environments.

## Container Communication
The frontend app uses the backend in a client-server pattern:

- Frontend collects input from the user
- Frontend transforms the values into the expected JSON shape
- Frontend sends a POST request to the backend prediction endpoint
- Backend loads the trained ML model and returns the predicted rental price
- The Streamlit UI then displays the result to the user

For batch prediction, the frontend uploads a CSV file to the backend, which reads it, performs predictions, and returns a JSON/dictionary-style result.

## Model and Prediction Logic
The backend uses a trained regression model to predict log-price values, then converts them back to actual dollar values using exponential transformation:

- Model predicts log-price
- `np.exp(predicted_log_price)` converts it to the actual price
- Result is rounded to 2 decimal places

This is important for accurate output formatting in the API response.

## Local Run Instructions
From the project root, run:

```bash
docker compose up --build
```

Then open:

- Frontend: http://localhost:8501
- Backend API: http://localhost:7860

To stop the services:

```bash
docker compose down
```

## Service Details

### Backend API Endpoints
- `GET /` returns a welcome message
- `POST /v1/rental` accepts a JSON object with fields like:
  - `room_type`
  - `accommodates`
  - `bathrooms`
  - `cancellation_policy`
  - `cleaning_fee`
  - `instant_bookable`
  - `review_scores_rating`
  - `bedrooms`
  - `beds`
- `POST /v1/rentalbatch` accepts a CSV upload file containing multiple property records

### Frontend Features
- Online prediction form for a single listing
- Batch prediction flow for multiple listings via CSV upload
- User-friendly Streamlit widgets for each input field
- Displays prediction result and uploaded batch output

## Notes
- The backend is the computational layer and keeps the ML inference logic isolated.
- The frontend is the presentation layer and interacts with the backend via HTTP requests.
- Docker Compose enables local development and deployment with minimal setup complexity.
- The project is intentionally simple and suitable for demonstration, testing, and lightweight deployment.

## Author
Bijon Guha

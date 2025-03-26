# Install dependencies for both backend and frontend
install:
	@echo "Installing backend dependencies..."
	cd backend && poetry install
	@echo "Installing frontend dependencies..."
	cd frontend && npm install

# Start backend in development mode
dev-backend:
	@echo "Starting backend in development mode..."
	cd backend && poetry run uvicorn backend.main:app --reload

# Start frontend in development mode
dev-frontend:
	@echo "Starting frontend in development mode..."
	cd frontend && npm run dev

# Start only the database using Docker Compose
dev-database:
	@echo "Starting database..."
	docker compose up -d database

# Clean up
clean:
	@echo "Cleaning up..."
	docker compose down
	cd frontend && rm -rf node_modules
	cd backend && poetry env remove --all
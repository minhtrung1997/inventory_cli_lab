# Project Structure and Development Setup

This document outlines the recommended project structure and development setup for implementing the KidSpire Inventory System based on the blueprint.

## Project Structure

```
inventory-system/
├── README.md
├── INVENTORY_BLUEPRINT.md
├── REQUIREMENTS.md
├── docs/                          # Documentation
│   ├── api/                       # API documentation
│   ├── deployment/                # Deployment guides
│   ├── user-guides/              # End-user documentation
│   └── technical/                # Technical specifications
├── backend/                       # Python/FastAPI backend
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py               # FastAPI application
│   │   ├── config/               # Configuration management
│   │   ├── models/               # Database models
│   │   ├── schemas/              # Pydantic schemas
│   │   ├── services/             # Business logic
│   │   ├── api/                  # API routes
│   │   ├── core/                 # Core utilities
│   │   └── tests/                # Backend tests
│   ├── alembic/                  # Database migrations
│   ├── requirements.txt          # Python dependencies
│   ├── Dockerfile
│   └── docker-compose.yml
├── frontend/                      # React frontend
│   ├── public/
│   ├── src/
│   │   ├── components/           # Reusable components
│   │   ├── pages/               # Page components
│   │   ├── hooks/               # Custom React hooks
│   │   ├── services/            # API clients
│   │   ├── store/               # Redux store
│   │   ├── utils/               # Utility functions
│   │   └── styles/              # CSS/styling
│   ├── package.json
│   ├── Dockerfile
│   └── nginx.conf
├── mobile/                        # React Native app
│   ├── src/
│   ├── android/
│   ├── ios/
│   └── package.json
├── infrastructure/                # Infrastructure as Code
│   ├── terraform/               # AWS infrastructure
│   ├── kubernetes/              # K8s manifests
│   └── helm/                    # Helm charts
├── scripts/                       # Automation scripts
│   ├── setup.sh                 # Development setup
│   ├── deploy.sh                # Deployment script
│   └── migrate.py               # Data migration
└── .github/                      # GitHub workflows
    └── workflows/
        ├── ci.yml               # Continuous Integration
        ├── cd.yml               # Continuous Deployment
        └── security.yml         # Security scanning
```

## Development Environment Setup

### Prerequisites

```bash
# Required software versions
Python 3.11+
Node.js 18+
Docker 20.10+
PostgreSQL 14+
Redis 6.2+
```

### Quick Start Script

Create a setup script to automate development environment:

```bash
#!/bin/bash
# scripts/setup.sh

set -e

echo "🚀 Setting up KidSpire Inventory System development environment..."

# Check prerequisites
command -v python3 >/dev/null 2>&1 || { echo "Python 3 is required"; exit 1; }
command -v node >/dev/null 2>&1 || { echo "Node.js is required"; exit 1; }
command -v docker >/dev/null 2>&1 || { echo "Docker is required"; exit 1; }

# Backend setup
echo "📦 Setting up backend..."
cd backend
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
cd ..

# Frontend setup
echo "🖥️  Setting up frontend..."
cd frontend
npm install
cd ..

# Mobile setup
echo "📱 Setting up mobile app..."
cd mobile
npm install
cd ..

# Infrastructure setup
echo "🏗️  Starting development services..."
docker-compose -f docker-compose.dev.yml up -d

echo "✅ Development environment ready!"
echo "📝 Next steps:"
echo "   1. Copy .env.example to .env and configure"
echo "   2. Run: cd backend && source venv/bin/activate"
echo "   3. Run: alembic upgrade head (database migrations)"
echo "   4. Run: cd frontend && npm start"
```

## API Design Standards

### RESTful Endpoints

```python
# Backend API structure
/api/v1/
├── auth/
│   ├── POST /login
│   ├── POST /logout
│   ├── POST /refresh
│   └── GET  /me
├── users/
│   ├── GET    /users          # List users
│   ├── POST   /users          # Create user
│   ├── GET    /users/{id}     # Get user
│   ├── PUT    /users/{id}     # Update user
│   └── DELETE /users/{id}     # Delete user
├── items/
│   ├── GET    /items          # List items (with search/filter)
│   ├── POST   /items          # Create item
│   ├── GET    /items/{id}     # Get item details
│   ├── PUT    /items/{id}     # Update item
│   ├── DELETE /items/{id}     # Delete item
│   └── GET    /items/{id}/history # Item transaction history
├── transactions/
│   ├── GET    /transactions   # List transactions
│   ├── POST   /transactions/checkout    # Check out item
│   ├── POST   /transactions/checkin     # Check in item
│   ├── POST   /transactions/reserve     # Reserve item
│   └── GET    /transactions/{id}        # Get transaction
├── categories/
│   ├── GET    /categories     # List categories
│   ├── POST   /categories     # Create category
│   └── GET    /categories/{id}/items # Items in category
└── reports/
    ├── GET    /reports/usage  # Usage analytics
    ├── GET    /reports/items  # Item reports
    └── GET    /reports/users  # User activity
```

### Response Standards

```json
{
  "success": true,
  "data": {
    "items": [...],
    "pagination": {
      "page": 1,
      "per_page": 20,
      "total": 100,
      "pages": 5
    }
  },
  "message": "Items retrieved successfully"
}
```

Error responses:
```json
{
  "success": false,
  "error": {
    "code": "ITEM_NOT_FOUND",
    "message": "Item with ID 123 not found",
    "details": {}
  }
}
```

## Database Schema Implementation

### Core Tables Creation Scripts

```sql
-- Database setup script
-- backend/alembic/versions/001_create_core_tables.py

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- Users table
CREATE TABLE users (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    username VARCHAR(50) UNIQUE NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    first_name VARCHAR(50) NOT NULL,
    last_name VARCHAR(50) NOT NULL,
    role VARCHAR(20) NOT NULL CHECK (role IN ('teacher', 'admin', 'student', 'staff')),
    department VARCHAR(50),
    active BOOLEAN DEFAULT true,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Categories table
CREATE TABLE categories (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(100) NOT NULL,
    parent_id UUID REFERENCES categories(id),
    description TEXT,
    checkout_rules JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Items table
CREATE TABLE items (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    name VARCHAR(200) NOT NULL,
    description TEXT,
    category_id UUID REFERENCES categories(id),
    sku VARCHAR(50) UNIQUE,
    barcode VARCHAR(100),
    location VARCHAR(100),
    condition VARCHAR(20) DEFAULT 'good' CHECK (condition IN ('excellent', 'good', 'fair', 'poor', 'damaged')),
    purchase_date DATE,
    purchase_price DECIMAL(10,2),
    current_value DECIMAL(10,2),
    availability_status VARCHAR(20) DEFAULT 'available' CHECK (availability_status IN ('available', 'checked_out', 'maintenance', 'retired')),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Transactions table
CREATE TABLE transactions (
    id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
    item_id UUID REFERENCES items(id),
    user_id UUID REFERENCES users(id),
    transaction_type VARCHAR(20) NOT NULL CHECK (transaction_type IN ('checkout', 'checkin', 'reservation', 'maintenance')),
    checkout_date TIMESTAMP,
    expected_return_date TIMESTAMP,
    actual_return_date TIMESTAMP,
    notes TEXT,
    condition_out VARCHAR(20),
    condition_in VARCHAR(20),
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- Indexes for performance
CREATE INDEX idx_items_category ON items(category_id);
CREATE INDEX idx_items_status ON items(availability_status);
CREATE INDEX idx_transactions_item ON transactions(item_id);
CREATE INDEX idx_transactions_user ON transactions(user_id);
CREATE INDEX idx_transactions_type ON transactions(transaction_type);
```

## Testing Strategy

### Backend Testing
```python
# backend/app/tests/test_items.py
import pytest
from fastapi.testclient import TestClient
from app.main import app

client = TestClient(app)

def test_create_item():
    response = client.post(
        "/api/v1/items",
        json={
            "name": "Test Textbook",
            "description": "Sample textbook for testing",
            "category_id": "uuid-here",
            "location": "Library-A1"
        },
        headers={"Authorization": "Bearer token"}
    )
    assert response.status_code == 201
    assert response.json()["data"]["name"] == "Test Textbook"

def test_get_items():
    response = client.get("/api/v1/items")
    assert response.status_code == 200
    assert "items" in response.json()["data"]
```

### Frontend Testing
```javascript
// frontend/src/components/__tests__/ItemCard.test.tsx
import { render, screen } from '@testing-library/react';
import { ItemCard } from '../ItemCard';

test('renders item card with correct information', () => {
  const mockItem = {
    id: '1',
    name: 'Test Item',
    description: 'Test Description',
    availability_status: 'available'
  };

  render(<ItemCard item={mockItem} />);
  
  expect(screen.getByText('Test Item')).toBeInTheDocument();
  expect(screen.getByText('Test Description')).toBeInTheDocument();
  expect(screen.getByText('Available')).toBeInTheDocument();
});
```

## Deployment Configuration

### Docker Compose for Development
```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  db:
    image: postgres:14
    environment:
      POSTGRES_DB: inventory_dev
      POSTGRES_USER: inventory
      POSTGRES_PASSWORD: password
    ports:
      - "5432:5432"
    volumes:
      - postgres_data:/var/lib/postgresql/data

  redis:
    image: redis:6-alpine
    ports:
      - "6379:6379"

  backend:
    build: ./backend
    environment:
      DATABASE_URL: postgresql://inventory:password@db:5432/inventory_dev
      REDIS_URL: redis://redis:6379
    ports:
      - "8000:8000"
    volumes:
      - ./backend:/app
    depends_on:
      - db
      - redis

  frontend:
    build: ./frontend
    ports:
      - "3000:3000"
    volumes:
      - ./frontend:/app
    environment:
      REACT_APP_API_URL: http://localhost:8000

volumes:
  postgres_data:
```

## CI/CD Pipeline

### GitHub Actions Workflow
```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [ main, develop ]
  pull_request:
    branches: [ main ]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: postgres
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5

    steps:
    - uses: actions/checkout@v3
    
    - name: Set up Python
      uses: actions/setup-python@v3
      with:
        python-version: '3.11'
    
    - name: Set up Node.js
      uses: actions/setup-node@v3
      with:
        node-version: '18'
    
    - name: Install backend dependencies
      run: |
        cd backend
        pip install -r requirements.txt
    
    - name: Run backend tests
      run: |
        cd backend
        pytest
    
    - name: Install frontend dependencies
      run: |
        cd frontend
        npm install
    
    - name: Run frontend tests
      run: |
        cd frontend
        npm test
    
    - name: Build frontend
      run: |
        cd frontend
        npm run build
```

This structure provides a solid foundation for implementing the KidSpire Inventory System according to the blueprint specifications.
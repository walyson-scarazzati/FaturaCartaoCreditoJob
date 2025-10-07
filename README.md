# Credit Card Invoice Processing Job

A Spring Batch application that processes credit card transactions and generates consolidated invoices for customers.

This project is from the "Curso Otimização de desempenho para jobs Spring Batch" - Udemy: https://www.udemy.com/course/otimizacao-de-desempenho-para-jobs-spring-batch/

## 📋 Project Overview

This Spring Batch job processes credit card transactions from a database and generates consolidated invoices by:

1. **Reading** transactions from the `transacao` table grouped by credit card
2. **Processing** each invoice by fetching customer details from an external REST API
3. **Writing** the consolidated invoice data to a `fatura` table

### 🏗️ Architecture

The application follows the Spring Batch architecture with:
- **Reader**: `FaturaCartaoCreditoReader` - Groups transactions by credit card number
- **Processor**: `CarregarDadosClienteProcessor` - Enriches data with customer information from REST API
- **Writer**: `BancoFaturaCartaoCreditoWriter` - Saves invoice totals to database

### 📊 Database Schema

**Source Tables:**
- `transacao`: Transaction records
  - `id`: Transaction ID
  - `numero_cartao_credito`: Credit card number
  - `descricao`: Transaction description
  - `valor`: Transaction amount
  - `data`: Transaction date

- `cartao_credito`: Credit card information
  - `numero_cartao_credito`: Credit card number (PK)
  - `cliente`: Customer ID

**Target Table:**
- `fatura`: Generated invoices
  - `valor`: Total invoice amount
  - `data`: Invoice generation date

## 🚀 Setup and Installation

### Prerequisites
- Java 8 or higher
- Maven 3.6+
- Docker and Docker Compose (recommended)

### 1. Database Setup

#### Option A: Using Docker Compose (Recommended)

```bash
# Navigate to the resources directory
cd src/main/resources

# Start MySQL container with pre-configured data
docker-compose up -d

# Verify container is running
docker ps
```

The Docker setup automatically:
- Creates MySQL 8.0 container on port 3306
- Sets up `spring_batch` and `fatura_cartao_credito` databases
- Loads initial data from `files/scripts.sql`
- Creates user `fatura` with password `123456`

#### Option B: Manual MySQL Setup

1. Install MySQL and create databases:
```sql
CREATE DATABASE spring_batch;
CREATE DATABASE fatura_cartao_credito;
```

2. Execute the SQL script:
```bash
mysql -u root -p < files/scripts.sql
```

### 2. Application Configuration

The application uses two data sources configured in `application.properties`:

- **Spring Batch DataSource**: For job metadata (spring_batch database)
- **Application DataSource**: For business data (fatura_cartao_credito database)

### 3. Build and Run

```bash
# Clone the repository
git clone <repository-url>
cd FaturaCartaoCreditoJob

# Build the project
./mvnw clean install

# Run the application
./mvnw spring-boot:run

# Alternative: Run the JAR
java -jar target/FaturaCartaoCreditoJob-0.0.1-SNAPSHOT.jar
```

## 🔧 Configuration

### External API Integration

The processor integrates with a REST API to fetch customer details:
- **Service URL**: `https://my-json-server.typicode.com/giuliana-bezerra/demo/profile/{customerId}`
- **Purpose**: Enriches invoice data with customer name and address

### Performance Features

- **Multi-threading**: Configured with ThreadPoolTaskExecutor (4 core threads)
- **Chunk Processing**: Processes invoices in chunks for optimal performance
- **Transaction Management**: Separate transaction managers for batch and business operations

## 📈 Job Flow

1. **Start**: Job reads transactions ordered by credit card number
2. **Group**: Reader groups transactions by credit card into invoice objects
3. **Enrich**: Processor calls REST API to get customer details for each invoice
4. **Calculate**: Invoice total is calculated from all associated transactions
5. **Persist**: Writer saves invoice with total amount and current date to database
6. **Complete**: Job execution summary is logged

## 🛠️ Development

### Key Components

- `FaturaCartaoCreditoJobConfig`: Main job configuration
- `FaturaCartaoCreditoStepConfig`: Step definition with reader, processor, writer
- `DataSourceConfig`: Dual database configuration
- `TaskExecutorConfig`: Multi-threading setup

### Domain Objects

- `FaturaCartaoCredito`: Invoice aggregate with transactions list
- `Transacao`: Individual transaction record
- `CartaoCredito`: Credit card information
- `Cliente`: Customer details from external API

## 📝 Notes

- The job uses chunk-oriented processing with chunk size of 1
- Customer data is fetched synchronously from external REST API
- Invoice totals are calculated in-memory using Java streams
- The application automatically closes after job completion

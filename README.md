Order Processor System

## 1. Project Overview

This project is a **file-based order processing system** built using **.NET Worker Service**. The system automatically monitors a folder for incoming order files (JSON format), validates them, processes valid orders, stores them in a SQLite database, and safely handles invalid orders.

The project is designed to simulate a **real-world backend background service** that runs continuously without manual intervention.

---

## 2. Architecture Overview

The solution contains **two applications**:

### 1. OrderProcessorService (Worker Service)

* Runs continuously in the background
* Watches a folder for incoming order files
* Validates and processes orders
* Saves valid orders to a SQLite database
* Moves invalid orders to a failure folder

### 2. OrderFileGenerator (Console App)

* Generates bulk order files automatically
* Used to test high-volume order processing

---

## 3. Folder Structure

```
OrderProcessorSolution
│
├── OrderProcessorService
│   ├── IncomingOrders     # New order files arrive here
│   ├── ProcessedOrders    # Successfully processed orders
│   ├── FailedOrders       # Invalid order files
│   ├── Database
│   │   └── orders.db      # SQLite database
│   ├── Program.cs
│   ├── Worker.cs
│   └── OrderProcessorService.csproj
│
├── OrderFileGenerator
│   ├── Program.cs
│   └── OrderFileGenerator.csproj
│
└── README.md
```

---

## 4. How the System Works

1. The Worker Service starts and monitors the `IncomingOrders` folder
2. When a new `.json` file is detected:

   * The file is read and parsed
   * Validation rules are applied
3. If valid:

   * Order is saved to the database
   * File is moved to `ProcessedOrders`
4. If invalid:

   * Error is logged
   * File is moved to `FailedOrders`
5. The service continues running without stopping

---

## 5. Validation Rules

An order is considered **valid** only if:

* `CustomerName` is NOT empty
* `TotalAmount` is greater than 0

If any rule fails, the order is treated as invalid.

---

## 6. Database Details

* Database used: **SQLite**
* File location: `OrderProcessorService/Database/orders.db`
* Tables:

  * Orders (valid orders)
  * InvalidOrders (invalid orders)

SQLite was chosen because it is lightweight and easy to use for local persistence.

---

## 7. Idempotency Handling

Idempotency ensures that **duplicate orders are not saved multiple times**.

This is achieved by:

* Using a unique `OrderId` for each order
* Checking if an order with the same `OrderId` already exists before inserting

This prevents duplicate records even if the same file is processed again.

---

## 8. Logging & Error Handling

* Important events are logged to the console
* Errors during processing do not crash the service
* Invalid files are safely moved to `FailedOrders`

This ensures the service remains stable and resilient.

---

## 9. Graceful Shutdown

The Worker Service supports **graceful shutdown**.

* Pressing `Ctrl + C` stops the service safely
* Ongoing operations are completed
* No data corruption occurs

---

## 10. High-Volume Processing

The `OrderFileGenerator` project generates multiple order files (100+ at a time) to test:

* Performance
* Stability
* Error handling

The Worker Service processes these files without crashing.

---

## 11. How to Run the Project

### Step 1: Run the Worker Service

```
dotnet run
```

(from `OrderProcessorService` folder)

### Step 2: Run the File Generator

```
dotnet run
```

(from `OrderFileGenerator` folder)

---

## 12. Testing

* Manual testing performed using generated order files
* Valid and invalid scenarios verified
* High-volume testing performed using generator application

---

## 13. Future Improvements

* Add retry mechanism for transient failures
* Implement file-based or structured logging
* Add unit tests using xUnit
* Dockerize the application
* Cloud storage integration (Azure Blob / AWS S3)

---

## 14. Conclusion

This project demonstrates a robust background processing system with validation, persistence, error handling, and scalability. It follows clean architecture principles and is suitable for real-world backend services.

this is the readme.md
OrderProcessorSolution

Overview

This repository contains two projects:

- OrderProcessorService: A background worker service that processes order JSON files.
  - IncomingOrders: Where new order files are placed.
  - ProcessedOrders: Where successfully processed orders are moved.
  - FailedOrders: Where orders that fail processing are moved.
  - Database: Contains the SQLite database (orders.db).
  - Worker.cs: The service worker.
  - Program.cs: Service host entrypoint.

- OrderFileGenerator: A small console application that generates order files for testing.

Build & Run

- Build the entire solution:
```
dotnet build OrderProcessorSolution.sln
```

- Run the service (from the OrderProcessorService folder):
```
dotnet run --project OrderProcessorService/OrderProcessorService.csproj
```

- Run the generator (from the OrderFileGenerator folder):
```
dotnet run --project OrderFileGenerator/OrderFileGenerator.csproj
```

Notes

- This repository already contains folders `IncomingOrders`, `ProcessedOrders`, `FailedOrders`, and `Database` with an example `orders.db`.
- If the service's exe file is running, stop it before building to avoid file lock errors when building.

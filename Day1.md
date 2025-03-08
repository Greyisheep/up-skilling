# Days 1 – 10 of Software Engineering Skills Improvement

## Day 1: Database Fundamentals & PostgreSQL Setup

### Theory (40 min)
- **Overview of PostgreSQL architecture:** storage engine, transaction management, and concurrency control  
- **Understanding transaction isolation levels and ACID properties**

### Practical (30 min)
- Set up PostgreSQL in Docker with persistent volumes  
- Run basic SQL commands and verify connectivity

### Resources
- PostgreSQL documentation (chapters 1–3)  
- Docker volumes documentation

---

# Database Fundamentals (Focusing on Distributed System Consistency)

## Introduction to Database Architecture

Modern applications, especially in distributed environments, require a solid understanding of database fundamentals to ensure data integrity and consistent performance. This note explores the core concepts of database systems—with a focus on PostgreSQL, transaction management, concurrency control, and techniques for maintaining consistency across distributed systems.

---

## 1. PostgreSQL Architecture Overview

### 1.1 Storage Engine

PostgreSQL uses a storage system called "heap" for table data:
- **Relations:** Data is stored in files called relations, organized in pages (typically 8KB)
- **Multi-Version Concurrency Control (MVCC):** Updates create new row versions rather than overwriting existing data
- **Write-Ahead Log (WAL):** Ensures data integrity during crashes by logging changes before they're applied to data files
- **Buffer Cache:** Minimizes disk I/O by caching frequently accessed data pages in memory

### 1.2 Transaction Management

PostgreSQL's transaction system ensures data consistency through:
- **Atomic Operations:** Each transaction either completes entirely or not at all
- **Two-Phase Commit Protocol:** Enables distributed transactions across multiple nodes
- **Transaction Logs:** Records all changes for recovery purposes
- **Savepoints:** Allows for partial rollbacks within transactions

### 1.3 Concurrency Control

PostgreSQL employs sophisticated mechanisms to handle multiple simultaneous connections:
- **MVCC Implementation:** Allows readers to access data without blocking writers and vice versa
- **Snapshot Isolation:** Each transaction sees a consistent view of the database at a specific point in time
- **Row-Level Locking:** Used when necessary to prevent conflicting changes
- **Vacuum Process:** Removes old row versions (dead tuples) to reclaim storage space

---

## 2. ACID Properties Deep Dive

### 2.1 Atomicity

Transactions in PostgreSQL are all-or-nothing operations. When you execute a transaction:
- All operations within the transaction either complete successfully or none take effect.
- If an error occurs partway through, all changes are rolled back.
- This ensures partial changes never appear in the database.

**Example:**

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00 WHERE name = 'Alice';
UPDATE accounts SET balance = balance + 100.00 WHERE name = 'Bob';
COMMIT;
```

If the second update fails (e.g., Bob's account doesn't exist), the first update is rolled back and Alice's balance remains unchanged.

### 2.2 Consistency

Database consistency ensures that transactions transform the database from one valid state to another:
- Constraints, triggers, and cascades maintain data integrity.
- Foreign key constraints ensure referential integrity.
- Check constraints enforce business rules.
- The database remains in a valid state before and after transactions.

### 2.3 Isolation

Transaction isolation prevents interference between concurrent transactions:
- Each transaction operates as if it were the only one accessing the database.
- Different isolation levels provide trade-offs between performance and consistency.
- PostgreSQL's isolation levels prevent various anomalies based on application needs.

### 2.4 Durability

Once a transaction is committed, changes persist even through system failures:
- Committed changes are written to the WAL.
- In the event of a crash, WAL is used to recover the database to a consistent state.
- Physical replication provides additional durability guarantees.

---

## 3. Transaction Isolation Levels

PostgreSQL supports all four isolation levels defined in the SQL standard, each with different guarantees:

### 3.1 Read Uncommitted
- PostgreSQL treats this the same as Read Committed.
- Does not allow dirty reads (unlike some other database systems).

### 3.2 Read Committed (Default)
- Each query sees only data committed before the query began.
- Never sees uncommitted data.
- May see changes committed by concurrent transactions between statements.
- Prevents dirty reads but allows nonrepeatable reads and phantom reads.

*Real-world example:*  
In a banking application, a report that runs multiple queries under Read Committed might see inconsistent data if transactions are committing during the report generation. For instance, a transfer might be counted in one account but not yet in the other.

### 3.3 Repeatable Read
- Provides a stable view of committed data as of the start of the transaction.
- All queries within the transaction see the same data.
- Prevents dirty reads and nonrepeatable reads.
- May still allow serialization anomalies.
- Implemented using Snapshot Isolation.

*Real-world example:*  
An inventory management system needs to ensure consistent counts when processing an order. With Repeatable Read, all items in the order will be seen with the same inventory levels throughout the transaction, preventing inconsistencies even if other transactions are updating inventory.

### 3.4 Serializable
- Strictest isolation level.
- Emulates serial transaction execution.
- Prevents all types of isolation anomalies including serialization anomalies.
- Transactions may fail with serialization errors requiring retries.
- Uses predicate locking to detect conflicts.

*Real-world example:*  
In a financial application performing complex accounting calculations, Serializable isolation ensures that concurrent modifications don't lead to inconsistent results, such as different balance totals depending on calculation order.

---

## 4. Advanced PostgreSQL Features

### 4.1 Views and Data Abstraction

Views provide a powerful abstraction layer:
- Encapsulate complex queries behind a simple interface.
- Hide table structure details from application code.
- Can be updated (with certain restrictions).
- Support nesting (views based on other views).

```sql
CREATE VIEW customer_summary AS
    SELECT c.customer_id, c.name, COUNT(o.order_id) as order_count, SUM(o.total) as total_spent
    FROM customers c
    LEFT JOIN orders o ON c.customer_id = o.customer_id
    GROUP BY c.customer_id, c.name;

SELECT * FROM customer_summary WHERE order_count > 10;
```

### 4.2 Table Inheritance

PostgreSQL supports table inheritance:
- Create specialized tables that inherit from a base table.
- Query the parent table to include data from all child tables.
- Useful for partitioning data or modeling hierarchical relationships.

```sql
CREATE TABLE cities (
  name       text,
  population real,
  elevation  int
);

CREATE TABLE capitals (
  state      char(2) UNIQUE NOT NULL
) INHERITS (cities);
```

### 4.3 Savepoints and Partial Rollbacks

Savepoints allow for more granular transaction control:
- Define points within a transaction to which you can roll back.
- Useful for complex transactions with error handling.
- Released automatically when no longer needed.

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100.00 WHERE name = 'Alice';
SAVEPOINT my_savepoint;
UPDATE accounts SET balance = balance + 100.00 WHERE name = 'Bob';
-- Oops, Bob's account shouldn't be updated
ROLLBACK TO my_savepoint;
UPDATE accounts SET balance = balance + 100.00 WHERE name = 'Charlie';
COMMIT;
```

---

## 5. Concurrency Challenges and Solutions

### 5.1 Common Concurrency Issues

When multiple transactions operate concurrently, several issues can arise:

#### 5.1.1 Dirty Reads
- Transaction reads data written by a concurrent uncommitted transaction.
- Can lead to processing based on data that may be rolled back.
- Prevented by all PostgreSQL isolation levels.

#### 5.1.2 Nonrepeatable Reads
- Transaction re-reads data and finds it modified by another committed transaction.
- Can lead to inconsistent processing within a transaction.
- Prevented by Repeatable Read and Serializable isolation levels.

#### 5.1.3 Phantom Reads
- Transaction re-executes a query and finds the set of rows has changed.
- New rows may appear that match a previous query's conditions.
- Prevented by Serializable isolation (partially by Repeatable Read).

#### 5.1.4 Serialization Anomalies
- Results of concurrent transactions are inconsistent with any serial execution.
- Most complex to detect and prevent.
- Only prevented by Serializable isolation level.

### 5.2 Locking Strategies

PostgreSQL provides various locking mechanisms:

#### 5.2.1 Row-Level Locks
- **SELECT FOR UPDATE:** Locks returned rows for update.
- **SELECT FOR SHARE:** Locks rows against concurrent updates but allows concurrent reads.
- Useful for implementing optimistic concurrency control.

#### 5.2.2 Table-Level Locks
- **LOCK TABLE:** Acquires explicit locks on entire tables.
- Various lock modes with different compatibility rules.
- Useful for administrative operations or ensuring table-wide consistency.

#### 5.2.3 Advisory Locks
- Application-defined locks not tied to specific database objects.
- Useful for coordinating activities across multiple sessions.
- Can be session-level or transaction-level.

### 5.3 Handling Concurrency in Applications

Best practices for handling concurrency in applications:
- Choose appropriate isolation levels based on consistency requirements.
- Implement retry logic for serialization failures.
- Keep transactions short to minimize contention.
- Consider using optimistic concurrency control for high-contention scenarios.
- Use explicit locking when necessary to enforce business rules.

**Example retry logic in Go:**

```go
func executeTransaction(db *sql.DB, txFunc func(*sql.Tx) error) error {
    maxRetries := 3
    for retries := 0; retries < maxRetries; retries++ {
        tx, err := db.Begin()
        if err != nil {
            return err
        }
        
        err = txFunc(tx)
        if err != nil {
            tx.Rollback()
            
            // Check if it's a serialization error
            if pqErr, ok := err.(*pq.Error); ok && pqErr.Code == "40001" {
                // Serialization failure, retry after exponential backoff
                sleepTime := time.Duration(100 * math.Pow(2, float64(retries))) * time.Millisecond
                time.Sleep(sleepTime)
                continue
            }
            return err
        }
        
        // Try to commit
        err = tx.Commit()
        if err != nil {
            // Check if it's a serialization error
            if pqErr, ok := err.(*pq.Error); ok && pqErr.Code == "40001" {
                // Serialization failure, retry after exponential backoff
                sleepTime := time.Duration(100 * math.Pow(2, float64(retries))) * time.Millisecond
                time.Sleep(sleepTime)
                continue
            }
            return err
        }
        
        // Success
        return nil
    }
    
    return fmt.Errorf("transaction failed after %d retries", maxRetries)
}
```

## 6. Distributed Database Consistency

### 6.1 Challenges in Distributed Systems
- **Network Partitions:** Communication failures between nodes  
- **Latency:** Delays in propagating changes across nodes  
- **Partial Failures:** Some nodes may fail while others continue operating  
- **Concurrent Updates:** The same data may be modified on different nodes simultaneously  

### 6.2 CAP Theorem and Trade-offs
The CAP theorem states that distributed systems can provide at most two of three guarantees:
- **Consistency:** All nodes see the same data at the same time  
- **Availability:** Every request receives a response  
- **Partition Tolerance:** The system continues to operate despite network partitions  

Database architects must make trade-offs based on business requirements:
- **CP systems:** Prioritize consistency over availability  
- **AP systems:** Prioritize availability over consistency  
- **CA systems:** Are not practical because partition tolerance is required  

### 6.3 Consistency Models

#### 6.3.1 Strong Consistency
- All nodes see the same data at the same time  
- Changes are immediately visible to all nodes  
- Higher latency and reduced availability during partitions  
- Implemented using distributed transactions, consensus protocols, or synchronous replication  

#### 6.3.2 Eventual Consistency
- Guarantees that, given enough time, all nodes will converge to the same state  
- Updates propagate asynchronously  
- Lower latency and higher availability  
- May temporarily return stale data  

#### 6.3.3 Causal Consistency
- Ensures that causally related operations are seen in the same order by all nodes  
- Stronger than eventual consistency but weaker than strong consistency  
- Useful for applications that need to preserve causal relationships  

---

## 7. Distributed Consistency Patterns in Microservices

### 7.1 Saga Pattern
Coordinates transactions across multiple services through a series of local transactions:
1. Each service performs a local transaction and publishes an event  
2. Other services listen for events and perform their own local transactions  
3. If a step fails, compensating transactions roll back the changes  

*Example in Fintech:* A payment processing system might implement a saga for fund transfers:
- **Account Service:** Reserve funds from the sender’s account  
- **Compliance Service:** Verify the transaction against anti-fraud rules  
- **Transfer Service:** Process the transfer  
- **Notification Service:** Send confirmation to users  

**Code Example:**

```go
// SagaCoordinator orchestrates a distributed transaction across services
type SagaCoordinator struct {
    steps         []SagaStep
    compensations map[int]CompensatingAction
    logger        *zap.Logger
    mutex         sync.Mutex
}

type SagaStep struct {
    Name       string
    Execute    func(ctx context.Context) error
    Compensate func(ctx context.Context) error
}

type CompensatingAction struct {
    StepName string
    Execute  func(ctx context.Context) error
}

// NewSaga creates a new saga transaction
func NewSaga(logger *zap.Logger) *SagaCoordinator {
    return &SagaCoordinator{
        steps:         []SagaStep{},
        compensations: make(map[int]CompensatingAction),
        logger:        logger,
    }
}

// AddStep adds a step to the saga
func (s *SagaCoordinator) AddStep(step SagaStep) {
    s.mutex.Lock()
    defer s.mutex.Unlock()
    s.steps = append(s.steps, step)
}

// Execute runs the saga and performs compensation if any step fails
func (s *SagaCoordinator) Execute(ctx context.Context) error {
    s.mutex.Lock()
    defer s.mutex.Unlock()
    
    for i, step := range s.steps {
        s.logger.Info("Executing saga step", zap.String("step", step.Name))
        
        if err := step.Execute(ctx); err != nil {
            s.logger.Error("Step failed, initiating compensation",
                zap.String("step", step.Name),
                zap.Error(err))
            // Compensate in reverse order
            return s.compensate(ctx, i-1)
        }
        
        // Store compensation function
        s.compensations[i] = CompensatingAction{
            StepName: step.Name,
            Execute:  step.Compensate,
        }
    }
    
    s.logger.Info("Saga completed successfully")
    return nil
}

// compensate performs compensation actions for failed saga
func (s *SagaCoordinator) compensate(ctx context.Context, lastSuccessfulStep int) error {
    var compensationErr error
    
    // Compensate in reverse order
    for i := lastSuccessfulStep; i >= 0; i-- {
        compensation, exists := s.compensations[i]
        if !exists {
            continue
        }
        
        s.logger.Info("Executing compensation", zap.String("step", compensation.StepName))
        
        if err := compensation.Execute(ctx); err != nil {
            s.logger.Error("Compensation failed", 
                zap.String("step", compensation.StepName),
                zap.Error(err))
            
            if compensationErr == nil {
                compensationErr = fmt.Errorf("compensation failed: %w", err)
            }
        }
    }
    
    if compensationErr != nil {
        return fmt.Errorf("saga failed with compensation errors: %w", compensationErr)
    }
    
    return fmt.Errorf("saga failed but compensated successfully")
}
```

### 7.2 Event Sourcing
Stores changes to application state as a sequence of events:
- Each state change is recorded as an immutable event  
- Current state is derived by replaying events  
- Provides a complete audit trail  
- Facilitates event-driven architectures  

*Example in Fintech:* An investment platform might use event sourcing to track all portfolio changes:
- Events like "StockPurchased", "DividendReceived", or "StockSold" are stored  
- The current portfolio value is calculated by processing all events  
- Provides a complete history for auditing and compliance  

**Code Example:**

```go
// Event represents a domain event
type Event struct {
    ID        string    `json:"id"`
    Type      string    `json:"type"`
    EntityID  string    `json:"entity_id"`
    Data      []byte    `json:"data"`
    Metadata  Metadata  `json:"metadata"`
    Timestamp time.Time `json:"timestamp"`
    Version   int64     `json:"version"`
}

type Metadata map[string]interface{}

// EventStore interface for storing and retrieving events
type EventStore interface {
    AppendEvents(ctx context.Context, streamID string, expectedVersion int64, events []Event) error
    ReadEvents(ctx context.Context, streamID string, fromVersion int64, maxCount int) ([]Event, error)
}

// PostgresEventStore implements EventStore using PostgreSQL
type PostgresEventStore struct {
    db     *sql.DB
    logger *zap.Logger
}

func NewPostgresEventStore(db *sql.DB, logger *zap.Logger) *PostgresEventStore {
    return &PostgresEventStore{
        db:     db,
        logger: logger,
    }
}

// AppendEvents adds new events to the event store
func (s *PostgresEventStore) AppendEvents(ctx context.Context, streamID string, expectedVersion int64, events []Event) error {
    tx, err := s.db.BeginTx(ctx, &sql.TxOptions{
        Isolation: sql.LevelSerializable,
    })
    if err != nil {
        return fmt.Errorf("failed to begin transaction: %w", err)
    }
    defer tx.Rollback()

    // Check expected version to prevent concurrency issues
    var currentVersion int64
    err = tx.QueryRowContext(ctx, 
        "SELECT COALESCE(MAX(version), 0) FROM events WHERE stream_id = $1", 
        streamID).Scan(&currentVersion)
    if err != nil {
        return fmt.Errorf("failed to get current version: %w", err)
    }

    if expectedVersion != currentVersion {
        return fmt.Errorf("optimistic concurrency conflict: expected version %d, got %d", 
            expectedVersion, currentVersion)
    }

    // Insert events
    stmt, err := tx.PrepareContext(ctx, `
        INSERT INTO events (id, type, stream_id, entity_id, data, metadata, timestamp, version)
        VALUES ($1, $2, $3, $4, $5, $6, $7, $8)
    `)
    if err != nil {
        return fmt.Errorf("failed to prepare statement: %w", err)
    }
    defer stmt.Close()

    for i, event := range events {
        event.Version = currentVersion + int64(i) + 1
        event.Timestamp = time.Now().UTC()
        if event.ID == "" {
            event.ID = uuid.New().String()
        }

        _, err = stmt.ExecContext(ctx,
            event.ID,
            event.Type,
            streamID,
            event.EntityID,
            event.Data,
            event.Metadata,
            event.Timestamp,
            event.Version,
        )
        if err != nil {
            return fmt.Errorf("failed to insert event: %w", err)
        }
    }

    // Commit transaction
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("failed to commit transaction: %w", err)
    }

    return nil
}

// ReadEvents retrieves events from the event store
func (s *PostgresEventStore) ReadEvents(ctx context.Context, streamID string, fromVersion int64, maxCount int) ([]Event, error) {
    rows, err := s.db.QueryContext(ctx, `
        SELECT id, type, entity_id, data, metadata, timestamp, version
        FROM events
        WHERE stream_id = $1 AND version > $2
        ORDER BY version ASC
        LIMIT $3
    `, streamID, fromVersion, maxCount)
    if err != nil {
        return nil, fmt.Errorf("failed to query events: %w", err)
    }
    defer rows.Close()

    var events []Event
    for rows.Next() {
        var event Event
        var metadataBytes []byte
        
        if err := rows.Scan(
            &event.ID,
            &event.Type,
            &event.EntityID,
            &event.Data,
            &metadataBytes,
            &event.Timestamp,
            &event.Version,
        ); err != nil {
            return nil, fmt.Errorf("failed to scan event: %w", err)
        }

        if err := json.Unmarshal(metadataBytes, &event.Metadata); err != nil {
            return nil, fmt.Errorf("failed to unmarshal metadata: %w", err)
        }

        events = append(events, event)
    }

    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("error iterating over rows: %w", err)
    }

    return events, nil
}

// Example usage: Portfolio Event Sourcing

type Portfolio struct {
    ID            string
    AccountID     string
    Holdings      map[string]decimal.Decimal // Symbol -> Quantity
    CostBasis     map[string]decimal.Decimal // Symbol -> Cost Basis
    TotalDeposits decimal.Decimal
    Version       int64
}

// Apply updates portfolio state from an event
func (p *Portfolio) Apply(event Event) error {
    switch event.Type {
    case "PortfolioCreated":
        var data struct {
            AccountID string `json:"account_id"`
        }
        if err := json.Unmarshal(event.Data, &data); err != nil {
            return err
        }
        p.ID = event.EntityID
        p.AccountID = data.AccountID
        p.Holdings = make(map[string]decimal.Decimal)
        p.CostBasis = make(map[string]decimal.Decimal)
        p.TotalDeposits = decimal.Zero
        
    case "FundsDeposited":
        var data struct {
            Amount decimal.Decimal `json:"amount"`
        }
        if err := json.Unmarshal(event.Data, &data); err != nil {
            return err
        }
        p.TotalDeposits = p.TotalDeposits.Add(data.Amount)
        
    case "StockPurchased":
        var data struct {
            Symbol   string          `json:"symbol"`
            Quantity decimal.Decimal `json:"quantity"`
            Price    decimal.Decimal `json:"price"`
        }
        if err := json.Unmarshal(event.Data, &data); err != nil {
            return err
        }
        
        currentQuantity := p.Holdings[data.Symbol]
        p.Holdings[data.Symbol] = currentQuantity.Add(data.Quantity)
        
        currentCost := p.CostBasis[data.Symbol]
        additionalCost := data.Price.Mul(data.Quantity)
        p.CostBasis[data.Symbol] = currentCost.Add(additionalCost)
        
    case "StockSold":
        var data struct {
            Symbol   string          `json:"symbol"`
            Quantity decimal.Decimal `json:"quantity"`
            Price    decimal.Decimal `json:"price"`
        }
        if err := json.Unmarshal(event.Data, &data); err != nil {
            return err
        }
        
        currentQuantity := p.Holdings[data.Symbol]
        p.Holdings[data.Symbol] = currentQuantity.Sub(data.Quantity)
        
        // Adjust cost basis proportionally
        if currentQuantity.GreaterThan(decimal.Zero) {
            percentageSold := data.Quantity.Div(currentQuantity)
            currentCost := p.CostBasis[data.Symbol]
            costReduction := currentCost.Mul(percentageSold)
            p.CostBasis[data.Symbol] = currentCost.Sub(costReduction)
        }
    }
    
    p.Version = event.Version
    return nil
}

// PortfolioService manages portfolios using event sourcing
type PortfolioService struct {
    eventStore EventStore
    logger     *zap.Logger
}

func NewPortfolioService(eventStore EventStore, logger *zap.Logger) *PortfolioService {
    return &PortfolioService{
        eventStore: eventStore,
        logger:     logger,
    }
}

// GetPortfolio rebuilds a portfolio by replaying all events
func (s *PortfolioService) GetPortfolio(ctx context.Context, portfolioID string) (*Portfolio, error) {
    portfolio := &Portfolio{
        Holdings:  make(map[string]decimal.Decimal),
        CostBasis: make(map[string]decimal.Decimal),
    }
    
    var fromVersion int64 = 0
    const batchSize = 100
    
    for {
        events, err := s.eventStore.ReadEvents(ctx, portfolioID, fromVersion, batchSize)
        if err != nil {
            return nil, fmt.Errorf("failed to read events: %w", err)
        }
        
        if len(events) == 0 {
            break
        }
        
        for _, event := range events {
            if err := portfolio.Apply(event); err != nil {
                return nil, fmt.Errorf("failed to apply event: %w", err)
            }
            fromVersion = event.Version
        }
        
        if len(events) < batchSize {
            break
        }
    }
    
    return portfolio, nil
}

// CreatePortfolio creates a new portfolio
func (s *PortfolioService) CreatePortfolio(ctx context.Context, accountID string) (string, error) {
    portfolioID := uuid.New().String()
    
    event := Event{
        Type:     "PortfolioCreated",
        EntityID: portfolioID,
        Data:     []byte(fmt.Sprintf(`{"account_id":"%s"}`, accountID)),
        Metadata: Metadata{
            "user_id": accountID,
        },
    }
    
    err := s.eventStore.AppendEvents(ctx, portfolioID, 0, []Event{event})
    if err != nil {
        return "", fmt.Errorf("failed to append event: %w", err)
    }
    
    return portfolioID, nil
}

// PurchaseStock adds stock to a portfolio
func (s *PortfolioService) PurchaseStock(ctx context.Context, portfolioID string, 
    symbol string, quantity decimal.Decimal, price decimal.Decimal) error {
    
    // First load the current state to get the version
    portfolio, err := s.GetPortfolio(ctx, portfolioID)
    if err != nil {
        return fmt.Errorf("failed to get portfolio: %w", err)
    }
    
    data, err := json.Marshal(struct {
        Symbol   string          `json:"symbol"`
        Quantity decimal.Decimal `json:"quantity"`
        Price    decimal.Decimal `json:"price"`
    }{
        Symbol:   symbol,
        Quantity: quantity,
        Price:    price,
    })
    if err != nil {
        return fmt.Errorf("failed to marshal event data: %w", err)
    }
    
    event := Event{
        Type:     "StockPurchased",
        EntityID: portfolioID,
        Data:     data,
        Metadata: Metadata{
            "transaction_time": time.Now().UTC(),
        },
    }
    
    // Use optimistic concurrency control with the current version
    err = s.eventStore.AppendEvents(ctx, portfolioID, portfolio.Version, []Event{event})
    if err != nil {
        return fmt.Errorf("failed to append event: %w", err)
    }
    
    return nil
}
```

### 7.3 CQRS (Command Query Responsibility Segregation)
Separates read and write operations:
- **Write operations:** Go through the command model  
- **Read operations:** Use specialized query models  
- Different storage mechanisms can be used for each model  
- Read models can be optimized for specific query patterns  

*Example in Fintech:* A trading platform might implement CQRS:
- Commands for placing orders go through a transactional database  
- Queries for market data, portfolio views, and analytics use specialized read models  
- Real-time market data might use in-memory data grids  
- Historical analysis might use columnar databases  

**Code Example:**

```go
// Command represents an instruction to change state
type Command struct {
    ID        string                 `json:"id"`
    Type      string                 `json:"type"`
    Data      map[string]interface{} `json:"data"`
    Metadata  map[string]interface{} `json:"metadata"`
    Timestamp time.Time              `json:"timestamp"`
}

// CommandHandler processes commands
type CommandHandler interface {
    Handle(ctx context.Context, command Command) error
}

// Query represents a request for information
type Query struct {
    ID       string                 `json:"id"`
    Type     string                 `json:"type"`
    Params   map[string]interface{} `json:"params"`
    Metadata map[string]interface{} `json:"metadata"`
}

// QueryHandler processes queries
type QueryHandler interface {
    Handle(ctx context.Context, query Query) (interface{}, error)
}

// EventPublisher broadcasts events to subscribers
type EventPublisher interface {
    Publish(ctx context.Context, event Event) error
}

// OrderCommandHandler processes order commands
type OrderCommandHandler struct {
    eventStore     EventStore
    eventPublisher EventPublisher
    logger         *zap.Logger
}

func NewOrderCommandHandler(
    eventStore EventStore,
    eventPublisher EventPublisher,
    logger *zap.Logger,
) *OrderCommandHandler {
    return &OrderCommandHandler{
        eventStore:     eventStore,
        eventPublisher: eventPublisher,
        logger:         logger,
    }
}

// Handle processes order commands
func (h *OrderCommandHandler) Handle(ctx context.Context, command Command) error {
    switch command.Type {
    case "CreateOrder":
        return h.handleCreateOrder(ctx, command)
    case "PayOrder":
        return h.handlePayOrder(ctx, command)
    case "ShipOrder":
        return h.handleShipOrder(ctx, command)
    default:
        return fmt.Errorf("unknown command type: %s", command.Type)
    }
}

func (h *OrderCommandHandler) handleCreateOrder(ctx context.Context, command Command) error {
    orderID, ok := command.Data["order_id"].(string)
    if !ok || orderID == "" {
        return errors.New("order_id is required")
    }
    
    customerID, ok := command.Data["customer_id"].(string)
    if !ok || customerID == "" {
        return errors.New("customer_id is required")
    }
    
    itemsData, ok := command.Data["items"].([]interface{})
    if !ok || len(itemsData) == 0 {
        return errors.New("items are required")
    }
    
    // Convert items to proper format
    items := make([]map[string]interface{}, 0, len(itemsData))
    for _, item := range itemsData {
        itemMap, ok := item.(map[string]interface{})
        if !ok {
            return errors.New("invalid item format")
        }
        items = append(items, itemMap)
    }
    
    // Create an event
    eventData, err := json.Marshal(map[string]interface{}{
        "customer_id": customerID,
        "items":       items,
        "status":      "created",
        "created_at":  time.Now().UTC(),
    })
    if err != nil {
        return fmt.Errorf("failed to marshal event data: %w", err)
    }
    
    event := Event{
        ID:       uuid.New().String(),
        Type:     "OrderCreated",
        EntityID: orderID,
        Data:     eventData,
        Metadata: map[string]interface{}{
            "command_id": command.ID,
            "user_id":    command.Metadata["user_id"],
        },
    }
    
    // Store the event
    err = h.eventStore.AppendEvents(ctx, "order-"+orderID, 0, []Event{event})
    if err != nil {
        return fmt.Errorf("failed to store event: %w", err)
    }
    
    // Publish the event
    err = h.eventPublisher.Publish(ctx, event)
    if err != nil {
        h.logger.Error("Failed to publish event",
            zap.String("event_type", event.Type),
            zap.String("order_id", orderID),
            zap.Error(err))
    }
    
    return nil
}

func (h *OrderCommandHandler) handlePayOrder(ctx context.Context, command Command) error {
    // Implementation omitted for brevity
    return nil
}

func (h *OrderCommandHandler) handleShipOrder(ctx context.Context, command Command) error {
    // Implementation omitted for brevity
    return nil
}

// OrderQueryHandler processes order queries
type OrderQueryHandler struct {
    db     *sql.DB
    logger *zap.Logger
}

func NewOrderQueryHandler(db *sql.DB, logger *zap.Logger) *OrderQueryHandler {
    return &OrderQueryHandler{
        db:     db,
        logger: logger,
    }
}

// Handle processes order queries
func (h *OrderQueryHandler) Handle(ctx context.Context, query Query) (interface{}, error) {
    switch query.Type {
    case "GetOrder":
        return h.handleGetOrder(ctx, query)
    case "GetOrderHistory":
        return h.handleGetOrderHistory(ctx, query)
    case "GetCustomerOrders":
        return h.handleGetCustomerOrders(ctx, query)
    default:
        return nil, fmt.Errorf("unknown query type: %s", query.Type)
    }
}

func (h *OrderQueryHandler) handleGetOrder(ctx context.Context, query Query) (interface{}, error) {
    orderID, ok := query.Params["order_id"].(string)
    if !ok || orderID == "" {
        return nil, errors.New("order_id is required")
    }
    
    var order struct {
        ID         string          `json:"id"`
        CustomerID string          `json:"customer_id"`
        Status     string          `json:"status"`
        Total      decimal.Decimal `json:"total"`
        CreatedAt  time.Time       `json:"created_at"`
        UpdatedAt  time.Time       `json:"updated_at"`
    }
    
    err := h.db.QueryRowContext(ctx, `
        SELECT id, customer_id, status, total, created_at, updated_at
        FROM order_view
        WHERE id = $1
    `, orderID).Scan(
        &order.ID,
        &order.CustomerID,
        &order.Status,
        &order.Total,
        &order.CreatedAt,
        &order.UpdatedAt,
    )
    if err != nil {
        if err == sql.ErrNoRows {
            return nil, fmt.Errorf("order not found: %s", orderID)
        }
        return nil, fmt.Errorf("failed to query order: %w", err)
    }
    
    // Query order items
    rows, err := h.db.QueryContext(ctx, `
        SELECT product_id, quantity, price
        FROM order_item_view
        WHERE order_id = $1
    `, orderID)
    if err != nil {
        return nil, fmt.Errorf("failed to query order items: %w", err)
    }
    defer rows.Close()
    
    var items []map[string]interface{}
    for rows.Next() {
        var item struct {
            ProductID string
            Quantity  int
            Price     decimal.Decimal
        }
        if err := rows.Scan(&item.ProductID, &item.Quantity, &item.Price); err != nil {
            return nil, fmt.Errorf("failed to scan order item: %w", err)
        }
        
        items = append(items, map[string]interface{}{
            "product_id": item.ProductID,
            "quantity":   item.Quantity,
            "price":      item.Price,
            "subtotal":   item.Price.Mul(decimal.NewFromInt(int64(item.Quantity))),
        })
    }
    
    if err := rows.Err(); err != nil {
        return nil, fmt.Errorf("error iterating order items: %w", err)
    }
    
    return map[string]interface{}{
        "order": order,
        "items": items,
    }, nil
}

func (h *OrderQueryHandler) handleGetOrderHistory(ctx context.Context, query Query) (interface{}, error) {
    // Implementation omitted for brevity
    return nil, nil
}

func (h *OrderQueryHandler) handleGetCustomerOrders(ctx context.Context, query Query) (interface{}, error) {
    // Implementation omitted for brevity
    return nil, nil
}

// OrderProjector updates the read model when events occur
type OrderProjector struct {
    db     *sql.DB
    logger *zap.Logger
}

func NewOrderProjector(db *sql.DB, logger *zap.Logger) *OrderProjector {
    return &OrderProjector{
        db:     db,
        logger: logger,
    }
}

// HandleEvent updates the read model based on events
func (p *OrderProjector) HandleEvent(ctx context.Context, event Event) error {
    switch event.Type {
    case "OrderCreated":
        return p.handleOrderCreated(ctx, event)
    case "OrderPaid":
        return p.handleOrderPaid(ctx, event)
    case "OrderShipped":
        return p.handleOrderShipped(ctx, event)
    default:
        // Ignore unknown events
        return nil
    }
}

func (p *OrderProjector) handleOrderCreated(ctx context.Context, event Event) error {
    var data struct {
        CustomerID string                   `json:"customer_id"`
        Items      []map[string]interface{} `json:"items"`
        Status     string                   `json:"status"`
        CreatedAt  time.Time                `json:"created_at"`
    }
    if err := json.Unmarshal(event.Data, &data); err != nil {
        return fmt.Errorf("failed to unmarshal event data: %w", err)
    }
    
    // Calculate total from items
    total := decimal.Zero
    for _, item := range data.Items {
        price, ok := item["price"].(float64)
        if !ok {
            return errors.New("invalid price format")
        }
        
        quantity, ok := item["quantity"].(float64)
        if !ok {
            return errors.New("invalid quantity format")
        }
        
        itemTotal := decimal.NewFromFloat(price).Mul(decimal.NewFromFloat(quantity))
        total = total.Add(itemTotal)
    }
    
    // Begin transaction
    tx, err := p.db.BeginTx(ctx, nil)
    if err != nil {
        return fmt.Errorf("failed to begin transaction: %w", err)
    }
    defer tx.Rollback()
    
    // Insert order into read model
    _, err = tx.ExecContext(ctx, `
        INSERT INTO order_view (
            id, customer_id, status, total, created_at, updated_at
        ) VALUES ($1, $2, $3, $4, $5, $6)
    `,
        event.EntityID,
        data.CustomerID,
        data.Status,
        total,
        data.CreatedAt,
        time.Now().UTC(),
    )
    if err != nil {
        return fmt.Errorf("failed to insert order: %w", err)
    }
    
    // Insert order items
    stmt, err := tx.PrepareContext(ctx, `
        INSERT INTO order_item_view (
            order_id, product_id, quantity, price
        ) VALUES ($1, $2, $3, $4)
    `)
    if err != nil {
        return fmt.Errorf("failed to prepare statement: %w", err)
    }
    defer stmt.Close()
    
    for _, item := range data.Items {
        productID, ok := item["product_id"].(string)
        if !ok {
            return errors.New("invalid product_id format")
        }
        
        price, ok := item["price"].(float64)
        if !ok {
            return errors.New("invalid price format")
        }
        
        quantity, ok := item["quantity"].(float64)
        if !ok {
            return errors.New("invalid quantity format")
        }
        
        _, err = stmt.ExecContext(ctx,
            event.EntityID,
            productID,
            int(quantity),
            decimal.NewFromFloat(price),
        )
        if err != nil {
            return fmt.Errorf("failed to insert order item: %w", err)
        }
    }
    
    // Update customer's order count
    _, err = tx.ExecContext(ctx, `
        INSERT INTO customer_stats (customer_id, order_count, total_spent)
        VALUES ($1, 1, $2)
        ON CONFLICT (customer_id) DO UPDATE
        SET order_count = customer_stats.order_count + 1,
            total_spent = customer_stats.total_spent + $2
    `,
        data.CustomerID,
        total,
    )
    if err != nil {
        return fmt.Errorf("failed to update customer stats: %w", err)
    }
    
    // Commit transaction
    if err := tx.Commit(); err != nil {
        return fmt.Errorf("failed to commit transaction: %w", err)
    }
    
    return nil
}

func (p *OrderProjector) handleOrderPaid(ctx context.Context, event Event) error {
    // Implementation omitted for brevity
    return nil
}

func (p *OrderProjector) handleOrderShipped(ctx context.Context, event Event) error {
    // Implementation omitted for brevity
    return nil
}
```

### 7.4 Outbox Pattern
Ensures reliable message publishing alongside database transactions:
1. The application stores outgoing messages in an "outbox" table within the same transaction.  
2. A separate process reads the outbox and publishes messages to the message broker.  
3. Once published, messages are marked as processed.

*Example in Fintech:* A payment gateway using the outbox pattern would:
- Store both the payment record and notification message in the same transaction.  
- Guarantee that the notification is sent even if the message broker is temporarily unavailable.  
- Prevent situations where a payment is processed but the notification fails.

---

## 8. Real-world Implementation Examples

### 8.1 Stripe's Distributed Architecture
Stripe, a leading payment processor, handles distributed consistency through:
- **Idempotency keys:** For safe retries of payment operations  
- **Event-driven architecture:** With guaranteed message delivery  
- **Optimistic concurrency control:** For high-throughput operations  
- **Consistent hashing:** For request routing and data partitioning  
- **Specialized data stores:** For different aspects of payment processing  

### 8.2 Netflix's Eventual Consistency Model
Netflix prioritizes availability and partition tolerance:
- **Cassandra clusters:** For highly available, eventually consistent data storage  
- **Chaos Engineering:** To test resilience to network partitions and node failures  
- **Client-side caching with TTLs:** To handle temporary inconsistencies  
- **Background reconciliation processes:** To detect and resolve data conflicts  
- **Circuit breakers:** To prevent cascading failures  

### 8.3 PayPal's Transaction Processing
PayPal balances consistency and availability through:
- **Sharded databases:** With consistent hashing  
- **Two-phase commit:** For critical financial operations  
- **Asynchronous replication:** For non-critical data  
- **Read-after-write consistency guarantees:** For user operations  
- **Specialized fraud detection systems:** Using eventual consistency  

### 8.4 Financial Exchange Architectures
Stock exchanges and cryptocurrency trading platforms require high throughput with strong consistency:
- **In-memory order matching engines:** For performance  
- **Write-ahead logging:** For durability  
- **Multi-datacenter replication:** With synchronous commits for critical paths  
- **Specialized consensus algorithms:** For distributed order books  
- **Lamport timestamps or vector clocks:** To establish event ordering  

---

## 9. Implementing Consistency in Microservices (Fintech Example)

### 9.1 Architecture Overview
A modern fintech payment system might include these components:
- **Account Service:** Manages user account balances  
- **Transaction Service:** Processes payments and transfers  
- **Notification Service:** Sends alerts to users  
- **Compliance Service:** Handles regulatory requirements  
- **Reporting Service:** Generates financial reports and analytics  

### 9.2 Consistency Patterns Implementation

#### 9.2.1 Strong Consistency for Critical Operations
For money transfers between accounts:

**Code Example:**

```go
// TransactionService struct
type TransactionService struct {
    accountService  AccountServiceClient
    transactionRepo TransactionRepository
    eventBus        EventBus
    distributedLock DistributedLock
}

// TransferFunds handles money transfers with strong consistency
func (s *TransactionService) TransferFunds(ctx context.Context, fromAccount, toAccount string, amount decimal.Decimal) (string, error) {
    // Acquire distributed lock on both accounts
    lockKey := fmt.Sprintf("accounts:%s:%s", fromAccount, toAccount)
    unlock, err := s.distributedLock.Lock(ctx, lockKey, 30*time.Second)
    if err != nil {
        return "", fmt.Errorf("failed to acquire lock: %w", err)
    }
    defer unlock()
    
    // Start a database transaction
    tx, err := s.transactionRepo.BeginTx(ctx)
    if err != nil {
        return "", fmt.Errorf("failed to begin transaction: %w", err)
    }
    defer tx.Rollback() // Will be ignored if tx.Commit() is called
    
    // Call Account Service to debit
    debitReq := &pb.DebitRequest{
        AccountId: fromAccount,
        Amount:    amount.String(),
        Reference: "Transfer",
    }
    debitResp, err := s.accountService.Debit(ctx, debitReq)
    if err != nil {
        return "", fmt.Errorf("failed to debit account: %w", err)
    }
    if !debitResp.Success {
        return "", fmt.Errorf("debit failed: %s", debitResp.Reason)
    }
    
    // Call Account Service to credit
    creditReq := &pb.CreditRequest{
        AccountId: toAccount,
        Amount:    amount.String(),
        Reference: "Transfer",
    }
    creditResp, err := s.accountService.Credit(ctx, creditReq)
    if err != nil {
        return "", fmt.Errorf("failed to credit account: %w", err)
    }
    if !creditResp.Success {
        return "", fmt.Errorf("credit failed: %s", creditResp.Reason)
    }
    
    // Record the transaction
    transactionID, err := uuid.NewUUID()
    if err != nil {
        return "", fmt.Errorf("failed to generate transaction ID: %w", err)
    }
    
    err = s.transactionRepo.CreateTransaction(ctx, tx, &Transaction{
        ID:          transactionID.String(),
        FromAccount: fromAccount,
        ToAccount:   toAccount,
        Amount:      amount,
        Status:      "COMPLETED",
        CreatedAt:   time.Now(),
    })
    if err != nil {
        return "", fmt.Errorf("failed to record transaction: %w", err)
    }
    
    // Commit the transaction
    if err := tx.Commit(); err != nil {
        return "", fmt.Errorf("failed to commit transaction: %w", err)
    }
    
    // Publish event for async processing (notifications, etc.)
    event := &TransferCompletedEvent{
        TransactionID: transactionID.String(),
        FromAccount:   fromAccount,
        ToAccount:     toAccount,
        Amount:        amount,
        Timestamp:     time.Now(),
    }
    if err := s.eventBus.Publish(ctx, "TransferCompleted", event); err != nil {
        // Log but don't fail the transfer - event can be republished later
        log.Printf("Failed to publish TransferCompleted event: %v", err)
    }
    
    return transactionID.String(), nil
}
```

#### 9.2.2 Eventual Consistency for Non-Critical Operations
For activity feed updates and notifications:

**Code Example:**

```go
// NotificationService struct
type NotificationService struct {
    userService      UserServiceClient
    notificationRepo NotificationRepository
    activityService  ActivityServiceClient
    retryQueue       RetryQueue
}

// TransferCompletedEvent represents a completed transfer event
type TransferCompletedEvent struct {
    TransactionID string          `json:"transaction_id"`
    FromAccount   string          `json:"from_account"`
    ToAccount     string          `json:"to_account"`
    Amount        decimal.Decimal `json:"amount"`
    Timestamp     time.Time       `json:"timestamp"`
}

// HandleTransferCompleted processes transfer completion notifications
func (s *NotificationService) HandleTransferCompleted(ctx context.Context, event *TransferCompletedEvent) error {
    // Set timeout context to prevent blocking
    timeoutCtx, cancel := context.WithTimeout(ctx, 500*time.Millisecond)
    defer cancel()
    
    // Retrieve user preferences (with timeout to prevent blocking)
    senderPrefsReq := &pb.GetNotificationPreferencesRequest{
        UserId: event.FromAccount,
    }
    senderPrefs, err := s.userService.GetNotificationPreferences(timeoutCtx, senderPrefsReq)
    if err != nil {
        // Use default preferences if unable to fetch
        log.Printf("Failed to get sender preferences: %v. Using defaults.", err)
        senderPrefs = &pb.NotificationPreferences{
            TransferNotifications: true,
            PreferredChannels:     []string{"email"},
        }
    }
    
    receiverPrefsReq := &pb.GetNotificationPreferencesRequest{
        UserId: event.ToAccount,
    }
    receiverPrefs, err := s.userService.GetNotificationPreferences(timeoutCtx, receiverPrefsReq)
    if err != nil {
        // Use default preferences if unable to fetch
        log.Printf("Failed to get receiver preferences: %v. Using defaults.", err)
        receiverPrefs = &pb.NotificationPreferences{
            TransferNotifications: true,
            PreferredChannels:     []string{"email"},
        }
    }
    
    // Send notifications based on preferences
    var wg sync.WaitGroup
    
    if senderPrefs.TransferNotifications {
        wg.Add(1)
        go func() {
            defer wg.Done()
            notification := &Notification{
                UserID:      event.FromAccount,
                Message:     fmt.Sprintf("Your transfer of $%s to account %s is complete.", event.Amount, event.ToAccount),
                Channels:    senderPrefs.PreferredChannels,
                ReferenceID: event.TransactionID,
                CreatedAt:   time.Now(),
            }
            if err := s.notificationRepo.CreateNotification(ctx, notification); err != nil {
                log.Printf("Failed to create sender notification: %v", err)
            }
        }()
    }
    
    if receiverPrefs.TransferNotifications {
        wg.Add(1)
        go func() {
            defer wg.Done()
            notification := &Notification{
                UserID:      event.ToAccount,
                Message:     fmt.Sprintf("You received $%s from account %s.", event.Amount, event.FromAccount),
                Channels:    receiverPrefs.PreferredChannels,
                ReferenceID: event.TransactionID,
                CreatedAt:   time.Now(),
            }
            if err := s.notificationRepo.CreateNotification(ctx, notification); err != nil {
                log.Printf("Failed to create receiver notification: %v", err)
            }
        }()
    }
    
    // Update activity feeds (non-critical, can be eventually consistent)
    wg.Add(2)
    go func() {
        defer wg.Done()
        activityReq := &pb.RecordActivityRequest{
            UserId:    event.FromAccount,
            Type:      "transfer_sent",
            Data:      marshalEvent(event),
            Timestamp: timestamppb.New(event.Timestamp),
        }
        if _, err := s.activityService.RecordActivity(ctx, activityReq); err != nil {
            // Log and retry later - don't block notification delivery
            s.retryQueue.Enqueue(ctx, "record_activity", activityReq, 0)
        }
    }()
    
    go func() {
        defer wg.Done()
        activityReq := &pb.RecordActivityRequest{
            UserId:    event.ToAccount,
            Type:      "transfer_received",
            Data:      marshalEvent(event),
            Timestamp: timestamppb.New(event.Timestamp),
        }
        if _, err := s.activityService.RecordActivity(ctx, activityReq); err != nil {
            // Log and retry later - don't block notification delivery
            s.retryQueue.Enqueue(ctx, "record_activity", activityReq, 0)
        }
    }()
    
    wg.Wait()
    return nil
}

// Helper to marshal event to protobuf Any
func marshalEvent(event *TransferCompletedEvent) *anypb.Any {
    // Implementation details omitted
    return &anypb.Any{}
}
```

#### 9.3 Handling Consistency Failures
When service calls fail, compensating actions maintain consistency.

**Code Example:**

```go
// TransactionRecoveryService struct
type TransactionRecoveryService struct {
    transactionRepo     TransactionRepository
    accountService      AccountServiceClient
    notificationService NotificationServiceClient
    logger              *log.Logger
}

// RecoverIncompleteTransactions implements scheduled task pattern
func (s *TransactionRecoveryService) RecoverIncompleteTransactions(ctx context.Context) error {
    // Find transactions that are stuck in an incomplete state
    incompleteTransactions, err := s.transactionRepo.FindIncomplete(ctx, time.Now().Add(-15*time.Minute))
    if err != nil {
        return fmt.Errorf("failed to find incomplete transactions: %w", err)
    }
    
    for _, txn := range incompleteTransactions {
        if txn.Status == "DEBIT_COMPLETED" {
            // Credit was never completed, need to reverse the debit
            s.logger.Printf("Recovering transaction %s by reversing debit", txn.ID)
            
            req := &pb.CreditRequest{
                AccountId: txn.FromAccount,
                Amount:    txn.Amount.String(),
                Reference: fmt.Sprintf("REVERSAL-%s", txn.ID),
            }
            
            resp, err := s.accountService.Credit(ctx, req)
            if err != nil {
                s.logger.Printf("Failed to reverse transaction %s: %v", txn.ID, err)
                
                alertReq := &pb.SendAdminAlertRequest{
                    Message: fmt.Sprintf("URGENT: Manual intervention needed for transaction %s", txn.ID),
                    Level:   "HIGH",
                }
                if _, alertErr := s.notificationService.SendAdminAlert(ctx, alertReq); alertErr != nil {
                    s.logger.Printf("Failed to send admin alert: %v", alertErr)
                }
                
                continue
            }
            
            if !resp.Success {
                s.logger.Printf("Failed to reverse transaction %s: %s", txn.ID, resp.Reason)
                
                alertReq := &pb.SendAdminAlertRequest{
                    Message: fmt.Sprintf("URGENT: Manual intervention needed for transaction %s: %s", txn.ID, resp.Reason),
                    Level:   "HIGH",
                }
                if _, alertErr := s.notificationService.SendAdminAlert(ctx, alertReq); alertErr != nil {
                    s.logger.Printf("Failed to send admin alert: %v", alertErr)
                }
                
                continue
            }
            
            // Update transaction status
            if err := s.transactionRepo.UpdateStatus(ctx, txn.ID, "REVERSED"); err != nil {
                s.logger.Printf("Failed to update transaction status: %v", err)
                continue
            }
            
            alertReq := &pb.SendAdminAlertRequest{
                Message: fmt.Sprintf("Transaction %s was automatically reversed after timeout", txn.ID),
                Level:   "MEDIUM",
            }
            if _, alertErr := s.notificationService.SendAdminAlert(ctx, alertReq); alertErr != nil {
                s.logger.Printf("Failed to send admin alert: %v", alertErr)
            }
        }
    }
    
    return nil
}
```

---

## 10. Best Practices for Distributed Database Consistency

### 10.1 Design Principles
- **Partition by Business Context:** Organize data to minimize cross-partition transactions  
- **Identify Consistency Requirements:** Not all operations need strong consistency  
- **Design for Failure:** Assume network partitions and node failures will happen  
- **Use Idempotent Operations:** Ensure operations can be safely retried  
- **Implement Compensating Transactions:** For rolling back distributed operations  

### 10.2 Implementation Techniques
- **Use Distributed Tracing:** Track requests as they flow through services  
- **Implement Health Checks:** Detect node failures quickly  
- **Apply Circuit Breakers:** Prevent cascading failures  
- **Set Appropriate Timeouts:** Balance between responsiveness and success rate  
- **Monitor Consistency Metrics:** Track replication lag and conflict rates  

### 10.3 Testing Strategies
- **Chaos Engineering:** Intentionally introduce failures to test resilience  
- **Jepsen Tests:** Verify consistency guarantees under network partitions  
- **Simulated Load Testing:** Validate system behavior under high concurrency  
- **Long-Running Tests:** Some consistency issues only appear after sustained operation  
- **Data Reconciliation Tests:** Verify eventual consistency actually converges  

---

## Conclusion

Building distributed systems that maintain data consistency requires a deep understanding of database fundamentals, transaction isolation, and consistency patterns. By choosing appropriate consistency models for different operations and implementing robust error handling, modern applications can achieve both reliability and performance.

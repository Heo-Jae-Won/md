## <span style="color:#802548">_isolation level_</span>
- isolation level is designed for managing concurrency conflict
    - it's based mainly on locks and MVCC(it's different from optimistic lock or pessimistic lock)
    - lock is for changes to row, so let user wait 
    - "Wait until I'm done."
        - Shared lock (S) --> almost not used, optimistic lock is preferred. only in SQL SERVER select, S lock is used
        - Exclusive lock (X)
        - Range or gap lock (range condition in where using index)
        - Predicate lock ()
    - MVCC is for reading row, so let user read previous version
    - "Here's an older version, so you don't need to wait."
    - just locking all rows when DML harm responsiviness of server so MVCC is needed
        - MYSQL -> undo log based
        - PostgreSQL -> Old row versions in table based
            - but for each rows, there is a metadata like transactionID, so physically 2 rows created but only 1 row is visible
            - obsolete rows are removed by VACCUMM(like GC)
        - SQL SERVER -> version store based like temp db 
        - Oracle -> segment based like undo log
- basically, update/delete uses X lock regardingless of isolation level
    - but we can access to that rows using MVCC, when another transaction executing DML
    - for select for update, X lock is triggered
    - for read committed isolation level, select for update triggers only X lock
    - for repetable read or serializable isolation level, select for update triggers range/predicate lock, so insert on rage or where condition become impossible
        - but if no proper index existing, lock escalation can be occured
        - it means all table is locked
- then why modern web prefer read committed ?
    - High Concurrency
        - It maximizes throughput. 
        - no locking for same table even multiple request
    - No Long-Lived Database Locks: 
        - It only locks data for the brief millisecond a write query is executing
        - This keeps connection pools open and free.
    - Deadlock Prevention
        - Higher isolation levels releates with crashes or deadlocks under high traffic

## <span style="color:#802548">_pessimistic lock_</span>
- pessimistic lock usually use 'select for update'

```sql
select *
from table
FOR UPDATE
```

- redis is also possible way to execute pessimistic lock
    - Distributed Memory Locks in multiple servers config

```C#
using RedLockNet;

public class WalletService
{
    private readonly IDistributedLockFactory _lockFactory;
    private readonly MyDbContext _context;
    private readonly ILogger<WalletService> _logger;

    public WalletService(IDistributedLockFactory lockFactory, MyDbContext context, ILogger<WalletService> logger)
    {
        _lockFactory = lockFactory;
        _context = context;
        _logger = logger;
    }

    public async Task ProcessTransferAsync(int userId, decimal amountToDeduct)
    {
        // Define a unique, consistent lock key name based on the specific resource ID
        string lockKey = $"lock:user:wallet:{userId}";
        
        // ExpiryTime: Safety net. If this server crashes mid-execution, 
        // Redis will auto-release the key after 30 seconds.
        TimeSpan expiryTime = TimeSpan.FromSeconds(30); 
        
        // WaitTime: How long to block and retry if another server currently owns the lock
        TimeSpan waitTime = TimeSpan.FromSeconds(10);   
        
        // RetryTime: How long to wait between internal attempts to check the lock status
        TimeSpan retryTime = TimeSpan.FromSeconds(1);  

        // Acquire the distributed lock across all servers via Redis
        await using (var redLock = await _lockFactory.CreateLockAsync(lockKey, expiryTime, waitTime, retryTime))
        {
            // Always verify if the lock was successfully acquired
            if (redLock.IsAcquired)
            {
                _logger.LogInformation($"Lock acquired for User {userId}. Processing transfer...");
                
                // --- CRITICAL BUSINESS SECTION ---
                // Only ONE server across your whole load balancer can be here at a time.
                var wallet = await _context.Wallets.FindAsync(userId);
                
                if (wallet.Balance >= amountToDeduct)
                {
                    wallet.Balance -= amountToDeduct;
                    await _context.SaveChangesAsync();
                }
                else
                {
                    throw new InvalidOperationException("Insufficient funds.");
                }
                // --- END CRITICAL SECTION ---
            }
            else
            {
                // The wait time expired, meaning another server is taking too long
                _logger.LogWarning($"Could not acquire lock for User {userId} within the timeout limit.");
                throw new TimeoutException("System is busy processing your request. Please try again.");
            }
        } // 💥 The lock is automatically released here immediately across all servers!
    }
}

```

## <span style="color:#802548">_lock escalation in SQL SERVER and chunk process_</span>
- basically, pessimistic lock used in RDBMS locks specific rows if using index scan
- but if using table scan, then lock escalates to table lock 
- it applies to every where clause including update or delete or select for update(in summary, with X lock)
- lock escalation is not for data safety, it's for memory management
- for example, sql server consider table locking when fetching more than 5,000 rows
    - especially for repetable read and serializble, high possibility

- bulk insert also can trigger lock escalation
- then why SQL Server consider lock escalation ?
    - SQL SERVER uses RAM to keep lock info, not disk like Oracle/Postgre
    - locking and unlocking rows is incredibly fast but many row lock means RAM shortage
        - 100,000 locks × 96 bytes ≈ 9.6 Megabytes of ram is needed for single query
        - convert it table lock means 1 locks. so RAM is freed
- for sure we can stop lock_ESCALATION

```sql
ALTER TABLE your_table_name SET (LOCK_ESCALATION = DISABLE);
```

- but disable LOCK_ESCALATION is not first option when problem emerges
- especially, for batch process, chunk process is 1st option for considering
    - chunk Keeps number of Locks Low
        - for server, RAM shortage problem doesnt occur
- In SQL SERVER, default select is S lock
- so, it can cause table lock or select/update blocking, especially in batching process

```
chunk process transaction in for each toward a temp table
if success all, insert all temp records to original table
if fails, delete all temp records
```


- there is another option for prevent lock escalating
- in many service using SQL SERVER, RCSI is turned on
    - when READ_COMMITTED_SNAPSHOT(RCSI) is on, select dont use S lock
    - instead, select use Row Version, which means lock escalation would not occur often
    - so lock escalation dont occur, but many row lock has a memory RAM shortage problem
        - so chunk batching process is recommended


## <span style="color:#802548">_optimistic lock_</span>
- optimistic lock usaually use 'version column'
    - version column with int value is better than timestamp
    - if u must have status column, u can alter status column(SHIPPED, PROCESSING)
- why optimistic lock using version is better than timestamp ? 
    - Atomic Execution
        - version = version + 1 can protect atomic execution
        - but precalculated version harms atomicity( like set version = 6)
    - No concurrency problem
        - timestamp cannot deal nanosecond, but version can
    - Guaranteed Precision
        - A sequential number increment (1 to 2) is absolute and never suffers from precision loss.
    - Immune to network issue
        - Servers rely on network protocols
        - it can generate a timestamp that is older than the previous record. 
    - Simpler Data Types and Comparison
        - Comparing two integers is computationally cheaper
        - less prone to formatting bugs
    - no local time problem, no use to consider UTC time


```sql
UPDATE products 
SET stock = 45, version = version + 1 
WHERE id = 123 AND version = 5;
```

- but how to convey version from front? 
    - In the Request Body
    - In the HTTP Headers



## <span style="color:#802548">_how to avoid phantom read_</span>
- optimistic/pessimistic lock cannot prevent insert

```
Transaction A (Manager App)                  Transaction B (Customer App)
---------------------------                  ---------------------------
1. START TRANSACTION;

2. SELECT COUNT(*) FROM orders 
   WHERE zone = 'Zone-A' 
   AND status = 'PENDING';
   -- Database returns: 9 orders.
   -- (Perfect! There is room for 1 more.)

                                             3. START TRANSACTION;
                                             4. INSERT INTO orders (id, zone, status) 
                                                VALUES (999, 'Zone-A', 'PENDING');
                                             5. COMMIT;
                                             -- A brand new order is saved.

6. SELECT COUNT(*) FROM orders 
   WHERE zone = 'Zone-A' 
   AND status = 'PENDING';
   -- Database returns: 10 orders!
   -- (The new row "phantoms" into view 
   -- because Transaction B committed).
```


- for preventing insert, below is recommended
    - UK
    - "Upsert" Strategies (ON CONFLICT / MERGE)
- for upsert, Oracle and SQL Server doesnt support it fully
- so temp table is needed

```sql
-- Step 1: Bulk insert your new rows into a temporary staging table
INSERT INTO #StagingProducts (sku, stock) VALUES ('A', 10), ('B', 20);

-- Step 2: Use MERGE with the staging table as the source
MERGE target_products AS Target
USING #StagingProducts AS Source
ON (Target.sku = Source.sku)
WHEN MATCHED THEN
    UPDATE SET Target.stock = Source.stock
WHEN NOT MATCHED THEN
    INSERT (sku, stock) VALUES (Source.sku, Source.stock);
```

- then what about count limit ? 
- let's suppose compnay take holds a event for first 80 visitor can get coupon
- so people try to get a coupon, which means insert. 
- but in this situation, the transactionA first time select did not reflect transactionB insert. 
- but right before insert of trasactionA, limit touches. so he cannot get coupon. in this case, phantom read is recommdend ? or is there other way to prevent this kinda concurrency problem ?
    - Pre-Generated Coupons with Pessimistic Locking (Highly Recommended) is needed
    - select count is not recommended 
    - treat the coupons like physical movie tickets that already exist in the database.
        - Pre-insert exactly 80 rows into a coupons table with user_id = NULL and an auto-incrementing id from 1 to 80.
        - The Logic: When a user claims a coupon, you grab just one available row and lock it so no one else can grab the same one.

```sql
/* select count(*) is not good*/
insert 80 rows before coupon event
-- when visitor came
SELECT id FROM coupons 
WHERE user_id IS NULL 
ORDER BY id ASC 
LIMIT 1 
FOR UPDATE SKIP LOCKED;

update available row
```


- another option is single counter table

```sql
-- This forces all concurrent transactions to line up single-file
SELECT current_count FROM coupon_counters WHERE coupon_id = 'PROMO80' FOR UPDATE;

-- In your application: 
-- If current_count >= 80, abort!
-- If current_count < 80, proceed:
UPDATE coupon_counters SET current_count = current_count + 1 WHERE coupon_id = 'PROMO80';
INSERT INTO claimed_coupons (user_id) VALUES (?);
```

- for High Traffic, redis is needed

```C#
// Atomic increment happens instantly in memory
const currentCount = await redis.incr("coupon_counter");

if (currentCount > 80) {
    return "Sorry, coupons are sold out!";
}

// Safe to insert into the database asynchronously or synchronously
await db.query("INSERT INTO claimed_coupons ..."); 
```
---------

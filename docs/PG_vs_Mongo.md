# PostgreSQL vs. MongoDB
## Postgres
### Set up database
```
postgres=# create database test;
CREATE DATABASE
postgres=# \c test
You are now connected to database "test" as user "postgres".
```
```
test=# DROP TABLE IF EXISTS estados CASCADE;
DROP TABLE IF EXISTS transaction;

-- Create the first table, transaction
CREATE TABLE transaction (
    id SERIAL PRIMARY KEY,  
    fecha DATE NOT NULL,
    bicOrdenante BIGINT NOT NULL,
    bicBeneficiario BIGINT NOT NULL,
    referencia VARCHAR(255),
    endToEndId VARCHAR(35),
    instrId VARCHAR(35),
    ibanOrdenante VARCHAR(34) NOT NULL,
    ibanBeneficiario VARCHAR(34) NOT NULL
);

-- Create the second table, estados with a foreign key reference to transaction
CREATE TABLE estados (
    id SERIAL PRIMARY KEY, 
    transaction_id INT NOT NULL,  
    estado VARCHAR(255) NOT NULL,
    timestamp TIMESTAMP NOT NULL,
    xml TEXT NOT NULL,
    FOREIGN KEY (transaction_id) REFERENCES transaction(id) ON DELETE CASCADE  
);
```
```
my_database=# DO $$
DECLARE
    i INT;
BEGIN
    FOR i IN 1..10000 LOOP
        INSERT INTO transaction (fecha, bicOrdenante, bicBeneficiario, referencia, endToEndId, instrId, ibanOrdenante, ibanBeneficiario)
        VALUES (
            CURRENT_DATE - (random() * 365)::int,  -- Random date within the last year
            (8000000000 + (random() * 1000000000)::BIGINT),   -- Random BIC as BIGINT
            (8000000000 + (random() * 1000000000)::BIGINT),   -- Random BIC as BIGINT
            substr(md5(random()::text), 1, 255),   -- Random reference string (255 chars)
            substr(md5(random()::text), 1, 35),    -- Random EndToEndId (35 chars)
            substr(md5(random()::text), 1, 35),    -- Random InstrId (35 chars)
            'GB29NWBK60161331926819' || i::text,   -- Fixed length IBAN
            'GB29NWBK60161331926819' || (i + 100)::text -- Fixed length IBAN with offset
        );
    END LOOP;
END $$;

DO $$
DECLARE
    trans_id INT;
    estado_count INT;
    j INT;
BEGIN
    FOR trans_id IN 1..10000 LOOP
        -- Generate a random number of estados (between 1 and 18)
        estado_count := (random() * 18 + 1)::int;

        FOR j IN 1..estado_count LOOP
            INSERT INTO estados (transaction_id, estado, timestamp, xml)
            VALUES (
                trans_id,
                substr(md5(random()::text), 1, 255),
                CURRENT_TIMESTAMP - (random() * 1000000)::int * INTERVAL '1 second',
                '<?xml version="1.0"?><data>Random XML data for estado ' || j || '</data>'
            );
        END LOOP;
    END LOOP;
END $$;
```

### Query database
```
psql -d my_database -c "\timing; SELECT * FROM transaction t LEFT JOIN estados e ON t.id = e.transaction_id;" > /dev/null
```

## MongoDB
### Set up database
```
use my_database;
db.transactions.drop();
for (let i = 1; i <= 10000; i++) {
    // Generate random number of estados (between 1 and 18)
    const estadoCount = Math.floor(Math.random() * 18) + 1;

    // Initialize the estados array
    const estados = [];

    for (let j = 0; j < estadoCount; j++) {
        estados.push({
            estado: Math.random().toString(36).substring(2, 15), // Random estado string
            timestamp: new Date(Date.now() - (Math.random() * 10000000)), // Random timestamp
            xml: `<data>Random XML data for estado ${j + 1}</data>` // Sample XML
        });
    }

    // Create a transaction document
    const transaction = {
        fecha: new Date(Date.now() - (Math.random() * 31536000000)), // Random date within the last year
        bicOrdenante: Math.floor(8000000000 + Math.random() * 1000000000), // Random BIC as BIGINT
        bicBeneficiario: Math.floor(8000000000 + Math.random() * 1000000000), // Random BIC as BIGINT
        referencia: Math.random().toString(36).substring(2, 15), // Random reference string
        endToEndId: Math.random().toString(36).substring(2, 15), // Random EndToEndId
        instrId: Math.random().toString(36).substring(2, 15), // Random InstrId
        ibanOrdenante: `GB29NWBK60161331926819${i}`, // Fixed length IBAN
        ibanBeneficiario: `GB29NWBK60161331926819${i + 100}`, // Fixed length IBAN with offset
        estados: estados // Embedded estados array
    };

    // Insert the transaction record into the collection
    db.transactions.insert(transaction);
}
```

### Query database
```
let start = new Date();
db.transactions.find({}).limit(0).toArray();  // This doesn’t retrieve any records but allows measuring
let end = new Date();
print(`Query time: ${end - start} ms`);  // Show query time
```

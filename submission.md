# Covalent One Million Wallets RSK SQL Track

Submission by #isonar#5236

## Dashboard #1: RSK Mainnet Data

Metabase: https://isonar.herokuapp.com/public/dashboard/8292960b-28de-4625-bc12-c6d9cf6cf881

### Block Producing Speeds, per miner

```
select
       bt.producer as miner,
       count(bt.producer) as total_blocks_mined,
       avg(block_time) as avg_block_time
from (
         select '0x' || encode(lag(miner, 1) over (order by id DESC), 'hex')
                as producer,
                extract(seconds FROM (
                                    lag(signed_at, 1) over (order by id DESC)
                                         )  - signed_at
                    )                               as block_time
         from chain_rsk_mainnet.blocks
     ) bt
where bt.producer is not null
group by bt.producer
order by avg_block_time desc
```

### Block Times, sec

```
select
        lag(height, 1) over (order by id DESC) as block,
        extract(seconds FROM (lag(signed_at, 1) over (order by id DESC)) - signed_at) +
        extract(minutes FROM (lag(signed_at, 1) over (order by id DESC)) - signed_at) * 60
        as block_time

from chain_rsk_mainnet.blocks
limit 2000;
```

### Extended Staking Duration Events

```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Month",
    count(*)
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x5684a06CaB22Db16d901fEe2A5C081b4C91eA40e'
  AND e.topics[1] = '\xea2a9a2c14c575b2781faf01216ebc102565aeb3d85294480b7452371cf404c6'
GROUP BY 1
ORDER BY 1;
```

### Total Staked, per month

```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Month",
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Staked"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x5684a06CaB22Db16d901fEe2A5C081b4C91eA40e'
  AND e.topics[1] = '\x0fc5b1bac0416800b42a669229a346b6e5a15db3896339dbdc5fa376e1e4570a'
GROUP BY 1
ORDER BY 1
```

### Whale Transactions
```
select
       '0x' || encode(tx.hash, 'hex') as "Transaction ID",
       b.height as "Block #",
       '0x' || encode(lag(tx."from", 1) over (order by tx.id DESC), 'hex') as "Sender",
       '0x' || encode(lag(tx."to", 1) over (order by tx.id DESC), 'hex') as "Recipient",
       tx.value / 10^18 as "rBTC"
from chain_rsk_mainnet.block_transactions tx inner join chain_rsk_mainnet.blocks b
on tx.block_id = b.id
where
      tx.value / 10^18 > 1
  and
      b.height > 3000000
  and
      tx.successful
order by value desc
limit 100;
```

## Dashboard #2: Liquidity Pool Data

Metabase: https://isonar.herokuapp.com/public/dashboard/1b8b38f3-808a-4835-a7d5-8dc03b66e906

### DOC-RBTC Liquidity Added
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'DOC'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Added"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\xd715192612F03D20BaE53a5054aF530C9Bb0fA3f'
  AND e.topics[1] = '\x4a1a2a6176e9646d9e3157f7c2ab3c499f18337c0b0828cfb28e0a61de4a11f7'
GROUP BY 1, 2
ORDER BY 1;
```

### DOC-RBTC Liquidity Removed
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'DOC'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Removed"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\xd715192612F03D20BaE53a5054aF530C9Bb0fA3f'
  AND e.topics[1] = '\xbc7d19d505c7ec4db83f3b51f19fb98c4c8a99922e7839d1ee608dfbee29501b'
GROUP BY 1, 2
ORDER BY 1;
```

### ETH-RBTC Liquidity Added, per month
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Month",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'ETH'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Added"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\xcef26b429e272960d8fa2ea190b06df5dd8f68e2'
  AND e.topics[1] = '\x4a1a2a6176e9646d9e3157f7c2ab3c499f18337c0b0828cfb28e0a61de4a11f7'
GROUP BY 1, 2
ORDER BY 1;
```

### FISH-RBTC Liquidity Added
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'FISH'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Added"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\xe731DA93034D769c2045B1ee137D42E1Aa23C18e'
  AND e.topics[1] = '\x4a1a2a6176e9646d9e3157f7c2ab3c499f18337c0b0828cfb28e0a61de4a11f7'
GROUP BY 1, 2
ORDER BY 1;
```

### FISH-RBTC Liquidity Removed

```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'FISH'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Removed"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\xe731DA93034D769c2045B1ee137D42E1Aa23C18e'
  AND e.topics[1] = '\xbc7d19d505c7ec4db83f3b51f19fb98c4c8a99922e7839d1ee608dfbee29501b'GROUP BY 1, 2
ORDER BY 1;
```

### MOC-RBTC Liquidity Added

```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'MOC'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Added"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x34031D1cd14e2C80B0268B47eFf49643375aFaeb'
  AND e.topics[1] = '\x4a1a2a6176e9646d9e3157f7c2ab3c499f18337c0b0828cfb28e0a61de4a11f7'
GROUP BY 1, 2
ORDER BY 1;
```

### MOC-RBTC Liquidity Removed
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'MOC'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Removed"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x34031D1cd14e2C80B0268B47eFf49643375aFaeb'
  AND e.topics[1] = '\xbc7d19d505c7ec4db83f3b51f19fb98c4c8a99922e7839d1ee608dfbee29501b'
GROUP BY 1, 2
ORDER BY 1;
```

### RIF-RBTC Liquidity Added
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'RIF'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Added"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x1769044CBa7aD37719badE16Cc71EC3f027b943D'
  AND e.topics[1] = '\x4a1a2a6176e9646d9e3157f7c2ab3c499f18337c0b0828cfb28e0a61de4a11f7'
GROUP BY 1, 2
ORDER BY 1;
```

### RIF-RBTC Liquidity Removed
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'RIF'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Removed"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x1769044CBa7aD37719badE16Cc71EC3f027b943D'
  AND e.topics[1] = '\xbc7d19d505c7ec4db83f3b51f19fb98c4c8a99922e7839d1ee608dfbee29501b'
GROUP BY 1, 2
ORDER BY 1;
```

### RUSDT-RBTC Liquidity Added
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'RUSDT'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Added"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x448c2474b255576554EeD36c24430ccFac131cE3'
  AND e.topics[1] = '\x4a1a2a6176e9646d9e3157f7c2ab3c499f18337c0b0828cfb28e0a61de4a11f7'
GROUP BY 1, 2
ORDER BY 1;
```
### RUSDT-RBTC Liquidity Removed
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Period",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'RUSDT'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Removed"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x448c2474b255576554EeD36c24430ccFac131cE3'
  AND e.topics[1] = '\xbc7d19d505c7ec4db83f3b51f19fb98c4c8a99922e7839d1ee608dfbee29501b'
GROUP BY 1, 2
ORDER BY 1;
```

### SOV-RBTC Liquidity added
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Month",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'SOV'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Added"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x3fd679b01ddab34da8f72b7ec301aa75ea25f338'
  AND e.topics[1] = '\x4a1a2a6176e9646d9e3157f7c2ab3c499f18337c0b0828cfb28e0a61de4a11f7'
GROUP BY 1, 2
ORDER BY 1;
```

### XUSD-RBTC Liquidity Added
```
SELECT
    to_Date(extract(month from e.block_signed_at)::varchar ||' '|| extract(year from e.block_signed_at)::varchar, 'mm YYYY') as "Month",
    CASE
        WHEN '0x' || encode(extract_address(e.topics[3]), 'hex') = lower('0x542fDA317318eBF1d3DEAf76E0b632741A7e677d') THEN 'RBTC'
        ELSE 'XUSD'
    END as logged__reservetoken,
    sum(cast(abi_field(e.data, 0) as numeric)) / 10^18  AS "Total Liquidity Added"
FROM chain_rsk_mainnet.block_log_events e
WHERE
        e.sender = '\x029448377a56c15928ec783baf6ca736ed99a57f'
  AND e.topics[1] = '\x4a1a2a6176e9646d9e3157f7c2ab3c499f18337c0b0828cfb28e0a61de4a11f7'
GROUP BY 1, 2
ORDER BY 1;
```

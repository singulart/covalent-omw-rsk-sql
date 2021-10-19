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


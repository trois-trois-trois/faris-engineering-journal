# faris-engineering-journal
SDC journal

---

## February 4th, 2019
**1. Choosing FEC Project and Service**
- The chosen project for our SDC group is the ESPN NFL Team Page. I'll be working on the standings service that was originally made by Kevin Phung. This service renders the NFL Team standings within the different divisions of the league. There are two components to this service that are rendered to the page:

  - **Full Standings Component**: This component will render all of the NFL team standings within each sub division of the league. This component isn't rendered onto the main page but is accessed through the standings component. Sub divisions are hard coded, but the information from the database that is rendered to this page include:

    - Team name
    - Team logo
    - Wins
    - Losses
    - Ties
    - Percentage
    - Points for
    - Points against
    - Difference

  ![full standings][one]

  - **Standings Component**: This component renders one sub division of the Full Standings Component to the main page, which is always the NFC WEST division. The information from the database that gets rendered is the same as the Full Standings Component.

  ![standings][two]

**2. Choosing SQL DBMS**
- For my SQL DBMS choice I've chosen to implement a PostgreSQL database. I archived all of Kevin's MongoDB files with an underscore extension in case I'll need it again. Implementing the PostgreSQL was not difficult due to my previous knowledge in implementing one during the FEC project for my service. To utilize PostgresSQL I installed it with homebrew and used the PG client paired with KnexJS and Bookshelf to implement the database easily. I tested an insertion of the same mock data that Kevin created with my new database and it worked successfully. The next step will be to create a script that can insert 10 million records.

**3. Generating 10,000,000 records**
- I've completed the seed script that will execute the records to my SQL database. In summary I created a function that generates the data using a for loop. The only parameter supplied is how many records are to be generated. For quick record generation I used faker, but I'll need to edit the types of images that are supplied for the team logo because faker doesn't have a good method to supply team logo images. What I'll probably end up doing is hosting the sports team logos on S3 and randomize the selection so essentially each record looks more like a sport team. The number of wins,losses, and ties generated by faker also seem extreme so I'll probably tweak this later too.

- Generating 1 million records proved to be a success. It took about 2 minutes and 21 seconds to complete. At about 1.5 million records is when I get an allocation failure. I've copied the full error message below for reference.

```
<--- JS stacktrace --->

==== JS stack trace =========================================

Security context: 0xfc4822cf781 <JS Object>
    2: randomWord [/Users/fh/sdc/kevin-services-standings/node_modules/faker/lib/random.js:~116] [pc=0x30e249a6abcb] (this=0x1844f5d85be1 <a Random with map 0x1897ee7ef459>,type=0xfc482204381 <undefined>)
    3: arguments adaptor frame: 0->1
    4: generateDataSet [/Users/fh/sdc/kevin-services-standings/database/seeds/seed.js:~11] [pc=0x30e249a74b29] (this=0x22b8e5698c19 <JS Global Object>,numOf...

FATAL ERROR: CALL_AND_RETRY_LAST Allocation failed - JavaScript heap out of memory
 1: node::Abort() [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 2: node::FatalException(v8::Isolate*, v8::Local<v8::Value>, v8::Local<v8::Message>) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 3: v8::internal::V8::FatalProcessOutOfMemory(char const*, bool) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 4: v8::internal::Factory::NewFixedArray(int, v8::internal::PretenureFlag) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 5: v8::internal::TypeFeedbackVector::New(v8::internal::Isolate*, v8::internal::Handle<v8::internal::TypeFeedbackMetadata>) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 6: v8::internal::(anonymous namespace)::Ensurefactor: test 10m records. Fails at 1.5m
reFeedbackVector(v8::internal::CompilationInfo*) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 7: v8::internal::(anonymous namespace)::GenerateBaselineCode(v8::internal::CompilationInfo*) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 8: v8::internal::(anonymous namespace)::GetUnoptimizedCodeCommon(v8::internal::CompilationInfo*) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
 9: v8::internal::Compiler::Compile(v8::internal::Handle<v8::internal::JSFunction>, v8::internal::Compiler::ClearExceptionFlag) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
10: v8::internal::Runtime_CompileLazy(int, v8::internal::Object**, v8::internal::Isolate*) [/Users/fh/.nvm/versions/node/v6.13.1/bin/node]
11: 0x30e2493092a7
```
- I'll need to perform some fine-tuning with my seed script. One idea I have is turning the seed function into an async function so that records are inserted one at a time, but this would require me to refactor the code that uses knex's bulkInsert method, which I find to be helpful overall. I'll need to play around some more to figure out how I can insert 10 million records in about 50 minutes.

- I successfully completed the insertion of 10m records! The official time is 24min8s56ms. They key was chaining knex's batch insert in promise format.

```
exports.seed = knex => knex('standings').del()
  .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000)
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000))
    .then(() => knex.batchInsert('standings', generateDataSet(1000000), 1000)));

```

- I met with my team member Amit and we had discussed how chaining insertions might be the key. This is because it's hard to hold so many records in any one function. I knew that the batch insert utility could insert 1m records in 1000 chunks, so I decided to try chaining ten batch inserts in that same format and success!

- With 10 million records in the database I had to make adjustments to my server code so that the API call would not fetch all 10 million records. To do this I went through Bookshelf's documentation and found that the collection had a couple methods I could utilize. In summary I had to make sure I was performing the query below each time I was making an API request for the data:

```
  SELECT * FROM STANDINGS ORDER BY ID DESC LIMIT 100;
```

- Luckily Bookshelf has the OrderBy and Query methods to help structure the API call on the Standings collections to mimic the query above.

- Work for generating 10 million records is for the most part done. Now I'll be conducting some benchmarking tests on the database before I start with my NoSQL Database.
---

## February 6th, 2019

**1. Initial Benchmarking of SQL Database**

- One of our objectives in Phase 1 is performing some simple benchmarking tests to measure the initial performance of our databases. I accomplished this by measuring 2 aspects of my SQL database:

  - **1. Measure and record the time it takes to insert 10,000,000 records**
    - This was done by creating one date function before the seed function ran, another after it ran, and calculating the difference in time between the two in milliseconds. I've posted the results below.

  ![benchmark records][three]

  - **2. Measure and record time it takes to execute a simple read query**
    - I used one of PostgreSQL's built in functions to analyze the time it took to randomly select one of 10,000,000 records. The first initial results are unfortunate as it took about 19 seconds to perform a simple read query of 1 record. The goal is to achieve a read query speeds of 50ms, so I have some work to do to try to minimize my initial time.

  ![benchmark query][four]

---

## February 11th, 2019

**1. Refactor Database to Cassandra**

- The next step of phase 1 was to implement a NoSQL DBMS for our chosen project. Since MongoDB was the original NoSQL DBMS we chose Cassandra.

- Implmenting Cassandra came with quite a few challenges, the lack of clear documentation on how to implement Cassandra with Express/NodeJS being one of them. One of the first errors I encountered while setting up Cassandra was a thrift socket error:

![cassandra thrift socket error][five]

- When loading Cassandra it will create a thrift socket protocol. Supposedly this is a type of interface that was originally used with Cassandra, but the standard nowadays is CQL or the Cassandra Query Language. Why Cassandra stil lloads this I'm not sure as I already installed CQL with the Python package manager. After some trial and error I was able to resolve this issue by installing an older Java SDK (version 1.8). I had other issues running Cassandra that weren't resolved by simply running ```brew services start cassandra``` after installation. In order to resolve these issues I had to remove lines from the ```cassandra-env.sh``` file after running ```cassandra -f```. Luckily these got resolved quickly.

- The second major issue I had when finishing installation was running CQL for the first time. By default, CQL seems to connect to ```localhost 9042``` but I ran into issues connecting to this port. After searching for solutions on the internet I tried connecting to CQL using port 9160.

![cassandra cql port error][six]

- Since I'm not using the standard port I had to think about how to make my config file connect to the right port too. Luckily with the installation of Cassandra-Driver there was a helpful method to do just that.

- I was able to succesfully insert mock data and render it to the front end. The next step is to insert 10 million records.

**2. Seeding 10,000,000 records and Benchmarking**

- Inserting 10,000,000 records had it's own set of challenges. First was figuring out the right format to write my insertion function so that I wouldn't end up with a heap memory allocation error like I did with my Postgres database. I ended up utilizing the driver's built in batch function to insert multiple records. Unfortunately the batch function could only insert 125 records at a time, so I chained several batch methods together in promises so I could insert 20,000 records at a time. From there I looped the insertion to run 500 times so that I could insert all 10m records. The first warning I encountered with this was a timeout warning:

```
(node:16353) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 1): OperationTimedOutError: The host 127.0.0.1:9160 did not reply before timeout 12000 ms
```

- Luckily this isn't fatal, and I modified my timeout settings in the ```cassandra.yaml``` file to make sure it wasn't. It took some time but I was able to successfully insert 10 million records in about 21 minutes.

- The simplest way to check for 10,000,000 records is to perform a count function in CQL, but unfortunately performing such a function is not possible because of timeout issues:

```
cqlsh:espn> select count(uid) from standings;
ReadTimeout: Error from server: code=1200 [Coordinator node timed out waiting for replica nodes' responses] message="Operation timed out - received only 1 responses." info={'received_responses': 1, 'required_responses': 1, 'consistency': 'ONE'}
```
- In order to confirm whether I had 10 million records or not, I added a logging statement to the end of my batch insertions of 20,000:

```
const startTime = new Date().getTime();
...
.then( ()=> db.batch(generateBatch(), { prepare: true})) // 20,000
.then(() => console.log(`Successfully inserted 20,000 records in ${new Date().getTime() - startTime}ms`));
```

- Since the chain of batch insertions creates 20,000 records, and the loop runs 500 times, I copied each logging statement and pasted them into an Excel sheet and summed up the rows. This confirmed that I had 10,000,000.

![cassandra 10m record confirmation][seven]

- The logging statements also served as my benchmark. At every 20,000 records, a timestamp is calculated to figure out how many milliseconds the insertion of 20,000 took. Looking at the last insertion shows a time of 1274791ms or about 21 minutes.

![cassandra benchmark][eight]

- Although this method is crude, I was able to insert the required amount of data in less than 50 minutes. The next step will be to fine-tune Cassandra to see if I can reduce that time and make my seeding code look cleaner. I'll most likely deploy my PostgerSQL and Cassandra databases to AWS EC2 so that I can perform more tests to see which comes out faster. At this point it's about the same, PostgresSQL is only about 5 minutes faster (but the code is cleaner!).

---

[one]: images/fullstandingscomponentsnapshot.png
[two]: images/standingscomponentsnapshot.png
[three]: images/benchmark1.png
[four]: images/benchmark2.png
[five]: images/cassandraerror1.png
[six]: images/cassandraerror2.png
[seven]: images/cassandrabenchmark2.png
[eight]: images/cassandrabenchmark1.png
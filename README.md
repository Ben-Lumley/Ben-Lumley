# Neo4j Sales Engineering Technical Assignment Part 2

**Set up Neo4j Enterprise (Docker / VM/ Sandbox).**

I chose to use a Neo4j Sandbox
</br> URI: bolt+s://ead7d77d67ffd1f5f1d6e8bb2271dc15.neo4jsandbox.com:7687
</br> USERNAME: neo4j
</br> PASSWORD: parallels-confinements-warranty


**Model any of the below datasets from Tabular to Graph using arrows.app and share the JSON export or URL of the data model.**

I chose the:	Investigation for Public Safety - Social Network Data

</br> The data moedel I developed for this data using arrows.app:


![image](https://github.com/Ben-Lumley/Ben-Lumley/assets/155236280/37a0295f-e89f-47cb-99f4-a6cc04bb4854)




**Develop a graph-enabled solution using one of the following programming languages: JavaScript, Python, Go, .NET, or Java. This application should be capable of ingesting data based on the created data model.**

I used Python to develop the solution. Code is below:

</br> #pip3 install neo4j-driver

</br> from neo4j import GraphDatabase

</br> driver = GraphDatabase.driver("bolt://18.204.217.27:7687", auth=("neo4j", "parallels-confinements-warranty"))

</br> def execute_cypher_query(query_text):
</br>     records, summary, keys = driver.execute_query(
</br>     query_text,
</br>     database_="neo4j",
</br>     )
</br>     print("The query `{query}` returned {records_count} records in {time} ms.".format(
</br>         query=summary.query, records_count=len(records),
 </br>        time=summary.result_available_after
</br>         ))
</br>     print("---------------------------------------------------------------------------------------------")
</br> 
</br> 
</br> #Clean-up Neo4j DB prior to run
</br> execute_cypher_query(
</br>     '''MATCH (n)
</br>     DETACH DELETE (n)'''
</br>     )
</br> 
</br> #Ingest data and create nodes from input files as per data model
</br> 
</br> #Person Node
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS row
</br>     MERGE (:Person{
</br>     passportnumber : row.passportnumber,
</br>     name : row.name})'''
</br>     )
</br> #Institution Node
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_education.csv' AS row
</br>     MERGE (:Institution{
</br>     nameofinstitution: row.nameofinstitution})'''
</br>     )
</br> #Organization Node
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_work.csv' AS row
</br>     MERGE (:Organization{
</br>     nameoforganization : row.nameoforganization})'''
</br>     )
</br> #Merchant Node
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_transaction.csv' AS row
</br>     MERGE (:Merchant{
</br>     merchant_name : row.merchant})'''
</br>     )
</br> 
</br> 
</br> #Create Country nodes from all input files
</br> 
</br> #from trips
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>         'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS row
</br>         MERGE (:Country{
</br>         country : trim(row.citizenship)})'''
</br>     )
</br> #from education
</br> execute_cypher_query(
</br>         '''LOAD CSV WITH HEADERS FROM
</br>         'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_education.csv' AS row
</br>         MERGE (:Country{
</br>         country : trim(row.country)})'''
</br>     )
</br> #from work
</br> execute_cypher_query(
</br>         '''LOAD CSV WITH HEADERS FROM
</br>         'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_work.csv' AS row
</br>         MERGE (:Country{
</br>         country : trim(row.country)})'''
</br>     )
</br> #from transaction
</br> execute_cypher_query(
</br>         '''LOAD CSV WITH HEADERS FROM
</br>         'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_transaction.csv' AS row
</br>         MERGE (:Country{
</br>         country : trim(row.country)})'''
</br>     )
</br> 
</br> 
</br> 
</br> #Create relationships as per data model
</br> 
</br> #(person)-[ATTENDS]->(institution)
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM 
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_education.csv' AS education
</br>     MATCH (person : Person {passportnumber: education.passportnumber})
</br>     MATCH (institution : Institution {nameofinstitution: education.nameofinstitution})
</br>     MERGE (person)-[r:ATTENDS]->(institution)
</br>     SET
</br>     r.course = education.course,
</br>     r.startyear = education.startyear,
</br>     r.endyear = education.endyear'''
</br>     )
</br> #(person)-[WORKS_FOR]->(organization)
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM 
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_work.csv' AS work
</br>     MATCH (person : Person {passportnumber: work.passportnumber})
</br>     MATCH (organization : Organization {nameoforganization: work.nameoforganization})
</br>     MERGE (person)-[r:WORKS_FOR]->(organization)
</br>     SET
</br>     r.designation = work.designation,
</br>     r.startyear = work.startyear,
</br>     r.endyear = work.endyear'''
</br>     )
</br> #(person)-[CITIZEN_OF]->(country)
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS trips
</br>     MATCH (person : Person {passportnumber: trips.passportnumber})
</br>     MATCH (c : Country {country: trim(trips.citizenship)})
</br>     MERGE (person)-[r:CITIZEN_OF]->(c)'''
</br>     )
</br> #(person)-[DEPARTS_FROM]->(country)
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS trips
</br>     MATCH (person : Person {passportnumber: trips.passportnumber})
</br>     MATCH (c : Country {country: trim(trips.departurecountry)})
</br>     MERGE (person)-[r:DEPARTS_FROM]->(c)
</br>     SET
</br>     r.departuredate = trips.departuredate'''
</br>     )
</br> #(person)-[ARRIVES_AT]->(country)
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS trips
</br>     MATCH (person : Person {passportnumber: trips.passportnumber})
</br>     MATCH (c : Country {country: trim(trips.arrivalcountry)})
</br>     MERGE (person)-[r:ARRIVES_AT]->(c)
</br>     SET
</br>     r.arrivaldate = trips.departuredate'''
</br>     )
#(merchant)-[r:BASED_IN]->(country)
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_transaction.csv' AS transaction
</br>     MATCH (merchant : Merchant {merchant_name: transaction.merchant})
</br>     MATCH (c : Country {country: trim(transaction.country)})
</br>     MERGE (merchant)-[r:BASED_IN]->(c)'''
</br>     )
</br> #(person)-[TRANSACTS]->(merchant)
</br> execute_cypher_query(
</br>     '''LOAD CSV WITH HEADERS FROM
</br>     'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_transaction.csv' AS transaction
</br>     MATCH (person : Person {passportnumber: transaction.passportnumber})
</br>     MATCH (merchant : Merchant {merchant_name: transaction.merchant})
</br>     MERGE (person)-[r:TRANSACTS]->(merchant)
</br>     SET
</br>     r.cardnumber = transaction.cardnumber,
</br>     r.transactiondate = transaction.transactiondate,
</br>     r.amount = transaction.amount'''
</br>     )


</br> **The resulting graph:**
</br> 
![image](https://github.com/Ben-Lumley/Ben-Lumley/assets/155236280/ef347b05-22e7-4c64-8770-f3c0253fbf7f)


    


</br> **Generate business insights or patterns that deliver business value by writing Cypher or using Graph Data Science query. You may use Neo4j Browser or write an automated script to execute your queries. You are also encouraged to look into other Neo4j tools available such as Bloom and NeoDash to articulate business value.**
</br> 
</br> I used the Neo4j Browser to visualise the data and identified a potential relationship relevant to the Investigation for Public Safety. I then developed the following Cypher queries to deliver additional business value and explore and validate the potential safety threat:

</br> //Whose connected and how?
</br> 
</br> MATCH (a:Person)--(r)--(b:Person)
</br> RETURN a,b,r
</br> 
![image](https://github.com/Ben-Lumley/Ben-Lumley/assets/155236280/d9493ecd-93fb-4b9f-b234-67136fe94d6a)


</br> 
</br> 
//Did they really study together? YES
</br> 
</br> MATCH (a:Person)-[r1:ATTENDS]->(i:Institution)<-[r2:ATTENDS]-(b:Person)
</br> WHERE i.nameofinstitution = i.nameofinstitution	AND
</br> r1.startyear = r2.startyear AND
</br> r1.endyear = r2.endyear AND
</br> r1.course = r2.course
</br> RETURN DISTINCT a.name,i.nameofinstitution,r1.course,r1.startyear,r1.endyear
</br> 
![image](https://github.com/Ben-Lumley/Ben-Lumley/assets/155236280/a128acf6-901b-462c-a475-89a89b3d6a33)

</br> 
</br> 
</br> //Did they travel to Vietman at the same time? YES
</br> 
</br> MATCH (a:Person)-[r1:ARRIVES_AT]->(c:Country)<-[r2:ARRIVES_AT]-(b:Person)
</br> WHERE c.country = c.country	AND
</br> r1.arrivaldate = r2.arrivaldate
</br> RETURN DISTINCT a.name,c.country,r1.arrivaldate
</br> 
![image](https://github.com/Ben-Lumley/Ben-Lumley/assets/155236280/f9a5dcec-d9d7-4c51-b885-23f873012eaa)

</br> 
</br> 
</br> //Did they interact while in Vietman? VERY LIKELY
</br> 
</br> MATCH (a:Person)-[r1:TRANSACTS]->(m:Merchant)<-[r2:TRANSACTS]-(b:Person)
</br> WHERE m.merchant_name = m.merchant_name	AND
</br> r1.transactiondate = r2.transactiondate
</br> RETURN DISTINCT a.name, m.merchant_name, r1.transactiondate,r1.amount
</br> 
![image](https://github.com/Ben-Lumley/Ben-Lumley/assets/155236280/7b60d7a2-5bf6-49c6-9008-39e108e153eb)
</br> 
</br> 
**Document all of the above in a README.md file with the code base and share it through a public GitHub repository.**
as above

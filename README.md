Neo4j Sales Engineering Technical Assignment Part 2

**Set up Neo4j Enterprise (Docker / VM/ Sandbox).**

I chose to use a Neo4j Sandbox
URI: bolt+s://ead7d77d67ffd1f5f1d6e8bb2271dc15.neo4jsandbox.com:7687
USERNAME: neo4j
PASSWORD: parallels-confinements-warranty


**Model any of the below datasets from Tabular to Graph using arrows.app and share the JSON export or URL of the data model.**

I chose the:	Investigation for Public Safety - Social Network Data

The data moedel I developed for  this data using arrows.app can be found at: https://arrows.app/#/local/id=OALrv9s26BnVLfAvv77z


**Develop a graph-enabled solution using one of the following programming languages: JavaScript, Python, Go, .NET, or Java. This application should be capable of ingesting data based on the created data model.**

I used Python to develop the solution. Code is below:

#pip3 install neo4j-driver

from neo4j import GraphDatabase

driver = GraphDatabase.driver("bolt://18.204.217.27:7687", auth=("neo4j", "parallels-confinements-warranty"))

def execute_cypher_query(query_text):
    records, summary, keys = driver.execute_query(
    query_text,
    database_="neo4j",
    )
    print("The query `{query}` returned {records_count} records in {time} ms.".format(
        query=summary.query, records_count=len(records),
        time=summary.result_available_after
        ))
    print("---------------------------------------------------------------------------------------------")


#Clean-up Neo4j DB prior to run
execute_cypher_query(
    '''MATCH (n)
    DETACH DELETE (n)'''
    )

#Ingest data and create nodes from input files as per data model

#Person Node
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS row
    MERGE (:Person{
    passportnumber : row.passportnumber,
    name : row.name})'''
    )
#Institution Node
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_education.csv' AS row
    MERGE (:Institution{
    nameofinstitution: row.nameofinstitution})'''
    )
#Organization Node
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_work.csv' AS row
    MERGE (:Organization{
    nameoforganization : row.nameoforganization})'''
    )
#Merchant Node
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_transaction.csv' AS row
    MERGE (:Merchant{
    merchant_name : row.merchant})'''
    )


#Create Country nodes from all input files

#from trips
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
        'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS row
        MERGE (:Country{
        country : trim(row.citizenship)})'''
    )
#from education
execute_cypher_query(
        '''LOAD CSV WITH HEADERS FROM
        'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_education.csv' AS row
        MERGE (:Country{
        country : trim(row.country)})'''
    )
#from work
execute_cypher_query(
        '''LOAD CSV WITH HEADERS FROM
        'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_work.csv' AS row
        MERGE (:Country{
        country : trim(row.country)})'''
    )
#from transaction
execute_cypher_query(
        '''LOAD CSV WITH HEADERS FROM
        'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_transaction.csv' AS row
        MERGE (:Country{
        country : trim(row.country)})'''
    )



#Create relationships as per data model

#(person)-[ATTENDS]->(institution)
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM 
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_education.csv' AS education
    MATCH (person : Person {passportnumber: education.passportnumber})
    MATCH (institution : Institution {nameofinstitution: education.nameofinstitution})
    MERGE (person)-[r:ATTENDS]->(institution)
    SET
    r.course = education.course,
    r.startyear = education.startyear,
    r.endyear = education.endyear'''
    )
#(person)-[WORKS_FOR]->(organization)
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM 
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_work.csv' AS work
    MATCH (person : Person {passportnumber: work.passportnumber})
    MATCH (organization : Organization {nameoforganization: work.nameoforganization})
    MERGE (person)-[r:WORKS_FOR]->(organization)
    SET
    r.designation = work.designation,
    r.startyear = work.startyear,
    r.endyear = work.endyear'''
    )
#(person)-[CITIZEN_OF]->(country)
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS trips
    MATCH (person : Person {passportnumber: trips.passportnumber})
    MATCH (c : Country {country: trim(trips.citizenship)})
    MERGE (person)-[r:CITIZEN_OF]->(c)'''
    )
#(person)-[DEPARTS_FROM]->(country)
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS trips
    MATCH (person : Person {passportnumber: trips.passportnumber})
    MATCH (c : Country {country: trim(trips.departurecountry)})
    MERGE (person)-[r:DEPARTS_FROM]->(c)
    SET
    r.departuredate = trips.departuredate'''
    )
#(person)-[ARRIVES_AT]->(country)
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_trips.csv' AS trips
    MATCH (person : Person {passportnumber: trips.passportnumber})
    MATCH (c : Country {country: trim(trips.arrivalcountry)})
    MERGE (person)-[r:ARRIVES_AT]->(c)
    SET
    r.arrivaldate = trips.departuredate'''
    )
#(merchant)-[r:BASED_IN]->(country)
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_transaction.csv' AS transaction
    MATCH (merchant : Merchant {merchant_name: transaction.merchant})
    MATCH (c : Country {country: trim(transaction.country)})
    MERGE (merchant)-[r:BASED_IN]->(c)'''
    )
#(person)-[TRANSACTS]->(merchant)
execute_cypher_query(
    '''LOAD CSV WITH HEADERS FROM
    'https://gist.githubusercontent.com/maruthiprithivi/10b456c74ba99a35a52caaffafb9d3dc/raw/a46af9c6c4bf875ded877140c112e9ff36f8f2e8/sng_transaction.csv' AS transaction
    MATCH (person : Person {passportnumber: transaction.passportnumber})
    MATCH (merchant : Merchant {merchant_name: transaction.merchant})
    MERGE (person)-[r:TRANSACTS]->(merchant)
    SET
    r.cardnumber = transaction.cardnumber,
    r.transactiondate = transaction.transactiondate,
    r.amount = transaction.amount'''
    )
    


**Generate business insights or patterns that deliver business value by writing Cypher or using Graph Data Science query. You may use Neo4j Browser or write an automated script to execute your queries. You are also encouraged to look into other Neo4j tools available such as Bloom and NeoDash to articulate business value.**

I used the Neo4j Browser to visualise the data and identified a potential relationship relevant to the Investigation for Public Safety. I then developed the following Cypher queries to deliver additional business value and explore and validate the potential safety threat:

//Whose connected and how?
MATCH (a:Person)--(r)--(b:Person)
RETURN a,b,r

//Did they really study together? YES
MATCH (a:Person)-[r1:ATTENDS]->(i:Institution)<-[r2:ATTENDS]-(b:Person)
WHERE i.nameofinstitution = i.nameofinstitution	AND
r1.startyear = r2.startyear AND
r1.endyear = r2.endyear AND
r1.course = r2.course
RETURN DISTINCT a.name,i.nameofinstitution,r1.course,r1.startyear,r1.endyear

//Did they travel to Vietman at the same time? YES
MATCH (a:Person)-[r1:ARRIVES_AT]->(c:Country)<-[r2:ARRIVES_AT]-(b:Person)
WHERE c.country = c.country	AND
r1.arrivaldate = r2.arrivaldate
RETURN DISTINCT a.name,c.country,r1.arrivaldate

//Did they interact while in Vietman? VERY LIKELY
MATCH (a:Person)-[r1:TRANSACTS]->(m:Merchant)<-[r2:TRANSACTS]-(b:Person)
WHERE m.merchant_name = m.merchant_name	AND
r1.transactiondate = r2.transactiondate
RETURN DISTINCT a.name, m.merchant_name, r1.transactiondate,r1.amount


**Document all of the above in a README.md file with the code base and share it through a public GitHub repository.**
as above

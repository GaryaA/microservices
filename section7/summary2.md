# Section 7: Using MySQL DBs inside microservices

This section demonstrates the integration of MySQL as a persistent database for each microservice, following the "database-per-service" pattern. The key changes are:

1.  **Database-per-Service**: Each microservice (`accounts`, `loans`, `cards`) is configured to use its own dedicated MySQL database. The `docker-compose.yml` file defines separate database services (`accountsdb`, `loansdb`, `cardsdb`) for each microservice, ensuring data isolation.

2.  **Dependency Update**: The `pom.xml` for each microservice is updated to include the `mysql-connector-j` dependency, replacing the previous H2 database driver.

3.  **Configuration Update**: The microservices are configured with the appropriate JDBC connection strings to connect to their respective MySQL databases. This configuration is managed through environment variables in the `docker-compose.yml` file.

This setup illustrates a common and recommended practice in microservices architecture, where each service owns and manages its own data, preventing tight coupling between services at the database level.

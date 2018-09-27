# Azure-Databricks-HDInsight-Hive-Metastore
How to share an HDInsight Hive Metastore with Azure Databricks

## Steps
1. Create a SQL Database (PaaS) server in Azure
2. Create an empty database in under the server
3. Create an HDInsight cluster in Azure and point the external Hive metastore to your database
4. Delete the HDInsight cluster after it has been created
5. Create a Databricks workspace in Azure
6. Launch the workspace and open a notebook
7. Run Step 1 in a cell
8. Run Step 2 in a cell (replace your server name, database name, username and password)
9. Terminate your Databricks cluster
10. Restart your cluster
11. In a SQL notebook you can run SHOW TABLES (you should see the sample table from HDInsight)
    * NOTE: You will not be able to select the data from this table.  The table is probably pointing to blob or ADLS storage.  You should create all your tables in HDI/Databricks as EXTERNAL and makes sure you LOCATION property is set to blob or ADLS.  You Databricks cluster will also need to be authorized to access ADLS or blob.

### Step 1
```
dbutils.fs.mkdirs("dbfs:/databricks/init/")
```

### Step 2
```
dbutils.fs.put(
    "/databricks/init/external-metastore.sh",
    """#!/bin/sh
      |# Loads environment variables to determine the correct JDBC driver to use.
      |source /etc/environment
      |# Quoting the label (i.e. EOF) with single quotes to disable variable interpolation.
      |cat << 'EOF' > /databricks/driver/conf/00-custom-spark.conf
      |[driver] {
      |    # Hive specific configuration options for metastores in the local mode.
      |    "spark.hadoop.javax.jdo.option.ConnectionURL" = "jdbc:sqlserver://<<FILLL-ME-IN-SERVERNAME>>.database.windows.net:1433;database=<<FILLL-ME-IN-DATABASE-NAME>>;encrypt=true;trustServerCertificate=true;create=false;loginTimeout=300"
      |    "spark.hadoop.javax.jdo.option.ConnectionUserName" = "<<FILLL-ME-IN-USER>>"
      |    "spark.hadoop.javax.jdo.option.ConnectionPassword" = "<<FILLL-ME-IN-PASSWORD>>"
      |    "hive.metastore.schema.verification.record.version" = "true"
      |    "spark.sql.hive.metastore.jars" = "maven"
      |    "hive.metastore.schema.verification" = "true"
      |    "spark.sql.hive.metastore.version" = "2.1.1"
      |EOF
      |# Add the JDBC driver separately since must use variable expansion to choose the correct
      |# driver version.
      |cat << EOF >> /databricks/driver/conf/00-custom-spark.conf
      |    "spark.hadoop.javax.jdo.option.ConnectionDriverName" = "com.microsoft.sqlserver.jdbc.SQLServerDriver"
      |}
      |EOF
      |""".stripMargin,
    overwrite = true
)
```

# sqlserver-aws-datalake

## Objective
- to achieve cost performant ODS layer on AWS
- data source is sqlserver, hundreds of tables, each table with 10s level columns
- should consider change data capture 

## Solution Directions

<img width="931" alt="image" src="https://github.com/user-attachments/assets/398bbb34-d3f4-46e4-93bd-a7da848c0a3e">


## Implementation Guidance

### Direction1: DMS (Full Load + CDC) + Redshift Serverless + QuickSight

#### 1.1 sqlserver preparation
- database, table DDL

```sql
create database testdb;
use testdb;

create table db1 (
	persionid int,
	personname varchar(225)
	);
```
- table DML

```sql
insert into db1 values (1, 'John');
insert into db1 values (2, 'Mary');
insert into db1 values (3, 'Dick');
```

- enable sqlserver CDC

```sh
EXEC msdb.dbo.rds_cdc_enable_db 'testdb';
GO

EXECUTE sys.sp_cdc_enable_table @source_schema = N'dbo', @source_name =N'db1', @role_name = NULL;
GO

EXEC sys.sp_cdc_change_job @job_type = 'capture' ,@pollinginterval = 3599;
GO
```



- [1.Export MS SQL Server database to Amazon S3 via AWS DMS](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/export-a-microsoft-sql-server-database-to-amazon-s3-by-using-aws-dms.html)
- [2.Migrating Data to Delta Lake via AWS DMS](https://www.databricks.com/blog/2019/07/15/migrating-transactional-data-to-a-delta-lake-using-aws-dms.html)
- [3.AWS DMS to migrate SQL Server to Amazon S3 bucket - Full Load + CDC](https://www.youtube.com/watch?v=Uk8uFUrDUp8)
- [4.Analyzing Data in Amazon S3 using Amazon Athena](https://aws.amazon.com/blogs/big-data/analyzing-data-in-s3-using-amazon-athena/)
- [5.Amazon QuickSight Connect to Data Sources](https://docs.aws.amazon.com/quicksight/latest/user/supported-data-sources.html)
- [6.Amazon EMR Serveless Query S3 Data](https://github.com/aws-samples/emr-serverless-samples/tree/main/examples)
- [7.Databricks @ AWS MarketPlace](https://aws.amazon.com/marketplace/pp/prodview-wtyi5lgtce6n6?applicationId=AWS-Marketplace-Console&ref_=beagle&sr=0-1)

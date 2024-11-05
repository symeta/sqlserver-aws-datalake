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

#### 1.2 redshift serverless preparation
- [create workgroup & namespace](https://docs.aws.amazon.com/redshift/latest/mgmt/serverless-console-workgroups-create-workgroup-wizard.html)

- tips1: enable "manually enter the admin password" when creating namespace
<img width="746" alt="Screenshot 2024-11-05 at 12 03 12" src="https://github.com/user-attachments/assets/d3a24b19-29ec-4350-bad7-3179e9cd0dee">

- tips2: make sure role "dms-access-for-endpoint" has been attached to namespace. [dms-access-for-endpoint role creation guidance](https://docs.aws.amazon.com/dms/latest/userguide/security-iam.html)

<img width="1099" alt="Screenshot 2024-11-05 at 12 09 34" src="https://github.com/user-attachments/assets/7a95a84b-766e-40d1-9bbb-f6d47e63075a">

- trust policy configuration of role dms-access-for-endpoint
```json

 {
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "1",
      "Effect": "Allow",
      "Principal": {
        "Service": "dms.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    },
    {
      "Sid": "2",
      "Effect": "Allow",
      "Principal": {
        "Service": "redshift.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}           
```
- redshift serverless connection info

<img width="1084" alt="Screenshot 2024-11-05 at 12 22 03" src="https://github.com/user-attachments/assets/6221bc65-9fdf-48a1-8980-7f6b0f225640">


#### 1.3 DMS preparation
- [Replication Instance Creation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_ReplicationInstance.Creating.html)

<img width="1190" alt="Screenshot 2024-11-05 at 11 48 23" src="https://github.com/user-attachments/assets/a25a960d-b7ec-409b-88ef-83b0f556ac17">

- [Source Endpoint Creation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Source.SQLServer.html)

<img width="1195" alt="Screenshot 2024-11-05 at 11 55 27" src="https://github.com/user-attachments/assets/3a3a4206-4cd4-4926-a403-5d524caff34a">

- [Target Endpoint Creation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Target.Redshift.html)

<img width="1198" alt="Screenshot 2024-11-05 at 12 26 02" src="https://github.com/user-attachments/assets/78a2cd07-3882-4a88-a628-f0f5b4834438">


#### 1.4 DMS task creation

- [DMS migration task creation](https://docs.aws.amazon.com/dms/latest/userguide/CHAP_Tasks.Creating.html)

<img width="1197" alt="Screenshot 2024-11-05 at 12 30 05" src="https://github.com/user-attachments/assets/17497fed-b87b-4667-9de9-4d014cbbb0e9">

#### 1.5 CDC DML sql run on sqlserver
```sql
insert into db1 values (4, 'Adam');
update db1 set personname = 'Josh' where persionid = 1;
delete from db1 where persionid = 2

insert into db1 values (5, 'Lily');
update db1 set personname = 'name changed' where persionid = 1;
delete from db1 where persionid = 4
```

#### 1.6 redshift serverless query table show case

<img width="672" alt="Screenshot 2024-11-05 at 12 36 12" src="https://github.com/user-attachments/assets/91a626bb-9391-48a8-a552-f1b41d333bd1">

#### 1.7 quicksight activation & access to redshift serverless & amazon q in quicksight

- [quicksight activation guidance](https://catalog.us-east-1.prod.workshops.aws/workshops/cd8ebba2-2ef8-431a-8f72-ca7f6761713d/en-US/q-workshop/0-prerequisites)

- [quicksight access redshift serverless guidance](https://docs.aws.amazon.com/quicksight/latest/user/redshift-vpc-access.html)

- [amazon q in quicksight](https://catalog.us-east-1.prod.workshops.aws/workshops/cd8ebba2-2ef8-431a-8f72-ca7f6761713d/en-US/q-workshop/6-generative-bi)





## references
- [1.Export MS SQL Server database to Amazon S3 via AWS DMS](https://docs.aws.amazon.com/prescriptive-guidance/latest/patterns/export-a-microsoft-sql-server-database-to-amazon-s3-by-using-aws-dms.html)
- [2.Migrating Data to Delta Lake via AWS DMS](https://www.databricks.com/blog/2019/07/15/migrating-transactional-data-to-a-delta-lake-using-aws-dms.html)
- [3.AWS DMS to migrate SQL Server to Amazon S3 bucket - Full Load + CDC](https://www.youtube.com/watch?v=Uk8uFUrDUp8)
- [4.Analyzing Data in Amazon S3 using Amazon Athena](https://aws.amazon.com/blogs/big-data/analyzing-data-in-s3-using-amazon-athena/)
- [5.Amazon QuickSight Connect to Data Sources](https://docs.aws.amazon.com/quicksight/latest/user/supported-data-sources.html)
- [6.Amazon EMR Serveless Query S3 Data](https://github.com/aws-samples/emr-serverless-samples/tree/main/examples)
- [7.Databricks @ AWS MarketPlace](https://aws.amazon.com/marketplace/pp/prodview-wtyi5lgtce6n6?applicationId=AWS-Marketplace-Console&ref_=beagle&sr=0-1)

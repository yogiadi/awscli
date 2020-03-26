cat myredshiftrole_cli.json
```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "redshift.amazonaws.com"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}
```


aws iam create-role --role-name myredshiftrole_cli --assume-role-policy-document file://myredshiftrole_cli.json

{
    "Role": {
        "Path": "/",
        "RoleName": "myredshiftrole_cli",
        "RoleId": "AROAVWCIBUN3A5OJHKDZI",
        "Arn": "arn:aws:iam::390993126262:role/myredshiftrole_cli",
        "CreateDate": "2020-03-26T12:05:00+00:00",
        "AssumeRolePolicyDocument": {
            "Version": "2012-10-17",
            "Statement": [
                {
                    "Effect": "Allow",
                    "Principal": {
                        "Service": "redshift.amazonaws.com"
                    },
                    "Action": "sts:AssumeRole"
                }
            ]
        }
    }
}

aws iam attach-role-policy --role-name myredshiftrole_cli --policy-arn "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"

aws iam list-attached-role-policies --role-name myredshiftrole_cli
{
    "AttachedPolicies": [
        {
            "PolicyName": "AmazonS3ReadOnlyAccess",
            "PolicyArn": "arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess"
        }
    ]
}

aws redshift create-cluster --cluster-identifier redshift-cluster-1 --node-type dc2.large --master-username awsuser --master-user-password Awspassword_2020 --number-of-nodes 2 --iam-roles arn:aws:iam::390993126262:role/myredshiftrole_cli

aws s3api create-bucket --bucket yogiadibucket --region ap-south-1 --create-bucket-configuration LocationConstraint=ap-south-1

aws s3 ls s3://yogiadibucket

aws s3 sync . s3://yogiadibucket/

create table users(
	userid integer not null distkey sortkey,
	username char(8),
	firstname varchar(30),
	lastname varchar(30),
	city varchar(30),
	state char(2),
	email varchar(100),
	phone char(14),
	likesports boolean,
	liketheatre boolean,
	likeconcerts boolean,
	likejazz boolean,
	likeclassical boolean,
	likeopera boolean,
	likerock boolean,
	likevegas boolean,
	likebroadway boolean,
	likemusicals boolean);

create table venue(
	venueid smallint not null distkey sortkey,
	venuename varchar(100),
	venuecity varchar(30),
	venuestate char(2),
	venueseats integer);

create table category(
	catid smallint not null distkey sortkey,
	catgroup varchar(10),
	catname varchar(10),
	catdesc varchar(50));

create table date(
	dateid smallint not null distkey sortkey,
	caldate date not null,
	day character(3) not null,
	week smallint not null,
	month character(5) not null,
	qtr character(5) not null,
	year smallint not null,
	holiday boolean default('N'));

create table event(
	eventid integer not null distkey,
	venueid smallint not null,
	catid smallint not null,
	dateid smallint not null sortkey,
	eventname varchar(200),
	starttime timestamp);

create table listing(
	listid integer not null distkey,
	sellerid integer not null,
	eventid integer not null,
	dateid smallint not null  sortkey,
	numtickets smallint not null,
	priceperticket decimal(8,2),
	totalprice decimal(8,2),
	listtime timestamp);

create table sales(
	salesid integer not null,
	listid integer not null distkey,
	sellerid integer not null,
	buyerid integer not null,
	eventid integer not null,
	dateid smallint not null sortkey,
	qtysold smallint not null,
	pricepaid decimal(8,2),
    commission decimal(8,2),
    saletime timestamp);

copy event from 's3://yogiadibucket/allevents_pipe.txt'
credentials 'aws_iam_role=arn:aws:iam::390993126262:role/myredshiftrole_cli'
delimiter '|' region 'ap-south-1';


copy users from 's3://yogiadibucket/allusers_pipe.txt'
credentials 'aws_iam_role=arn:aws:iam::390993126262:role/myredshiftrole_cli'
delimiter '|' region 'ap-south-1';


copy venue from 's3://yogiadibucket/venue_pipe.txt'
credentials 'aws_iam_role=arn:aws:iam::390993126262:role/myredshiftrole_cli'
delimiter '|' region 'ap-south-1';


copy category from 's3://yogiadibucket/category_pipe.txt'
credentials 'aws_iam_role=arn:aws:iam::390993126262:role/myredshiftrole_cli'
delimiter '|' region 'ap-south-1';


copy date from 's3://yogiadibucket/date2008_pipe.txt'
credentials 'aws_iam_role=arn:aws:iam::390993126262:role/myredshiftrole_cli'
delimiter '|' region 'ap-south-1';


copy listing from 's3://yogiadibucket/listings_pipe.txt'
credentials 'aws_iam_role=arn:aws:iam::390993126262:role/myredshiftrole_cli'
delimiter '|' region 'ap-south-1';


copy sales from 's3://yogiadibucket/sales_tab.txt'
credentials 'aws_iam_role=arn:aws:iam::390993126262:role/myredshiftrole_cli'
delimiter '\t' timeformat 'MM/DD/YYYY HH:MI:SS' region 'ap-south-1';

SELECT eventname, total_price 
FROM  (SELECT eventid, total_price, ntile(1000) over(order by total_price desc) as percentile 
       FROM (SELECT eventid, sum(pricepaid) total_price
             FROM   sales
             GROUP BY eventid)) Q, event E
       WHERE Q.eventid = E.eventid
       AND percentile = 1
ORDER BY total_price desc;

aws redshift delete-cluster --cluster redshift-cluster-1 --skip-final-cluster-snapshot

aws s3 rm s3://yogiadibucket --recursive

aws s3api delete-bucket --bucket yogiadibucket --region ap-south-1

aws iam detach-role-policy --role-name myredshiftrole_cli --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess

aws iam delete-role --role-name myredshiftrole_cli

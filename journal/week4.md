# Week 4 — Postgres and RDS

# 1 Provision RDS Instance

Creade a PostgreSQL database with the following command:

```shell
aws rds create-db-instance \
  --db-instance-identifier  \
  --db-instance-class db.t3.micro \
  --engine postgres \
  --engine-version  14.6 \
  --master-username  \
  --master-user-password  \
  --allocated-storage 20 \
  --availability-zone  \
  --backup-retention-period 0 \
  --port 5432 \
  --no-multi-az \
  --db-name cruddur \
  --storage-type gp2 \
  --publicly-accessible \
  --storage-encrypted \
  --enable-performance-insights \
  --performance-insights-retention-period 7 \
  --no-deletion-protection
```

# 2 Temporarily stop the RDS instance via ClickOps in AWS Console

# 3 Make sure You can establish connection with the PostgreSQL on localhost

1. 
```shell
docker compose up
```

2. Attach a shell to Posgres Container

3. 
```shell
psql -Upostgres --host localhost
```

4. Common PSQL commands:

```sql
\x on -- expanded display when looking at data
\q -- Quit PSQL
\l -- List all databases
\c database_name -- Connect to a specific database
\dt -- List all tables in the current database
\d table_name -- Describe a specific table
\du -- List all users and their roles
\dn -- List all schemas in the current database
CREATE DATABASE database_name; -- Create a new database
DROP DATABASE database_name; -- Delete a database
CREATE TABLE table_name (column1 datatype1, column2 datatype2, ...); -- Create a new table
DROP TABLE table_name; -- Delete a table
SELECT column1, column2, ... FROM table_name WHERE condition; -- Select data from a table
INSERT INTO table_name (column1, column2, ...) VALUES (value1, value2, ...); -- Insert data into a table
UPDATE table_name SET column1 = value1, column2 = value2, ... WHERE condition; -- Update data in a table
DELETE FROM table_name WHERE condition; -- Delete data from a table
```

# 4 Create a database in PostgreSQL on localhost

1. Via the posgres shell (PSQL client):

```sql
CREATE DATABASE cruddur;
```

The alias is following:

```shell
createdb cruddur -h localhost -U postgres
```

2. Option for droppping the database:

```sql
\l
```

```sql
DROP DATABASE cruddur;
```

* Current PosgreSQL documantation at the following link:

```link
https://www.postgresql.org/docs/current/app-createdb.html
```

# 5 Create a schema file in backend

1. Create a file ```db/schema.sql``` in backend.

2. Let PostgreSQL create and add UUID (Universal Unified IDentifiers).
Use an extension called:

```sql
CREATE EXTENSION "uuid-ossp";
```

Add this second line to ```schema.sql``` in case You run the script and schema multiple times:

```sql
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```

# 6 Import script for schema file

From inside ```backend-flask``` directory execute following code:

```shell
psql cruddur < db/schema.sql -h localhost -U postgres
```

Insert password and ```CREATE EXTENSION``` will be logged on shell.

# 7 Use a Connection URL String for password of localhost PosgreSQL

The format for the PostgreSQL connection string / URL is following:

```shell
postgresql://[user[:password]@][netloc][:port][/dbname][?param1=value1&...]
```

Export the created Environment Variable like following:

```shell
export CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"
```

Set Environemnt Variable in GitPod Environment:

```shell
gp env CONNECTION_URL="postgresql://postgres:password@localhost:5432/cruddur"
```

To connect to PostgreSQL use the following command:

```shell
psql $CONNECTION_URL
```

# 8 Set up Production Connection URL String for password of RDS Instance

Export the created Environment Variable for remote connection like following:

```shell
export PROD_CONNECTION_URL=""
```

Set the same Environemnt Variable in GitPod Environment:

```shell
gp env PROD_CONNECTION_URL=""
```

Where netloc (network location) is the Endpoint provided by AWS.

# 9 Create scripts for creating and dropping database as well as loading schema

1. In new folder ```backend-flask/bin``` create the following five files:

```db-create.sh```
```db-drop.sh```
```db-schema-load```
```db-connect.sh```
```db-seed```

2. Change permissions of the files to 744 with following commands:

```shell
chmod 744 db-create.sh
```

```shell
chmod 744 db-drop.sh
```

```shell
chmod 744 db-schema-load.sh
```

```shell
chmod 744 db-connect.sh
```

```shell
chmod 744 db-seed.sh
```

3. Populate the ```db-drop.sh``` script with the following code:

```bash
#! /usr/bin/bash

#echo "== db-drop"
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-drop"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "DROP DATABSE IF EXISTS cruddur;"
```

4. Populate the ```db-create.sh``` script with the following code:

```bash
#! /usr/bin/bash

#echo "== db-create"
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-create"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

NO_DB_CONNECTION_URL=$(sed 's/\/cruddur//g' <<<"$CONNECTION_URL")
psql $NO_DB_CONNECTION_URL -c "CREATE DATABASE cruddur;"
```

5. Populate the ```db-schema-load.sh``` script with the following code:

```bash
#! /usr/bin/bash

#echo "== db-schema-load"
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-schema-load"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

schema_path="$(realpath .)/db/schema.sql"

echo $schema_path

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL cruddur < $schema_path
```

6. Populate the ```db-connect.sh``` script with the following code:

```bash
#! /usr/bin/bash

#echo "== db-connect"
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-connect"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

psql $URL
```

7. Populate the ```db-seed.sh``` script with the following code:

```bash
#! /usr/bin/bash

#echo "== db-seed"
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-seed"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

seed_path="$(realpath .)/db/seed.sql"

echo $seed_path

psql $CONNECTION_URL cruddur < $seed_path
```

# 10 Populate the schema for the database

1. Make a check if tables exist and delete them if they do in ```schema.sql```:

```sql
DROP TABLE IF EXISTS public.users;
DROP TABLE IF EXISTS public.activities;
```

2. Create some tables in the database via ```schema.sql``` (add two following code blocks):

```sql
CREATE TABLE public.users (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  display_name text  NOT NULL,
  handle text NOT NULL,
  email text NOT NULL,
  cognito_user_id text NOT NULL,
  created_at TIMESTAMP default current_timestamp NOT NULL
);
```

```sql
CREATE TABLE public.activities (
  uuid UUID DEFAULT uuid_generate_v4() PRIMARY KEY,
  user_uuid UUID NOT NULL,
  message text NOT NULL,
  replies_count integer DEFAULT 0,
  reposts_count integer DEFAULT 0,
  likes_count integer DEFAULT 0,
  reply_to_activity_uuid integer,
  expires_at TIMESTAMP,
  created_at TIMESTAMP default current_timestamp NOT NULL
);
```

# 11 Create a seed file in backend

1. Create a file ```db/seed.sql``` in backend.

2. Populate the file with the following code:

```sql
INSERT INTO public.users (display_name, handle, cognito_user_id)
VALUES
  ('Andrew Brown', 'andrewbrown' ,'MOCK'),
  ('Andrew Bayko', 'bayko' ,'MOCK');

INSERT INTO public.activities (user_uuid, message, expires_at)
VALUES
  (
    (SELECT uuid FROM public.users WHERE users.handle = 'andrewbrown' LIMIT 1),
    'This was imported as seed data!',
    current_timestamp + interval '10 day'
  )
```

# 12 See what connections are being using

1. In the folder ```backend-flask/bin``` create the following file:

```db-sessions.sh```

2. Change permissions of the files to 744 with following commands:

```shell
chmod 744 db-sessions.sh
```

3. Populate the ```db-sessions.sh``` script with the following code:

```bash
#! /usr/bin/bash

#echo "== db-sessions"
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-sessions"
printf "${CYAN}== ${LABEL}${NO_COLOR}\n"

if [ "$1" = "prod" ]; then
  echo "Running in production mode"
  URL=$PROD_CONNECTION_URL
else
  URL=$CONNECTION_URL
fi

NO_DB_URL=$(sed 's/\/cruddur//g' <<<"$URL")
psql $NO_DB_URL -c "select pid as process_id, \
       usename as user,  \
       datname as db, \
       client_addr, \
       application_name as app,\
       state \
from pg_stat_activity;"
```

# 13 Easily setup (reset) everything for our database

1. In the folder ```backend-flask/bin``` create the following file:

```db-setup.sh```

2. Change permissions of the files to 744 with following commands:

```shell
chmod 744 db-setup.sh
```

3. Populate the ```db-setup.sh``` script with the following code:

```bash
#! /usr/bin/bash
-e # stop if it fails at any point

#echo "==== db-setup"
CYAN='\033[1;36m'
NO_COLOR='\033[0m'
LABEL="db-setup"
printf "${CYAN}==== ${LABEL}${NO_COLOR}\n"

bin_path="$(realpath .)/bin"

source "$bin_path/db-drop.sh"
source "$bin_path/db-create.sh"
source "$bin_path/db-schema-load.sh"
source "$bin_path/db-seed.sh"
```

# 14 Install Postgres Client

1. Set the Environemnt Variable for backend-flask application in ```docker-compose.yml```:

```yml
  backend-flask:
    environment:
      CONNECTION_URL: "${CONNECTION_URL}"
```

2. Add the following software to ```requirments.txt```:

```txt
psycopg[binary]
psycopg[pool]
```

3. Run the install of software:

```shell
pip install -r requirements.txt
```

# 15 DB Object and Connection Pool

1. Create a file ```lib/db.py``` in backend.

2. Populate the newly created file with the following code:

```python
from psycopg_pool import ConnectionPool
import os
import re
import sys
from flask import current_app as app

class Db:
  def __init__(self):
    self.init_pool()

  def template(self, *args):
    pathing = list((app.root_path, 'db', 'sql') + args)
    pathing[-1] = pathing[-1] + ".sql"
    print("pathing")
    print(pathing)
    template_path = os.path.join(*pathing)
    no_color = '\033[0m'
    greem = '\033[92m' 
    print("\n")
    print(f"{green} Load SQL template [{template_path}]==========={no_color}")
    print("===========")
    with open(template_path, 'r') as f:
      template_content = f.read()
    return template_content

  def init_pool(self):
    connection_url = os.getenv("CONNECTION_URL")
    self.pool = ConnectionPool(connection_url)

  def print_sql(self, title, sql):
    no_color = '\033[0m'
    cyan = '\033[96m'
    print(f"{cyan} SQL statement [{title}]==========={no_color}")
    print(sql)
    print("===========")

  def print_params(self, params):
    no_color = '\033[0m'
    blue = '\033[94m'
    print(f"{blue} SQL params []==========={no_color}")
    for key, value in params.items():
      print(key, ":", value)

  # when we want to commit data such as an INSERT 
  # Be Sure to check for RETURNING in ALL UPPERCASEs
  def query_commit(self, sql, params={}):
    self.print_sql('commit with returning', sql)
    pattern = r"\bRETURNING\b"
    is_returning_id = re.search(pattern, sql)
    try:
      with self.pool.connection() as conn:
        cur = conn.cursor()
        cur.execute(sql, params)
        if is_returning_id:
          returning_id = cur.fetchone()[0]
        conn.commit()
        if is_returning_id:
          return returning_id
    except Exception as err:
      self.print_sql_err(err)

  def query_array_json(self, sql, params={}):
    self.print_sql('array', sql)
    wrapped_sql = self.query_wrap_array(sql)
    with self.pool.connection() as conn:
      with conn.cursor() as cur:
        cur.execute(wrapped_sql, params)
        json = cur.fetchone()
        return json[0]

  def query_object_json(self, sql, params={}):
    self.print_sql('json', sql)
    self.print_params(params)

    wrapped_sql = self.query_wrap_object(sql)
    with self.pool.connection() as conn:
      with conn.cursor() as cur:
        cur.execute(wrapped_sql, params)
        json = cur.fetchone()
        if json == None:
          "{}"
        else: 
          return json[0]

  def query_wrap_object(self, template):
    sql = f"""
      (SELECT COALESCE(row_to_json(object_row),'{{}}'::json) FROM (
        {template}
      ) object_row);
    """
    return sql
  def print_sql_err(self, err):
    err_type, err_obj, traceback = sys.exc_info()
    line_num = traceback.tb_lineno
    print ("\npsycopg ERROR:", err, "on line number:", line_num)
    print ("psycopg traceback:", traceback, "-- type:", err_type)
    print ("pgerror:", err.pgerror)
    print ("pgcode:", err.pgcode, "\n")

  def query_wrap_array(self, template):
    sql = f"""
      (SELECT COALESCE(array_to_json(array_agg(row_to_json(array_row))),'[]'::json) FROM (
        {template}
      ) array_row);
    """
    return sql

db = Db()
```

3. In our ```home_activities.py``` in backend services replace the whole structure with following:

```python
from datetime import datetime, timedelta, timezonerun
from lib.db import db

class HomeActivities:
  def run(cognito_user_id = None):
  sql = db.template('activities','home')
  results = db.query_array_json(sql)
  return results
```

# 16 Connect to RDS via Gitpod

1. Create a script ```bin/rds-update-sg-rule.sh``` in backend for auto update of access to Security Group.

```bash
#! /usr/bin/bash

aws ec2 modify-security-group-rules \
    --group-id $DB_SG_ID \
    --security-group-rules "SecurityGroupRuleId=$DB_SG_RULE_ID,SecurityGroupRule={Description=GITPOD,IpProtocol=tcp,FromPort=5432,ToPort=5432,CidrIpv4=$GITPOD_IP/32}"
```

2. Change permissions of the script

```shell
chmod 744 rds-update-sg-rule.sh
```

3. Need to provide Gitpod IP and whitelist for inbound traffic on port 5432 in Security Group for RDS.

```shell
GITPOD_IP=$(curl ifconfig.me)
```

4. Update Gitpod IP on new Environment Variable in ```.gitpod.yml```

```yml
  command: |
    export GITPOD_IP=$(curl ifconfig.me)
    source "$THEIA_WORKSPACE_ROOT/backend-flask/rds-update-sg-rule.sh"
```

5. Create an inbound rule for Postgres (5432) and provide the GITPOD ID.

6. Get the security group rule id so that can be easily modified it in the future from the terminal in Gitpod.

```shell
export DB_SG_ID="sg-"
gp env DB_SG_ID="sg-"
export DB_SG_RULE_ID="sgr-"
gp env DB_SG_RULE_ID="sgr-"
```

7. Wheneverthere is a need to update Security Groups do this for access:

```shell
aws ec2 modify-security-group-rules \
    --group-id $DB_SG_ID \
    --security-group-rules "SecurityGroupRuleId=$DB_SG_RULE_ID,SecurityGroupRule={Description=GITPOD,IpProtocol=tcp,FromPort=5432,ToPort=5432,CidrIpv4=$GITPOD_IP/32}"
```

or execute the script ```bin/rds-update-sg-rule.sh``` located in backend.

# 17 Setup Cognito post confirmation lambda

1. Sign into AWS Console and go to Lambda Functions and Create New Function with name: ```cruddur-post-confirmation```.
Select also RDS current VPC, as well as Subnet and Security Groups.

2. Select Runtime: Python 3.8.

3. Populate the Lambda Code source with the following code:

```python
import json
import psycopg2
import os

def lambda_handler(event, context):
    user = event['request']['userAttributes']
    user_display_name = user['name'] 
    user_email = user['email']
    user_handle = user['preferred_username'] 
    user_cognito_id = user['sub']
    try:
        sql = f"""
            INSERT INTO users 
            (
                display_name,
                email,
                handle, 
                cognito_user_id
            ) 
            VALUES
            (
                %s,
                %s,
                %s,
                %s
            )
        """
        conn = psycopg2.connect(os.getenv('CONNECTION_URL'))
        cur = conn.cursor()
        params = [
            user_display_name, 
            user_email, 
            user_handle,
            user_cognito_id
        ]
        cur.execute(sql, *params)
        conn.commit() 

    except (Exception, psycopg2.DatabaseError) as error:
        print(error)
        
    finally:
        if conn is not None:
            cur.close()
            conn.close()
            print('Database connection closed.')

    return event
```

4. Set the Environment Variable of the following:

```shell
CONNECTION_URL=""
```

5. Create your own development layer by downloading the psycopg2-binary source files from:

```link
https://pypi.org/project/psycopg2-binary/#files
```

```link
https://files.pythonhosted.org/packages/36/af/a9f06e2469e943364b2383b45b3209b40350c105281948df62153394b4a9/psycopg2_binary-2.9.5-cp311-cp311-manylinux_2_17_x86_64.manylinux2014_x86_64.whl
```

Or add the provided layers for the precise Region.
* The uploaded file should be extracted in folder and wtih a ```.zip``` extension afterwards.

6. Add Trigger in Cognito via ClockOps.

* After creating user and confirming the code send to e-mail Lambda shoud put the user in RDS
List new users by connecting to database with appropriate Query.

# 18 Add code to backend

1. Import into ```create_activity.py``` in backend the following from ```lib/db.py```

```python
from lib.db import db
```

2. Add following code and refractor code after the else statement in ```if model['errors']:``` conditional

```python
      expires_at = (now + ttl_offset)
      uuid = CreateActivity.create_activity(user_handle, message, expires_at)
      
      object_json = CreateActivity.query_object_activity(uuid)
      model['data'] = object_json
    return model

  def create_activity(handle, message, expires_at):
    sql = db.template('activities', 'create')
    uuid = db.query_commit(sql,
      {
        'handle': handle,
        'message': message,
        'expires_at': expires_at    
      }
    )
    return uuid
  
  def query_object_activity(uuid):
    sql = db.template('activities', 'object')
    return db.query_object_json(sql, {
      'uuid': uuid,
    })
```

# 19 Create several SQL code files into dedicated folder

1. Create new folder ```db/sql/activities```.

2. Create ```create.sql``` into the folder and populate with following SQL code:

```sql
INSERT INTO public.activities
(
    user_uuid,
    message,
    expires_at
)
VALUES (
    (SELECT uuid 
    FROM public.users 
    WHERE users.handle = %(handle)s 
    LIMIT 1
    ),
    %(message)s,
    %(expires_at)s
) RETURNING uuid;
```

3. Create ```home.sql``` into the folder and populate with following SQL code:

```sql
SELECT
    activities.uuid,
    users.display_name,
    users.handle,
    activities.message,
    activities.replies_count,
    activities.reposts_count,
    activities.likes_count,
    activities.reply_to_activity_uuid,
    activities.expires_at,
    activities.created_at
FROM public.activities
LEFT JOIN public.users ON users.uuid = activities.user_uuid
ORDER BY activities.created_at DESC
```

4. Create ```object.sql``` into the folder and populate with following SQL code:

```sql
SELECT 
    activities.uuid,
    users.display_name,
    users.handle,
    activities.message,
    activities.created_at,
    activities.expires_at
FROM public.
INNER JOIN public.users ON users.uuid = activities.user_uuid
WHERE 
    activities.uuid = %s(uuid)
```

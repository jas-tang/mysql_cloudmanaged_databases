# mysql_cloudmanaged_databases
This is a repository for Assignment 4a in HHA504

# mySQL Setup on Azure and GCP
## GCP

I set up a server on Google Cloud Platform that would work with mySQL Workbench. 
![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/google1.png)

I then created a database named 'jason' using
```
create database `jason`;
```
Followed by using the database with
```
use `jason`;
```
I then pasted the code 
![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure4.JPG)

Showing tables

![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure5.JPG)

Creating a table

![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/mysql1.JPG)

Seeing what's inside a table

![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure7.JPG)

Generating the Entity-Relation Diagram
![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure6.JPG)

## Azure

I set up a server on Microsoft Azure that would work with mySQL Workbench,
![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure1.JPG)

I followed the instructions provided by Microsoft

![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure2.JPG)

This is what my server looked like
![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure3.png)

Now I follow similar steps to using mySQL Workbench.

Showing tables

![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure5.JPG)

Creating a table

![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/mysql1.JPG)

Seeing what's inside a table

![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/azure7.JPG)


Generating the Entity-Relation Diagram

![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/mysql2.JPG)

# Python Script for Database Interaction

I used Google CloudShell to create the connection between mySQL database sitting in GCP to Python. I created a python file that contained the following code: 
```
import os
from dotenv import load_dotenv
from sqlalchemy import create_engine, inspect
from faker import Faker
from pandas import read_sql

load_dotenv()  # Load environment variables from .env file

# Database connection settings from environment variables
DB_HOST = os.getenv("DB_HOST")
DB_DATABASE = os.getenv("DB_DATABASE")
DB_USERNAME = os.getenv("DB_USERNAME")
DB_PASSWORD = os.getenv("DB_PASSWORD")
DB_PORT = int(os.getenv("DB_PORT", 3306))
DB_CHARSET = os.getenv("DB_CHARSET", "utf8mb4")

# Connection string
conn_string = (
    f"mysql+pymysql://{DB_USERNAME}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_DATABASE}"
    f"?charset={DB_CHARSET}"
)

# Create a database engine
db_engine = create_engine(conn_string, echo=False)


def get_tables(engine):
    """Get list of tables."""
    inspector = inspect(engine)
    return inspector.get_table_names()

def execute_query_to_dataframe(query: str, engine):
    """Execute SQL query and return result as a DataFrame."""
    return read_sql(query, engine)


# Example usage
tables = get_tables(db_engine)
print("Tables in the database:", tables)

sql_query = "SELECT * FROM patients"  # Modify as per your table
df = execute_query_to_dataframe(sql_query, db_engine)
print(df)
```
I had forgotten to use the sudo bash install command:
```
sudo apt-get install python3-dev default-libmysqlclient-dev
```
I believe I had to import "inspect", and "read_sql" because I had forgotten that step. However, the code worked after including those packages. 
I also specied ion the bottom line sql_query to select all from the patients table within the database named "jason". 

Prior to running this code, I made sure to create a dotenv file that contained login information to my GCP database instance.
```
DB_HOST=your_host
DB_DATABASE=your_database_name
DB_USERNAME=your_username
DB_PASSWORD=your_password
DB_PORT=3306
DB_CHARSET=utf8mb4
```
Something to note is that the DB_USERNAME is equal to "root" when using GCP as I did not stray from the default. 

I also included a gitignore file that ignored the .env file to avoid sharing my passwords. 

**After running the code, I was successfully able to connect to my database within GCP.**
![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/results.png)

I then attempted to populate data into the table. However, I was not able to do so. 

I used the following code:
```
import os
from dotenv import load_dotenv
from sqlalchemy import create_engine
from faker import Faker

# Load environment variables
load_dotenv()

# Database connection settings from environment variables
DB_HOST = os.getenv("DB_HOST")
DB_DATABASE = os.getenv("DB_DATABASE")
DB_USERNAME = os.getenv("DB_USERNAME")
DB_PASSWORD = os.getenv("DB_PASSWORD")
DB_PORT = int(os.getenv("DB_PORT", 3306))
DB_CHARSET = os.getenv("DB_CHARSET", "utf8mb4")

# Connection string
conn_string = (
    f"mysql+pymysql://{DB_USERNAME}:{DB_PASSWORD}@{DB_HOST}:{DB_PORT}/{DB_DATABASE}"
    f"?charset={DB_CHARSET}"
)

# Create a database engine
db_engine = create_engine(conn_string, echo=False)
fake = Faker()

def insert_fake_data(engine, num_records=100):
    # Start a connection
    with engine.connect() as connection:
        # Insert fake data into patients
        for _ in range(num_records):
            MRN = fake.unique.random_number(digits=10)
            connection.execute(f"INSERT INTO patients (MRN) VALUES ('{MRN}')")
        
        # Fetch all patient IDs
        patient_ids = [row[0] for row in connection.execute("SELECT patient_id FROM patients").fetchall()]
        
        # Insert fake data into demographics
        for patient_id in patient_ids:
            first_name = fake.first_name()
            last_name = fake.last_name()
            
            # Correct date formatting
            date_of_birth = fake.date_of_birth(minimum_age=10, maximum_age=90).strftime('%Y-%m-%d')
            
            address = fake.address().replace('\n', ', ')
            phone_number = fake.phone_number()
            email = fake.email()
            connection.execute(f"""
                INSERT INTO demographics (patient_id, first_name, last_name,
                                date_of_birth, address, phone_number, email)
                VALUES ({patient_id}, '{first_name}', '{last_name}', '{date_of_birth}', 
                '{address}', '{phone_number}', '{email}') 
            """)

if __name__ == "__main__":
    insert_fake_data(db_engine)
    print("Fake data insertion complete!")
```
However, the code failed when it tried to input data into "MRN" within the table "patients". It outputted this error: 
![](https://github.com/jas-tang/mysql_cloudmanaged_databases/blob/main/images/error.png)

Since the SQL code listed the "MRN" as a varchar, I tried changing the aforementioned code into a string or changing it entirely to input a string as I thought it was a formatting error with the table. 
```
 for _ in range(num_records):
            MRN = str(fake.unique.random_number(digits=10))
            connection.execute(f"INSERT INTO patients (MRN) VALUES ('{MRN}')")
```
However, it did not fix the problem.

My assumption is that there is an issue with either the engiene created or that the formatting of the SQL table was incorrect. 









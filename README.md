# ETL Using Shell Scripts – PostgreSQL Users Table

This repository contains an example ETL (Extract–Transform–Load) workflow implemented using a shell script and PostgreSQL.

The lab demonstrates how to:

- Extract user account data from `/etc/passwd`
- Transform the data from colon-delimited to CSV format
- Load the transformed data into a PostgreSQL table using `psql`
---

## Exercise: Build an ETL Script (`csv2db.sh`)

### 1. Start the PostgreSQL Server

1. From the **SkillsNetwork tools**, under **Databases**, choose **PostgresSQL Database server**.
2. Click **Start** to start the server (this may take a few minutes).
3. Once the server is running, click **PostgresSQL CLI** to start interacting with the PostgreSQL server.
4. This launches the interactive `psql` client, which connects to PostgreSQL and shows a prompt like:

   ```text
   postgres=#
   ```

---

### 2. Create the Shell Script File

1. Open a **new Terminal** in the lab environment.
2. Create a new shell script named `csv2db.sh`:

   ```bash
   touch csv2db.sh
   ```

3. Open `csv2db.sh` in the editor and add the following header comments:

   ```bash
   # This script
   # Extracts data from /etc/passwd file into a CSV file.
   # The csv data file contains the user name, user id and
   # home directory of each user account defined in /etc/passwd
   # Transforms the text delimiter from ":" to ",".
   # Loads the data from the CSV file into a table in PostgreSQL database.
   ```

4. Save the file (`Ctrl+S` or **File → Save**).

---

### 3. Extract Phase – Get Data from `/etc/passwd`

Add the following lines at the end of `csv2db.sh`:

```bash
# Extract phase
echo "Extracting data"
# Extract the columns 1 (user name), 3 (user id) and 
# 6 (home directory path) from /etc/passwd
cut -d":" -f1,3,6 /etc/passwd
```

Run the script:

```bash
bash csv2db.sh
```

Verify that the output contains the three fields you extracted.

---

### 4. Redirect Extracted Data to a File

Modify the `cut` command so the extracted data is saved into `extracted-data.txt`:

```bash
cut -d":" -f1,3,6 /etc/passwd > extracted-data.txt
```

Run the script again:

```bash
bash csv2db.sh
```

Confirm the file was created and contains data:

```bash
cat extracted-data.txt
```

---

### 5. Transform Phase – Convert `:` to `,` (CSV)

Add the following lines at the end of `csv2db.sh`:

```bash
# Transform phase
echo "Transforming data"
# Read the extracted data and replace the colons with commas.
tr ":" "," < extracted-data.txt > transformed-data.csv
```

Run the script:

```bash
bash csv2db.sh
```

Verify the CSV file:

```bash
cat transformed-data.csv
```

---

### 6. Load Phase – Load CSV into PostgreSQL

You will use the `psql` client utility in a non-interactive manner by sending database commands through a pipeline.

The general PostgreSQL `COPY` command syntax is:

```sql
COPY table_name FROM 'filename' DELIMITERS 'delimiter_character' CSV;
```

Add the following lines at the end of `csv2db.sh`:

```bash
# Load phase
echo "Loading data"
# Set the PostgreSQL password environment variable.
# Replace <yourpassword> with your actual PostgreSQL password.
export PGPASSWORD=<yourpassword>;
# Send the instructions to connect to 'template1' and
# copy the file to the table 'users' through command pipeline.
echo "\c template1;\COPY users FROM '/home/project/transformed-data.csv' DELIMITERS ',' CSV;" | psql --username=postgres --host=postgres
```

> Note: Update the path to `transformed-data.csv` if it differs in your environment.

---

### 7. Create the `users` Table in PostgreSQL

In the **PostgresSQL CLI** (interactive `psql`):

1. Connect to the `template1` database from the `postgres=#` prompt:

   ```sql
   \c template1
   ```

   You should see:

   ```text
   You are now connected to database "template1" as user "postgres".
   ```

   The prompt changes to:

   ```text
   template1=#
   ```

2. Create the `users` table:

   ```sql
   CREATE TABLE users(
       username      VARCHAR(50),
       userid        INT,
       homedirectory VARCHAR(100)
   );
   ```

   On success you will see:

   ```text
   CREATE TABLE
   ```

3. Run the script to load the data:

   ```bash
   bash csv2db.sh
   ```

---

### 8. Verify Data Load from the Script

Append the following line to the end of `csv2db.sh`:

```bash
echo "SELECT * FROM users;" | psql --username=postgres --host=postgres template1
```

Run the script again:

```bash
bash csv2db.sh
```

You should see the contents of the `users` table printed, populated from the `transformed-data.csv` file.

---

## Final `csv2db.sh` Example

Below is an example of what the complete `csv2db.sh` might look like (remember to update `<yourpassword>` and the CSV path as needed):

```bash
# This script
# Extracts data from /etc/passwd file into a CSV file.
# The csv data file contains the user name, user id and
# home directory of each user account defined in /etc/passwd
# Transforms the text delimiter from ":" to ",".
# Loads the data from the CSV file into a table in PostgreSQL database.

# Extract phase
echo "Extracting data"
# Extract the columns 1 (user name), 3 (user id) and 
# 6 (home directory path) from /etc/passwd
cut -d":" -f1,3,6 /etc/passwd > extracted-data.txt

# Transform phase
echo "Transforming data"
tr ":" "," < extracted-data.txt > transformed-data.csv

# Load phase
echo "Loading data"
# Replace <yourpassword> with your actual PostgreSQL password
export PGPASSWORD=<yourpassword>;
echo "\c template1;\COPY users FROM '/home/project/transformed-data.csv' DELIMITERS ',' CSV;" | psql --username=postgres --host=postgres

# Verification
echo "Verifying data in users table"
echo "SELECT * FROM users;" | psql --username=postgres --host=postgres template1
```

---

## Summary

1. Extracted user data from `/etc/passwd` using `cut`
2. Transformed colon-delimited text to CSV using `tr`
3. Loaded the transformed data into a PostgreSQL table using `psql` and `COPY`
4. Verified the loaded data by querying the `users` table

This provides a simple but complete ETL pipeline implemented entirely with shell scripting and PostgreSQL.

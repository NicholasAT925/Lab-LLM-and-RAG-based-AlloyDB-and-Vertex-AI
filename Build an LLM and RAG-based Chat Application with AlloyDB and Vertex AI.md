---
Category: AI Programming
Sub-Category: 
tags:
  - AI_Programming
  - Incomplete
  - Large_Language_Models_LLMs
  - Retrieval_Augmented_Generation_RAG
  - AlloyDB
  - Vertex_AI
Created At: 2025-05-24
Last Updated: 2025-05-25T16:28
Source Links: 
---
# <div class="divider">AI Programming</div>
# Source
[Source](https://www.cloudskillsboost.google/focuses/97381?catalog_rank=%7B"rank"%3A1%2C"num_filters"%3A0%2C"has_search"%3Atrue%7D&parent=catalog&search_id=46102807)
# LLM
[[Large Language Model (LLMs)]]
# RAG
[[Retrieval Augmented Generation (RAG)]]
# AlloyDB
[[AlloyDB Content Page|AlloyDB]]
# Vertex AI
[[Vertex AI]]

# 1. Install PostgreSQL

**_Insatll_**

```bash
sudo apt-get update
sudo apt-get install --yes postgresql-client
```

- `sudo`: Runs the command with **administrator (root)** privileges.
- `apt-get update`: Updates your system’s **package list** from all configured sources (like Ubuntu's software repositories).
- ✅ This ensures you're installing the **latest available version** of any package.
- `apt-get install`: Tells your system to **install a package**.
- `--yes`: Automatically answers "yes" to any prompts (so you don’t have to manually confirm).
- `postgresql-client`: The name of the package — this installs the **PostgreSQL client**, which gives you tools like:
    - `psql`: The PostgreSQL command-line tool
    - Useful for connecting to and managing remote PostgreSQL databases (e.g. AlloyDB or local PostgreSQL servers)
# 2. Create the vector database
## 1: Create the Database
Use the PostgreSQL client to create the AlloyDB database and enable vector embeddings.
Connected to an **AlloyDB** instance using the `psql` PostgreSQL client to:
1. **Create a new database**
2. **Enable vector search capabilities** using the `vector` extension

```shell
export PGPASSWORD=PG_PASSWORD
psql "host=$INSTANCE_IP user=$PGUSER" -c "CREATE DATABASE assistantdemo"
```

| Part                                 | Description                                             |
| ------------------------------------ | ------------------------------------------------------- |
| `export PGPASSWORD=PG_PASSWORD`      | Sets the PostgreSQL password as an environment variable |
| `psql "host=... user=..."`           | Connects to the AlloyDB instance                        |
| `-c "CREATE DATABASE assistantdemo"` | Creates a new database named `assistantdemo`            |
## Step 2: Enable Vector Embeddings
Creating the `assistantdemo` database gives your retrieval app a dedicated PostgreSQL-compatible environment to store and query structured and vector data.

```shell
psql "host=$INSTANCE_IP user=$PGUSER dbname=assistantdemo" -c "CREATE EXTENSION vector"
```

|Part|Description|
|---|---|
|`dbname=assistantdemo`|Connects specifically to the newly created database|
|`CREATE EXTENSION vector`|Enables the `vector` extension needed for similarity search|
## Why Vector Embeddings Are Important

- Vector embeddings allow you to store semantic representations (e.g. of text, images).
- This is essential for building **semantic search**, **RAG**, and **recommendation systems**.
- The `vector` extension (e.g. pgvector) adds support for operations like:
    - `cosine similarity`
    - `euclidean distance`
    - `dot product`

# 3. Install Python
Install Python in the VM. Python is used to populate the database.
Install and then run 
```shell
sudo apt install -y python3.11-venv git
python3 -m venv .venv
source ~/.venv/bin/activate
pip install --upgrade pip
```

Confirm in installation
```shell
python -V

(.venv) student@app-vm:~$ python -V
Python 3.11.2
(.venv) student@app-vm:~$
```

# 4. Populate the example database

Sets up and populates **vector-enabled AlloyDB** instance with sample data for use in a **chat + retrieval application**.

## Step 1: Clone the Project Repository
Clones the full retrieval + app pipeline including code, data, and setup files. Not really applicable everywhere as this clone changes.
```shell
git clone https://github.com/GoogleCloudPlatform/genai-databases-retrieval-app.git
```
## Step 2: Explore the Data Models
```shell
cd ~/genai-databases-retrieval-app
cat retrieval_service/models/models.py
```

| Part                                 | Explanation                                                              |
| ------------------------------------ | ------------------------------------------------------------------------ |
| `cat`                                | Linux command to **print the contents** of a file to the terminal        |
| `retrieval_service/models/models.py` | Path to the file containing **Python data models** for the retrieval app |
| `cd`                                 | **Change Directory** – the command to move between folders               |
| `~`                                  | Your **home directory** (e.g., `/home/your-username`)                    |
| `genai-databases-retrieval-app`      | The project folder cloned from GitHub                                    |
These are implemented as **Python classes** (likely SQLAlchemy or Pydantic) and define how each table is structured. You (as a human) **visually inspect** this file — it's not running code, just showing it.

## Step 3: View Sample Airport Data
```shell
head -1 data/airport_dataset.csv
grep SFO data/airport_dataset.csv
```

- First command shows the **CSV header**
- Second command displays the **row for San Francisco International (SFO)**
>[!note] 
>No Vector embedding yet

| Line of Code                              | Component                        | Explanation                                                                 |
|-------------------------------------------|----------------------------------|-----------------------------------------------------------------------------|
| `head -1 data/airport_dataset.csv`        | `head`                           | A Linux command to print the first N lines of a file (default is 10).       |
|                                           | `-1`                             | Flag that tells `head` to show only **1 line**.                            |
|                                           | `data/airport_dataset.csv`       | The path to the file you want to preview.                                  |
|                                           | → **Result**                     | Shows the **first line** of the CSV file, usually the column headers.      |

| Line of Code                              | Component                        | Explanation                                                                 |
|-------------------------------------------|----------------------------------|-----------------------------------------------------------------------------|
| `grep SFO data/airport_dataset.csv`       | `grep`                           | A Linux tool to **search for text patterns** in files.                     |
|                                           | `SFO`                            | The **search string**, in this case, the airport code for San Francisco.   |
|                                           | `data/airport_dataset.csv`       | The path to the file to be searched.                                       |
|                                           | → **Result**                     | Displays all lines in the file containing the text `SFO`.                  |
## Step 4: View Sample Flights Data
```shell
head -1 data/flights_dataset.csv
grep -m10 "SFO" data/flights_dataset.csv
```

- First line: column names for flight records
- Next lines: first 10 entries **to/from SFO**

| Component                     | Description                                                                 |
|------------------------------|-----------------------------------------------------------------------------|
| `head`                       | A command to print the **first few lines** of a file.                       |
| `-1`                         | Flag to print **only the first line**.                                      |
| `data/flights_dataset.csv`   | The path to the file being read.                                            |
| → **Result**                 | Displays the **column headers** of the flights dataset (1st line of file).  |

| Component                     | Description                                                                 |
|------------------------------|-----------------------------------------------------------------------------|
| `grep`                       | Searches for lines matching a string or pattern.                            |
| `-m10`                       | Limits the output to the **first 10 matches** only.                         |
| `"SFO"`                      | The **search pattern**, i.e., lines containing `SFO`.                       |
| `data/flights_dataset.csv`   | The file to search in.                                                      |
| → **Result**                 | Returns the **first 10 lines** in the dataset that mention `"SFO"`.         |

## Step 5: View Sample Amenities Data
```shell
head -2 data/amenity_dataset.csv
```

| Component                     | Description                                                                 |
|------------------------------|-----------------------------------------------------------------------------|
| `head`                       | A Unix/Linux command that displays the **first few lines** of a file.       |
| `-2`                         | Option that tells `head` to show the **first 2 lines** only.                |
| `data/amenity_dataset.csv`   | Path to the CSV file you want to preview.                                  |
| → **Result**                 | Prints **line 1** (usually the header) and **line 2** (first data row).     |
You'll notice that the first amenity has several simple values, including name, description, location, terminal, category, and business hours. The next value is `content`, which incorporates the name, description, and location. The last value is `embedding`, the vector embedding for the row.

The embedding is an array of 768 numbers which is used when performing a semantic search. These embeddings are calculated using an AI model provided by Vertex AI. When a user provides a query, a vector embedding can be created from the query, and data with vector embeddings that are close to the search's embedding can be retrieved.

## Step 6: Configure the Database Connection
```shell
export PGUSER=PG_USER
export PGPASSWORD=PG_PASSWORD
export PROJECT_ID=$(gcloud config get-value project)
export REGION=REGION
export ADBCLUSTER=CLUSTER

export INSTANCE_IP=$(gcloud alloydb instances describe $ADBCLUSTER-pr \
  --cluster=$ADBCLUSTER \
  --region=$REGION \
  --format="value(ipAddress)")

cd ~/genai-databases-retrieval-app/retrieval_service
cp example-config.yml config.yml

sed -i s/127.0.0.1/$INSTANCE_IP/g config.yml
sed -i s/my-user/$PGUSER/g config.yml
sed -i s/my-password/$PGPASSWORD/g config.yml
sed -i s/my_database/assistantdemo/g config.yml

cat config.yml
```


This script:
- Sets DB connection credentials using environment variables
- Fetches AlloyDB instance IP from GCP
- Copies a template config and customizes it with real values
- Outputs the resulting `config.yml` for verification

| Line of Code                             | Explanation                                                                 |
|------------------------------------------|-----------------------------------------------------------------------------|
| `export PGUSER=PG_USER`                  | Sets the Postgres username as an environment variable.                      |
| `export PGPASSWORD=PG_PASSWORD`          | Sets the Postgres password as an environment variable.                      |
| `export PROJECT_ID=$(gcloud config get-value project)` | Dynamically retrieves the active GCP project ID and stores it.            |
| `export REGION=REGION`                   | Stores the GCP region (must be replaced with actual region, e.g. `us-central1`). |
| `export ADBCLUSTER=CLUSTER`              | Stores the AlloyDB cluster name (must be replaced with actual name).        |

| Line of Code                                                              | Explanation                                                               |
| ------------------------------------------------------------------------- | ------------------------------------------------------------------------- |
| `export INSTANCE_IP=$(gcloud alloydb instances describe $ADBCLUSTER-pr \` | Uses gcloud to describe an AlloyDB instance named `<cluster>-pr`.         |
| `  --cluster=$ADBCLUSTER \`                                               | Specifies the cluster name for the AlloyDB instance.                      |
| `  --region=$REGION \`                                                    | Specifies the region where the cluster is hosted.                         |
| `  --format="value(ipAddress)")`                                          | Extracts the instance’s public IP address and stores it in `INSTANCE_IP`. |

| Line of Code                             | Explanation                                                                 |
|------------------------------------------|-----------------------------------------------------------------------------|
| `cd ~/genai-databases-retrieval-app/retrieval_service` | Navigates to the config folder of the retrieval service.                  |
| `cp example-config.yml config.yml`       | Creates a working config file by copying from a template.                   |

| Line of Code                             | Explanation                                                                 |
|------------------------------------------|-----------------------------------------------------------------------------|
| `sed -i s/127.0.0.1/$INSTANCE_IP/g config.yml`  | Replaces `127.0.0.1` with the actual AlloyDB IP in `config.yml`.         |
| `sed -i s/my-user/$PGUSER/g config.yml`         | Replaces `my-user` with the real Postgres user.                          |
| `sed -i s/my-password/$PGPASSWORD/g config.yml` | Replaces `my-password` with the real password.                           |
| `sed -i s/my_database/assistantdemo/g config.yml`| Sets the database name to `assistantdemo`.                              |

| Line of Code                             | Explanation                                                                 |
|------------------------------------------|-----------------------------------------------------------------------------|
| `cat config.yml`                         | Outputs the updated configuration file to verify changes.                  |

 `cat config.yml` output
 
```shell
host: 10.65.0.2
datastore:
  kind: "postgres"
  host: 10.65.0.2
  database: "assistantdemo"
  user: "postgres"
  password: "samplepassword"

```

## Step 7: Populate the Database with Data
```shell
pip install -r requirements.txt
python run_database_init.py
```

- `pip install ...` installs dependencies for the retrieval service
- `run_database_init.py` populates the database with.

# 5. Create a service account for the retrieval service
Do this in a new terminal
The **retrieval service** needs permission to interact with Google Cloud services (like Vertex AI and AlloyDB).
A **dedicated service account** is created to serve as the **identity of the Cloud Run deployment**.

```shell
export PROJECT_ID=$(gcloud config get-value project)
gcloud iam service-accounts create retrieval-identity
gcloud projects add-iam-policy-binding $PROJECT_ID \
--member="serviceAccount:retrieval-identity@$PROJECT_ID.iam.gserviceaccount.com" \
--role="roles/aiplatform.user"
```

- Cloud Run services need a service account to authenticate when calling other Google Cloud services.
- In this case, the service needs permission to **call Vertex AI** for embedding-based tasks.

| Command                                                 | Explanation                                                               |
| ------------------------------------------------------- | ------------------------------------------------------------------------- |
| `export PROJECT_ID=$(gcloud config get-value project)`  | Saves the currently active GCP project ID into the variable `PROJECT_ID`. |
| → Purpose                                               | Used so you don’t hardcode the project ID in later commands.              |
|                                                         |                                                                           |
| `gcloud iam service-accounts create retrieval-identity` | Creates a new service account called `retrieval-identity`.                |
| → Default email format                                  | `retrieval-identity@<PROJECT_ID>.iam.gserviceaccount.com`                 |
| → Purpose                                               | Used by your retrieval application to authenticate with GCP services.     |
|                                                         |                                                                           |
| `gcloud projects add-iam-policy-binding $PROJECT_ID \` | Adds a policy binding (IAM permission) to your current project. |
| `--member="serviceAccount:retrieval-identity@$PROJECT_ID.iam.gserviceaccount.com" \` | Specifies **which service account** the permission applies to. |
| `--role="roles/aiplatform.user"` | Grants the `aiplatform.user` role (access to Vertex AI services). |

| Action                     | Result                                                                 |
|----------------------------|------------------------------------------------------------------------|
| Export Project ID          | Avoids hardcoding your GCP project ID.                                |
| Create service account     | Adds a machine identity (`retrieval-identity`) for apps to use.        |
| Bind IAM role              | Gives it access to Vertex AI (`roles/aiplatform.user`).                |
# 6. Deploy the retrieval service to Cloud Run
This step publishes the retrieval service as a **private Cloud Run application** that interacts with AlloyDB and Vertex AI.
```shell
export REGION=REGION
cd ~/genai-databases-retrieval-app
gcloud alpha run deploy retrieval-service \
    --source=./retrieval_service/\
    --no-allow-unauthenticated \
    --service-account retrieval-identity \
    --region $REGION \
    --network=default \
    --quiet
```


| What It Does                              | Why It Matters                                                              |
|-------------------------------------------|------------------------------------------------------------------------------|
| Deploys a retrieval backend service       | Enables backend inference or data API endpoints for your GenAI app.         |
| Secures it using IAM                      | Only authenticated identities can invoke the service.                        |
| Uses custom service account               | Ensures least-privilege access via `retrieval-identity`.                     |

| Command                              | Explanation                                                                                |
| ------------------------------------ | ------------------------------------------------------------------------------------------ |
| `export REGION=REGION`               | Saves the region into an environment variable (replace `REGION` with e.g., `us-central1`). |
| → Purpose                            | Used later to specify the deployment region for Cloud Run.                                 |
|                                      |                                                                                            |
| `cd ~/genai-databases-retrieval-app` | Changes to the root folder of the GenAI retrieval application.                             |
| → Purpose                            | Ensures the relative path `./retrieval_service/` resolves correctly.                       |
|                                      |                                                                                            |
| `gcloud alpha run deploy retrieval-service \`        | Deploys a service named `retrieval-service` using **Cloud Run (Alpha)**.    |
| `--source=./retrieval_service/ \`                    | Uses the source code located in `./retrieval_service/` for deployment.     |
| `--no-allow-unauthenticated \`                       | **Restricts access** to only authenticated users (no public access).       |
| `--service-account retrieval-identity \`             | Uses the `retrieval-identity` service account for the app to access GCP.   |
| `--region $REGION \`                                 | Deploys the service in the specified region (`$REGION`).                   |
| `--network=default \`                                | Attaches the deployment to the **default VPC network**.                    |
| `--quiet`                                            | Suppresses confirmation prompts for a **non-interactive deployment**.      |

---


```shell
curl -H "Authorization: Bearer $(gcloud auth print-identity-token)" $(gcloud run services list --filter="(retrieval-service)" --format="value(URL)")
```
This command uses `curl` to make an **authenticated request** to your deployed Cloud Run service and check if it is working correctly.

|Component|Explanation|
|---|---|
|`curl`|Command-line tool for making HTTP requests.|
|`-H "Authorization: Bearer $(...)”`|Adds an HTTP header for **identity-based authentication**.|
|`$(gcloud auth print-identity-token)`|Generates a temporary **ID token** for the current user or service account.|
|`$(gcloud run services list --filter="(retrieval-service)" --format="value(URL)")`|Fetches the deployed Cloud Run service **URL** that matches `retrieval-service`.|
# 7. Register the OAuth consent screen

**OAuth 2.0 consent screen**, users will see when authorizing the application to access their data via Google login. Just look at lab ? 
# 8. Create a client ID for the application
The application uses **Google OAuth 2.0** for secure login. You’ll create a **Client ID** that links your app to Google's OAuth servers and defines allowed origins and redirect URIs.

```shell
echo "origin:"; echo "https://8080-$WEB_HOST"
echo "redirect:"; echo "https://8080-$WEB_HOST/login/google"
```

| Component                                    | Explanation                                                                            |
| -------------------------------------------- | -------------------------------------------------------------------------------------- |
| `echo "origin:"`                             | Prints the label `origin:` to the terminal.                                            |
| `echo "https://8080-$WEB_HOST"`              | Constructs and prints a URL by prepending `https://8080-` to the value of `$WEB_HOST`. |
| → Example Output                             | `https://8080-your-session-id.preview.domain`                                          |
| → Purpose                                    | Typically represents the base URL of your running service (e.g. backend).              |

| Component                                    | Explanation                                                                            |
| -------------------------------------------- | -------------------------------------------------------------------------------------- |
| `echo "redirect:"`                           | Prints the label `redirect:` to the terminal.                                          |
| `echo "https://8080-$WEB_HOST/login/google"` | Constructs and prints the **Google OAuth redirect URL** for login.                     |
| → Example Output                             | `https://8080-your-session-id.preview.domain/login/google`                             |
| → Purpose                                    | Used as a **redirect URI** for authentication callbacks (e.g. Google Sign-In).         |

# 9. Deploy the sample application
launches a **FastAPI-based chat interface** that interacts with your deployed **retrieval service**. 
It runs on your **VM** and supports natural language queries about flights and amenities.

```shell
cd ~/genai-databases-retrieval-app/llm_demo
pip install -r requirements.txt
```

- Go to directory
- install the files named in `requirement.txt`

```shell
export BASE_URL=$(gcloud run services list \
  --filter="(retrieval-service)" \
  --format="value(URL)")
echo $BASE_URL
```

- Stores the Cloud Run endpoint for the retrieval service in the `BASE_URL` variable
- Verifies it by printing to the console

|Component|Explanation|
|---|---|
|`export BASE_URL=`|Declares and sets the variable `BASE_URL`.|
|`$(...)`|Runs the inner command and assigns its result to `BASE_URL`.|
|`gcloud run services list`|Lists all deployed Cloud Run services.|
|`--filter="(retrieval-service)"`|Filters the output to match a service named `retrieval-service`.|
|`--format="value(URL)"`|Extracts just the **URL** of the matched service (e.g., `https://...`).|
|→ Result|`BASE_URL` now contains the full URL of your `retrieval-service`.|

| Component        | Explanation                                              |
| ---------------- | -------------------------------------------------------- |
| `echo $BASE_URL` | Prints the stored Cloud Run service URL.                 |
| → Use Case       | Helpful for testing, debugging, or setting up API calls. |

---

```shell
gcloud compute ssh app-vm --zone=ZONE -- -L 8080:localhost:8081
```

This sets up a **secure SSH tunnel**, mapping:
- Cloud Shell `localhost:8080` → VM `localhost:8081`

|Component|Explanation|
|---|---|
|`gcloud compute ssh`|Starts an SSH session with a VM instance on Google Compute Engine.|
|`app-vm`|Name of the virtual machine instance you are connecting to.|
|`--zone=ZONE`|Specifies the **GCP zone** (e.g., `us-central1-a`) where the VM is located.|
|`--`|Signals that all following arguments are meant for the SSH command itself.|
|`-L 8080:localhost:8081`|Sets up **port forwarding**: forwards local port `8080` to port `8081` on the VM.|
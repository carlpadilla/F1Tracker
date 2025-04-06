# F1TrackerApp

## Overview

**F1TrackerApp** is a mobile application built using React Native and Expo, designed to track Formula 1 race results, sprint race results, and driver standings for the 2025 season. The backend, which is the focus of this section, is powered by an Azure Function App (`function_app.py`) that fetches race data from an external F1 API, processes it, and stores it in Azure Cosmos DB for the frontend to consume.

---

## Why I Started This Project

As a Formula 1 enthusiast and aspiring full-stack developer, I started `F1TrackerApp` in early 2025 to combine my passion for F1 with my goal of learning modern development technologies. The backend was a critical component, allowing me to practice API integration, data management, and cloud services while providing the data needed for the app’s frontend.

### Learning Goals (Backend-Specific)
When I began working on the backend, I aimed to learn the following:
1. **Azure Function Apps**: Understand how to create and deploy serverless functions using Azure Function Apps.
2. **Azure Cosmos DB**: Learn to use Cosmos DB for NoSQL data storage and querying.
3. **API Integration**: Practice fetching and processing data from external APIs, specifically an F1 data source like the Ergast F1 API.
4. **Data Management**: Handle real-time race data, including race and sprint sessions, and address issues like duplicates and data normalization.
5. **Python for Backend**: Improve my Python skills for backend development, focusing on libraries like `requests` and `azure-cosmos`.

### What I Learned (Backend-Specific)
Building the backend taught me the following:
1. **Azure Function Apps**:
   - I learned how to set up and deploy a serverless function using Azure Function Apps, including configuring HTTP triggers.
   - I gained experience in managing environment variables for secure access to Cosmos DB credentials.
2. **Azure Cosmos DB**:
   - I became proficient in setting up Cosmos DB, creating databases and containers, and using SQL-like queries to manage data.
   - I resolved duplicate data issues by writing queries and scripts to clean up the database.
3. **API Integration**:
   - I successfully fetched data from an external F1 API using Python’s `requests` library and processed it for storage.
   - I handled API inconsistencies, such as missing fields, by adding fallback logic.
4. **Data Management**:
   - I implemented logic to differentiate between race and sprint sessions using `session_key` and `session_type` fields.
   - I addressed duplicate entries in Cosmos DB, which were causing issues in the frontend, by cleaning up the database.
5. **Python for Backend**:
   - I improved my Python skills, particularly in working with external APIs and NoSQL databases using the `azure-cosmos` library.
   - I learned how to structure a serverless function to handle data fetching and storage efficiently.

---

## Backend Development: Step-by-Step Guide

Below is a detailed guide on how I built the backend for `F1TrackerApp` using an Azure Function App (`function_app.py`) to fetch, process, and store F1 race data in Azure Cosmos DB.

### Step 1: Set Up Azure Services
1. **Create an Azure Account and Resource Group**:
   - I signed up for an Azure account and created a resource group named `F1TrackerResources` to organize my project’s resources.
   - This resource group would house both the Cosmos DB account and the Function App.

2. **Set Up Azure Cosmos DB**:
   - I created an Azure Cosmos DB account in the Azure portal:
     - Selected the Core (SQL) API for NoSQL data storage.
     - Named the account `f1tracker-cosmos`.
   - I created a database named `F1Data` within the Cosmos DB account.
   - Inside `F1Data`, I created a container named `RaceData` with a partition key of `/session_key` to optimize queries based on session identifiers.
   - I noted down the Cosmos DB URI and primary key from the “Keys” section in the Azure portal for use in the Function App.

3. **Set Up Azure Function App**:
   - I created an Azure Function App in the Azure portal:
     - Named it `f1-tracker-api`.
     - Selected Python 3.8 as the runtime stack.
     - Chose a consumption plan for serverless scaling.
   - I installed the Azure Functions Core Tools locally to develop and deploy the Function App:
     ```bash
     npm install -g azure-functions-core-tools@4 --unsafe-perm true
     ```
   - I verified the installation:
     ```bash
     func --version
     ```

### Step 2: Create the Azure Function App Locally
1. **Initialize the Function App Project**:
   - I created a new directory for the Function App and initialized a project using VS Code with the Azure Functions extension:
     ```bash
     mkdir f1-tracker-backend
     cd f1-tracker-backend
     ```
   - In VS Code, I used the Azure Functions extension to create a new project:
     - Selected Python as the language.
     - Chose HTTP trigger as the template to create a function that responds to HTTP requests.
     - Named the function `FetchRaceData`.

2. **Install Required Python Libraries**:
   - I navigated to the project directory and activated a virtual environment:
     ```bash
     python -m venv .venv
     source .venv/bin/activate  # On Windows: .venv\Scripts\activate
     ```
   - I installed the necessary Python libraries by updating the `requirements.txt` file:
     ```
     azure-functions
     requests
     azure-cosmos
     ```
   - I installed the dependencies:
     ```bash
     pip install -r requirements.txt
     ```

### Step 3: Write the Data Fetching and Storage Logic (`function_app.py`)
1. **Implement the Core Logic**:
   - I modified the generated `function_app.py` (originally named `FetchRaceData/__init__.py`) to fetch race data from an external F1 API and store it in Cosmos DB. Here’s the complete code:
     ```python
     import azure.functions as func
     import requests
     from azure.cosmos import CosmosClient, PartitionKey
     import os
     import logging

     def main(req: func.HttpRequest) -> func.HttpResponse:
         logging.info('Python HTTP trigger function processed a request.')

         # Initialize Cosmos DB client
         cosmos_uri = os.environ.get('COSMOS_URI')
         cosmos_key = os.environ.get('COSMOS_KEY')
         if not cosmos_uri or not cosmos_key:
             return func.HttpResponse(
                 "Cosmos DB credentials not found in environment variables.",
                 status_code=500
             )

         client = CosmosClient(cosmos_uri, cosmos_key)
         db = client.get_database_client('F1Data')
         container = db.get_container_client('RaceData')

         try:
             # Fetch race data from an external F1 API (e.g., Ergast API or mock API)
             response = requests.get("https://f1-data-api.example.com/2025/races")
             if response.status_code != 200:
                 return func.HttpResponse(
                     f"Failed to fetch race data: {response.status_code}",
                     status_code=response.status_code
                 )

             race_data = response.json()

             # Process and store each race result
             for item in race_data:
                 # Add session_key and normalize session_type
                 round_number = item.get('round', 'unknown')
                 session_type = item.get('session_type', 'Race').lower()
                 item['session_key'] = f"2025_{round_number}_{session_type}"
                 item['session_type'] = session_type

                 # Ensure required fields are present
                 item['id'] = f"{item['session_key']}_{item.get('Driver', 'unknown')}"
                 item['raceName'] = item.get('raceName', 'Unknown Grand Prix')
                 item['race_date'] = item.get('race_date', '2025-01-01')
                 item['Standing'] = item.get('Standing', 0)
                 item['Driver Number'] = item.get('Driver Number', 'N/A')
                 item['Driver'] = item.get('Driver', 'Unknown Driver')
                 item['Team'] = item.get('Team', 'Unknown Team')
                 item['Fastest Lap'] = item.get('Fastest Lap', 'N/A')
                 item['Points'] = item.get('Points', 0)

                 # Upsert the item into Cosmos DB
                 container.upsert_item(item)
                 logging.info(f"Stored item with session_key: {item['session_key']}")

             return func.HttpResponse(
                 "Race data fetched and stored successfully.",
                 status_code=200
             )

         except Exception as e:
             logging.error(f"Error processing race data: {str(e)}")
             return func.HttpResponse(
                 f"Error processing race data: {str(e)}",
                 status_code=500
             )
     ```
   - I added error handling to manage API failures and Cosmos DB connection issues.
   - I used `os.environ` to securely access Cosmos DB credentials via environment variables.

2. **Handle Race and Sprint Sessions**:
   - I ensured the function handled both race and sprint sessions by adding a `session_key` (e.g., `2025_1_Race`, `2025_2_Sprint`) and normalizing the `session_type` field to lowercase (`"race"` or `"sprint"`).
   - I added fallback values for missing fields to prevent errors during storage, such as defaulting `session_type` to `"Race"` if not provided by the API.

3. **Test the Function Locally**:
   - I set up environment variables locally in the `local.settings.json` file:
     ```json
     {
       "IsEncrypted": false,
       "Values": {
         "AzureWebJobsStorage": "",
         "FUNCTIONS_WORKER_RUNTIME": "python",
         "COSMOS_URI": "your-cosmos-uri",
         "COSMOS_KEY": "your-cosmos-key"
       }
     }
     ```
   - I replaced `"your-cosmos-uri"` and `"your-cosmos-key"` with the actual values from the Azure portal.
   - I ran the Function App locally:
     ```bash
     func start
     ```
   - I tested the endpoint at `http://localhost:7071/api/FetchRaceData` using a browser and Postman, ensuring it fetched data and stored it in Cosmos DB.
   - I queried Cosmos DB to verify the data:
     ```sql
     SELECT * FROM c WHERE c.raceName = "Australian Grand Prix"
     ```
     - Expected: 20 items with `session_key: "2025_1_Race"` and `session_type: "race"`.
     ```sql
     SELECT * FROM c WHERE c.raceName = "Chinese Grand Prix"
     ```
     - Expected: 40 items (20 with `session_key: "2025_2_Race"`, `session_type: "race"`, and 20 with `session_key: "2025_2_Sprint"`, `session_type: "sprint"`).

### Step 4: Deploy the Function App to Azure
1. **Configure Environment Variables in Azure**:
   - In the Azure portal, I navigated to the Function App (`f1-tracker-api`) and added the Cosmos DB credentials as application settings:
     - `COSMOS_URI`: My Cosmos DB URI.
     - `COSMOS_KEY`: My Cosmos DB primary key.
   - I saved the settings to ensure the Function App could access Cosmos DB when deployed.

2. **Deploy the Function App**:
   - I deployed the Function App to Azure using the Azure Functions Core Tools:
     ```bash
     func azure functionapp publish f1-tracker-api
     ```
   - The deployment process uploaded the function code and dependencies to Azure.
   - I tested the deployed endpoint (`https://f1-tracker-api.azurewebsites.net/api/FetchRaceData`) using Postman to ensure it returned a 200 status code and the message: `"Race data fetched and stored successfully."`.

3. **Verify Data in Cosmos DB**:
   - I queried Cosmos DB to confirm the data was stored correctly:
     ```sql
     SELECT * FROM c
     ```
   - I saw entries for the Australian Grand Prix (items 20-39 after cleanup) and Chinese Grand Prix (items 60-99 after cleanup), with proper `session_key` and `session_type` fields.

### Step 5: Handle Duplicate Data Issues
1. **Identify Duplicates**:
   - I noticed duplicate entries in Cosmos DB after running the Function App multiple times:
     - Items 0-19 and 20-39 for the Australian Grand Prix were identical except for `session_key`.
     - Items 40-59 and 60-79 for the Chinese Grand Prix race results were duplicates.
   - I queried Cosmos DB to identify old entries without a `session_type` field:
     ```sql
     SELECT * FROM c WHERE NOT IS_DEFINED(c.session_type)
     ```

2. **Clean Up Duplicates**:
   - I manually deleted the duplicate items (0-19, 40-59) using the Cosmos DB Data Explorer by selecting and deleting each item.
   - Alternatively, I wrote a Python script to automate the cleanup:
     ```python
     from azure.cosmos import CosmosClient
     import os

     cosmos_uri = os.environ.get('COSMOS_URI')
     cosmos_key = os.environ.get('COSMOS_KEY')
     client = CosmosClient(cosmos_uri, cosmos_key)
     db = client.get_database_client('F1Data')
     container = db.get_container_client('RaceData')

     query = "SELECT * FROM c WHERE NOT IS_DEFINED(c.session_type)"
     items = list(container.query_items(query=query, enable_cross_partition_query=True))
     for item in items:
         container.delete_item(item['id'], partition_key=item['session_key'])
         print(f"Deleted item with id: {item['id']}")
     ```
   - I ran the script to delete the old entries, leaving only the updated items:
     - Items 20-39: Australian Grand Prix Race (`session_key: "2025_1_Race"`, `session_type: "race"`)
     - Items 60-79: Chinese Grand Prix Race (`session_key: "2025_2_Race"`, `session_type: "race"`)
     - Items 80-99: Chinese Grand Prix Sprint (`session_key: "2025_2_Sprint"`, `session_type: "sprint"`)

3. **Prevent Future Duplicates**:
   - I modified `function_app.py` to use `upsert_item` instead of `create_item`, ensuring that items with the same `id` (e.g., `2025_1_Race_Lewis Hamilton`) would be updated rather than duplicated.
   - I added logging to track each upsert operation:
     ```python
     logging.info(f"Stored item with session_key: {item['session_key']}")
     ```

### Step 6: Test the Backend with the Frontend
1. **Update the API Endpoint**:
   - I created a new HTTP trigger function in the same Function App to serve the stored data to the frontend, named `GetRaceData`:
     ```python
     import azure.functions as func
     from azure.cosmos import CosmosClient
     import os
     import json
     import logging

     def main(req: func.HttpRequest) -> func.HttpResponse:
         logging.info('Fetching race data from Cosmos DB.')

         cosmos_uri = os.environ.get('COSMOS_URI')
         cosmos_key = os.environ.get('COSMOS_KEY')
         client = CosmosClient(cosmos_uri, cosmos_key)
         db = client.get_database_client('F1Data')
         container = db.get_container_client('RaceData')

         try:
             query = "SELECT * FROM c"
             items = list(container.query_items(query=query, enable_cross_partition_query=True))
             return func.HttpResponse(
                 json.dumps(items),
                 mimetype="application/json",
                 status_code=200
             )
         except Exception as e:
             logging.error(f"Error fetching race data: {str(e)}")
             return func.HttpResponse(
                 f"Error fetching race data: {str(e)}",
                 status_code=500
             )
     ```
   - I deployed this new function alongside `FetchRaceData`:
     ```bash
     func azure functionapp publish f1-tracker-api
     ```
   - The endpoint `https://f1-tracker-api.azurewebsites.net/api/GetRaceData` was used by the frontend to fetch race data.

2. **Verify Data Availability**:
   - I tested the `GetRaceData` endpoint to ensure it returned all race data in JSON format, including both race and sprint results for the Chinese Grand Prix.
   - I confirmed that the frontend could fetch this data, although I later encountered issues with rendering the Chinese Grand Prix race results (to be addressed in the frontend development).

---

## Backend Setup Instructions

To set up and run the backend for `F1TrackerApp`, follow these steps:

1. **Clone the Repository**:
   - The backend code is part of the main repository:
     ```bash
     git clone https://github.com/your-username/F1TrackerApp.git
     cd F1TrackerApp
     ```

2. **Set Up Python Environment**:
   - Navigate to the directory containing `function_app.py` (or create a separate backend directory if you prefer):
     ```bash
     python -m venv .venv
     source .venv/bin/activate  # On Windows: .venv\Scripts\activate
     ```
   - Install dependencies:
     ```bash
     pip install azure-functions requests azure-cosmos
     ```

3. **Configure Environment Variables**:
   - Create a `local.settings.json` file for local testing:
     ```json
     {
       "IsEncrypted": false,
       "Values": {
         "AzureWebJobsStorage": "",
         "FUNCTIONS_WORKER_RUNTIME": "python",
         "COSMOS_URI": "your-cosmos-uri",
         "COSMOS_KEY": "your-cosmos-key"
       }
     }
     ```
   - Replace `"your-cosmos-uri"` and `"your-cosmos-key"` with your actual Cosmos DB credentials.

4. **Test Locally**:
   - Run the Function App locally:
     ```bash
     func start
     ```
   - Test the `FetchRaceData` endpoint at `http://localhost:7071/api/FetchRaceData` to fetch and store data.
   - Test the `GetRaceData` endpoint at `http://localhost:7071/api/GetRaceData` to retrieve stored data.

5. **Deploy to Azure**:
   - Deploy the Function App to Azure:
     ```bash
     func azure functionapp publish f1-tracker-api
     ```
   - In the Azure portal, add the `COSMOS_URI` and `COSMOS_KEY` as application settings for the Function App.
   - Test the deployed endpoints:
     - `https://f1-tracker-api.azurewebsites.net/api/FetchRaceData`
     - `https://f1-tracker-api.azurewebsites.net/api/GetRaceData`

6. **Verify Data in Cosmos DB**:
   - Use the Cosmos DB Data Explorer to query the data:
     ```sql
     SELECT * FROM c
     ```
   - Ensure the data includes entries for the Australian Grand Prix and Chinese Grand Prix with correct `session_key` and `session_type` fields.

---

## Backend Challenges and Solutions

- **Duplicate Entries**:
  - **Challenge**: Initial runs of the Function App created duplicate entries in Cosmos DB, which caused issues like double-counting points in the frontend.
  - **Solution**: I identified and deleted duplicates using Cosmos DB queries and a cleanup script, then modified the function to use `upsert_item` to prevent future duplicates.
- **API Inconsistencies**:
  - **Challenge**: The external F1 API sometimes returned missing fields (e.g., `session_type`, `Fastest Lap`).
  - **Solution**: I added fallback values in `function_app.py` to ensure all required fields were present before storing data in Cosmos DB.
- **Environment Variables**:
  - **Challenge**: Hardcoding Cosmos DB credentials in the code was insecure.
  - **Solution**: I used environment variables (`os.environ`) to securely access credentials, configured both locally and in Azure.

---

## Future Backend Improvements


- **Data Validation**: Add more robust validation for API responses to handle malformed data.
- **Scheduled Data Fetching**: Modify the Function App to run on a timer trigger to automatically fetch race data after each race weekend.
- **Error Logging**: Integrate Azure Application Insights for better error tracking and monitoring.
- **Data Cleanup Automation**: Create a function to automatically clean up duplicates or outdated data in Cosmos DB.
- **API Rate Limiting**: Handle API rate limits by implementing retry logic in `function_app.py`.

---

## Contributing to the Backend

If you’d like to contribute to the backend of `F1TrackerApp`, please follow these steps:

1. **Fork the Repository**:
   - Fork the repository on GitHub to your own account:
     ```bash
     git clone https://github.com/your-username/F1TrackerApp.git
     cd F1TrackerApp
     ```

2. **Set Up the Development Environment**:
   - Follow the "Backend Setup Instructions" above to set up the Python environment and configure environment variables.
   - Ensure you have Azure Functions Core Tools installed:
     ```bash
     npm install -g azure-functions-core-tools@4 --unsafe-perm true
     ```

3. **Create a New Branch**:
   - Create a new branch for your changes:
     ```bash
     git checkout -b feature/your-feature-name
     ```

4. **Make Changes**:
   - Modify `function_app.py` or add new functions as needed.
   - Test your changes locally:
     ```bash
     func start
     ```
   - Use the local endpoints (`http://localhost:7071/api/FetchRaceData` and `http://localhost:7071/api/GetRaceData`) to verify your changes.
   - Query Cosmos DB to ensure data is being stored correctly:
     ```sql
     SELECT * FROM c
     ```

5. **Commit and Push Your Changes**:
   - Commit your changes with a descriptive message:
     ```bash
     git add .
     git commit -m "Add feature: your feature description"
     git push origin feature/your-feature-name
     ```

6. **Submit a Pull Request**:
   - Go to the GitHub repository page and create a pull request from your branch to the `main` branch.
   - Provide a detailed description of your changes, including any issues addressed or features added.
   - Reference any related issues (e.g., "Fixes #123").

7. **Code Review**:
   - I’ll review your pull request and provide feedback. Be prepared to make adjustments if necessary.
   - Focus on improving data handling, error management, or performance. If you encounter bugs or have feature ideas, open an issue on GitHub to discuss them.

---

Let’s create a detailed section for the frontend development of the F1TrackerApp to include in your README.md file. This section will focus on the step-by-step process of building the frontend using React Native and Expo, including setting up the project, creating the Race Results page (index.tsx), implementing the Driver Standings page (standings.tsx), adding navigation, handling sprint and race session separation, styling the UI, and debugging rendering issues (such as the Chinese Grand Prix race results not displaying). The content will be in Markdown format, ready for your GitHub repository.
markdown

## Frontend Development: Step-by-Step Guide

The frontend of `F1TrackerApp` is a mobile application built using React Native and Expo, designed to display Formula 1 race results, sprint race results, and driver standings for the 2025 season. It fetches data from the backend API (hosted on Azure Function App) and presents it through a user-friendly interface. Below is a detailed guide on how I built the frontend as of April 5, 2025.

### Step 1: Set Up the React Native Project with Expo
1. **Install Expo CLI**:
   - I installed the Expo CLI globally to create and manage the React Native project:
     ```bash
     npm install -g expo-cli
     ```

2. **Initialize the Project**:
   - I created a new Expo project with TypeScript for type safety:
     ```bash
     expo init F1TrackerApp --template expo-template-blank-typescript
     cd F1TrackerApp
     ```
   - This set up a basic React Native project with TypeScript configuration (`tsconfig.json`).

3. **Run the Project**:
   - I started the Expo development server:
     ```bash
     npx expo start --tunnel -c
     ```
   - The `--tunnel` flag allowed me to test on a physical device over the internet, and `-c` cleared the cache to ensure a fresh start.
   - I installed the Expo Go app on my mobile device, scanned the QR code from the Expo DevTools, and verified that the default app loaded successfully.

### Step 2: Create the Race Results Page (`index.tsx`)
1. **Set Up Basic Fetching and Display**:
   - I modified `app/index.tsx` to fetch race data from the backend API and display it in a list. Here’s the initial implementation:
     ```typescript
     import React, { useEffect, useState } from 'react';
     import { FlatList, StyleSheet, Text, View } from 'react-native';

     interface RaceResult {
       id: string;
       Standing: number;
       'Driver Number': string;
       Driver: string;
       Team: string;
       'Fastest Lap': string;
       Points: number;
       session_key: string;
       race_date: string;
       raceName: string;
       session_type?: string;
     }

     export default function RaceResults() {
       const [results, setResults] = useState<RaceResult[]>([]);
       const [loading, setLoading] = useState<boolean>(true);

       useEffect(() => {
         fetch('https://f1-tracker-api.azurewebsites.net/api/GetRaceData')
           .then(response => response.json())
           .then((data: RaceResult[]) => {
             setResults(data);
             setLoading(false);
           })
           .catch(error => {
             console.error('Error fetching race results:', error);
             setLoading(false);
           });
       }, []);

       if (loading) {
         return (
           <View style={styles.container}>
             <Text style={styles.loading}>Loading Race Results...</Text>
           </View>
         );
       }

       return (
         <View style={styles.container}>
           <Text style={styles.title}>F1 Race Results</Text>
           <FlatList
             data={results}
             keyExtractor={(item) => item.id}
             renderItem={({ item }) => (
               <View style={styles.item}>
                 <Text>{item.Standing}. {item.Driver}</Text>
                 <Text>Team: {item.Team}</Text>
                 <Text>Points: {item.Points}</Text>
               </View>
             )}
           />
         </View>
       );
     }

     const styles = StyleSheet.create({
       container: {
         flex: 1,
         padding: 20,
         backgroundColor: '#f5f5f5',
       },
       title: {
         fontSize: 24,
         fontWeight: 'bold',
         textAlign: 'center',
         marginBottom: 20,
       },
       loading: {
         fontSize: 18,
         textAlign: 'center',
         marginTop: 50,
         color: '#666',
       },
       item: {
         padding: 10,
         borderBottomWidth: 1,
         borderBottomColor: '#ddd',
       },
     });
     ```
   - I tested this setup to ensure the app fetched data from the backend API and displayed it in a basic list format.

2. **Add Race Selection Dropdown**:
   - I added a `Picker` component to allow users to select a race from a dropdown, filtering the displayed results accordingly:
     ```typescript
     import { Picker } from '@react-native-picker/picker';

     // Add state for race selection
     const [selectedRace, setSelectedRace] = useState<string>('');

     // Extract unique race names for the dropdown
     const raceNames = Array.from(new Set(results.map(result => result.raceName)));

     useEffect(() => {
       fetch('https://f1-tracker-api.azurewebsites.net/api/GetRaceData')
         .then(response => response.json())
         .then((data: RaceResult[]) => {
           const updatedData = data.map(item => ({
             ...item,
             session_type: (item.session_type || 'race').trim().toLowerCase(),
           }));
           setResults(updatedData);
           if (updatedData.length > 0) {
             setSelectedRace(updatedData[0].raceName);
           }
           setLoading(false);
         })
         .catch(error => {
           console.error('Error fetching race results:', error);
           setLoading(false);
         });
     }, []);

     // Filter results based on selected race
     const filteredResults = results.filter(result => result.raceName === selectedRace);

     // Add Picker to JSX
     <Picker
       selectedValue={selectedRace}
       onValueChange={(itemValue: string) => setSelectedRace(itemValue)}
       style={styles.picker}
     >
       {raceNames.map((raceName) => (
         <Picker.Item key={raceName} label={raceName} value={raceName} />
       ))}
     </Picker>
     ```
   - I updated the `FlatList` to use `filteredResults`:
     ```typescript
     <FlatList
       data={filteredResults}
       keyExtractor={(item) => item.id}
       renderItem={({ item }) => (
         <View style={styles.item}>
           <Text>{item.Standing}. {item.Driver}</Text>
           <Text>Team: {item.Team}</Text>
           <Text>Points: {item.Points}</Text>
         </View>
       )}
     />
     ```
   - I added styling for the `Picker`:
     ```typescript
     picker: {
       height: 50,
       width: '100%',
       marginBottom: 20,
       backgroundColor: '#fff',
       borderRadius: 8,
       borderWidth: 1,
       borderColor: '#ddd',
     },
     ```
   - I tested the dropdown to ensure races like the Australian Grand Prix and Chinese Grand Prix could be selected, filtering the results accordingly.

3. **Add Sprint and Race Separation**:
   - I grouped the filtered results by `session_type` to display sprint and race results separately for races that had both (e.g., Chinese Grand Prix):
     ```typescript
     const groupedResults = filteredResults.reduce((acc, result) => {
       const type = result.session_type === 'sprint' ? 'Sprint' : 'Race';
       if (!acc[type]) acc[type] = [];
       acc[type].push(result);
       return acc;
     }, {} as Record<string, RaceResult[]>);
     ```
   - I updated the rendering logic to display each session type separately:
     ```typescript
     // Add a date formatting function
     const formatDate = (dateString: string): string => {
       const date = new Date(dateString);
       return date.toLocaleDateString('en-US', {
         month: 'long',
         day: 'numeric',
         year: 'numeric',
       });
     };

     // Update the JSX to show race name, date, and grouped results
     {selectedRace && filteredResults.length > 0 && (
       <>
         <Text style={styles.raceTitle}>
           {filteredResults[0].raceName} - {formatDate(filteredResults[0].race_date)}
         </Text>
         {['Sprint', 'Race'].map((sessionType) => (
           groupedResults[sessionType]?.length > 0 && (
             <View key={sessionType} style={styles.sessionContainer}>
               <Text style={styles.sessionTypeTitle}>{sessionType} Results</Text>
               <FlatList
                 data={groupedResults[sessionType]}
                 keyExtractor={(item) => item.id}
                 renderItem={({ item }) => (
                   <View style={styles.item}>
                     <Text style={styles.position}>
                       {item.Standing}. {item.Driver} (#{item['Driver Number']})
                     </Text>
                     <Text>Team: {item.Team}</Text>
                     <Text>Fastest Lap: {item['Fastest Lap']}</Text>
                     <Text>Points: {item.Points}</Text>
                   </View>
                 )}
               />
               {sessionType === 'Sprint' && groupedResults['Race']?.length > 0 && (
                 <View style={styles.separator} />
               )}
             </View>
           )
         ))}
       </>
     )}
     ```
   - I added styles for the new elements:
     ```typescript
     raceTitle: {
       fontSize: 20,
       fontWeight: '600',
       color: '#e10600', // F1 red accent
       marginBottom: 10,
       textAlign: 'center',
     },
     sessionContainer: {
       marginBottom: 20,
     },
     sessionTypeTitle: {
       fontSize: 18,
       fontWeight: '500',
       color: '#333',
       marginTop: 10,
       marginBottom: 5,
     },
     position: {
       fontSize: 18,
       fontWeight: 'bold',
       color: '#333',
     },
     separator: {
       height: 1,
       backgroundColor: '#e10600',
       marginVertical: 15,
       width: '100%',
     },
     ```
   - I tested the separation logic with the Chinese Grand Prix, expecting to see both sprint and race results, but encountered an issue where only sprint results were displayed (to be debugged later).

### Step 3: Add Navigation with Expo Router
1. **Install Expo Router**:
   - I installed `expo-router` to handle navigation between pages:
     ```bash
     npm install expo-router
     ```
   - I updated the project structure to use Expo Router’s file-based navigation by ensuring my pages were in the `app` directory (`app/index.tsx` for the Race Results page).

2. **Add a Menu Button**:
   - I added a menu button to navigate to the sitemap, which Expo Router generates automatically:
     ```typescript
     import { useRouter } from 'expo-router';

     const router = useRouter();

     // Add to JSX at the top of the container in RaceResults
     <TouchableOpacity
       style={styles.menuButton}
       onPress={() => router.push('/_sitemap')}
     >
       <Text style={styles.menuIcon}>☰</Text>
     </TouchableOpacity>
     ```
   - I added styles for the menu button:
     ```typescript
     menuButton: {
       position: 'absolute',
       top: 10,
       left: 15,
       zIndex: 1,
     },
     menuIcon: {
       fontSize: 30,
       color: '#333',
     },
     ```
   - I updated the `container` style to account for the menu button:
     ```typescript
     container: {
       flex: 1,
       paddingTop: 50,
       paddingHorizontal: 15,
       backgroundColor: '#f5f5f5',
     },
     ```
   - I tested the navigation by clicking the menu button, ensuring the sitemap page (`_sitemap.tsx`) was displayed with links to other routes.

### Step 4: Create the Driver Standings Page (`standings.tsx`)
1. **Implement the Standings Page**:
   - I created a new file `app/standings.tsx` to display a leaderboard of driver standings, aggregating points across all races and avoiding double-counting duplicates:
     ```typescript
     import React, { useEffect, useState } from 'react';
     import { FlatList, StyleSheet, Text, View, TouchableOpacity } from 'react-native';
     import { useRouter } from 'expo-router';

     interface RaceResult {
       id: string;
       Standing: number;
       'Driver Number': string;
       Driver: string;
       Team: string;
       'Fastest Lap': string;
       Points: number;
       session_key: string;
       race_date: string;
       raceName: string;
       session_type?: string;
     }

     interface DriverStanding {
       Driver: string;
       Team: string;
       TotalPoints: number;
     }

     export default function DriverStandings() {
       const [standings, setStandings] = useState<DriverStanding[]>([]);
       const [loading, setLoading] = useState<boolean>(true);
       const router = useRouter();

       useEffect(() => {
         fetch('https://f1-tracker-api.azurewebsites.net/api/GetRaceData')
           .then(response => response.json())
           .then((data: RaceResult[]) => {
             const driverPoints = data.reduce((acc, result) => {
               const key = result.Driver;
               if (!acc[key]) {
                 acc[key] = { Driver: key, Team: result.Team, TotalPoints: 0, sessionKeys: new Set() };
               }
               if (!acc[key].sessionKeys.has(result.session_key)) {
                 acc[key].TotalPoints += result.Points;
                 acc[key].sessionKeys.add(result.session_key);
               }
               return acc;
             }, {} as Record<string, { Driver: string; Team: string; TotalPoints: number; sessionKeys: Set<string> }>);

             const sortedStandings = Object.values(driverPoints)
               .map(({ Driver, Team, TotalPoints }) => ({ Driver, Team, TotalPoints }))
               .sort((a, b) => b.TotalPoints - a.TotalPoints);
             setStandings(sortedStandings);
             setLoading(false);
           })
           .catch(error => {
             console.error('Error fetching standings:', error);
             setLoading(false);
           });
       }, []);

       if (loading) {
         return (
           <View style={styles.container}>
             <Text style={styles.loading}>Loading Driver Standings...</Text>
           </View>
         );
       }

       return (
         <View style={styles.container}>
           <TouchableOpacity
             style={styles.menuButton}
             onPress={() => router.push('/_sitemap')}
           >
             <Text style={styles.menuIcon}>☰</Text>
           </TouchableOpacity>
           <Text style={styles.title}>Driver Standings</Text>
           <FlatList
             data={standings}
             keyExtractor={(item) => item.Driver}
             renderItem={({ item, index }) => (
               <View style={styles.item}>
                 <Text style={styles.position}>
                   {index + 1}. {item.Driver}
                 </Text>
                 <Text>Team: {item.Team}</Text>
                 <Text>Points: {item.TotalPoints}</Text>
               </View>
             )}
           />
         </View>
       );
     }

     const styles = StyleSheet.create({
       container: {
         flex: 1,
         paddingTop: 50,
         paddingHorizontal: 15,
         backgroundColor: '#f5f5f5',
       },
       menuButton: {
         position: 'absolute',
         top: 10,
         left: 15,
         zIndex: 1,
       },
       menuIcon: {
         fontSize: 30,
         color: '#333',
       },
       title: {
         fontSize: 28,
         fontWeight: 'bold',
         textAlign: 'center',
         marginBottom: 20,
         color: '#333',
       },
       loading: {
         fontSize: 18,
         textAlign: 'center',
         marginTop: 50,
         color: '#666',
       },
       item: {
         backgroundColor: '#fff',
         padding: 15,
         marginBottom: 10,
         borderRadius: 8,
         borderWidth: 1,
         borderColor: '#ddd',
       },
       position: {
         fontSize: 18,
         fontWeight: 'bold',
         color: '#333',
       },
     });
     ```
   - I used a `Set` to track unique `session_key` values, preventing double-counting of points from duplicate entries (an issue that was resolved in the backend but still accounted for here).

2. **Add Navigation to Standings Page**:
   - Since I was using Expo Router, the `standings.tsx` file in the `app` directory automatically created a route at `/standings`.
   - I tested navigation by accessing the sitemap (`/_sitemap`) and clicking the link to `/standings`, ensuring the Driver Standings page loaded correctly.

### Step 5: Debug Rendering Issues
1. **Identify the Chinese Grand Prix Race Results Issue**:
   - I noticed that the Chinese Grand Prix race results (`session_key: "2025_2_Race"`, `session_type: "race"`) were not displaying in `index.tsx`, while the sprint results (`session_key: "2025_2_Sprint"`, `session_type: "sprint"`) were showing correctly.
   - I added debug logs to trace the data flow:
     ```typescript
     console.log('Step 1 - Fetched and updated data:', updatedData);
     console.log('Step 1 - Chinese GP data:', updatedData.filter(item => item.raceName === 'Chinese Grand Prix'));
     console.log('Step 2 - Filtered results for', selectedRace, ':', filteredResults);
     console.log('Step 3 - Grouped results:', groupedResults);
     console.log(`Step 4 - Rendering ${sessionType} results:`, groupedResults[sessionType]);
     ```
   - From the logs, I confirmed:
     - Step 1: The API response included 40 items for the Chinese Grand Prix (20 for `session_type: "race"`, 20 for `session_type: "sprint"`).
     - Step 2: `filteredResults` contained all 40 items when the Chinese Grand Prix was selected.
     - Step 3: `groupedResults` showed `Sprint` with 20 items, but `Race` was empty.
   - This indicated an issue in the grouping logic.

2. **Fix the Grouping Logic**:
   - I realized the `session_type` comparison was case-sensitive and not handling variations properly. I updated the grouping logic to be more robust:
     ```typescript
     const groupedResults = filteredResults.reduce((acc, result) => {
       const sessionType = result.session_type?.toLowerCase() === 'sprint' ? 'Sprint' : 'Race';
       if (!acc[sessionType]) acc[sessionType] = [];
       acc[sessionType].push(result);
       return acc;
     }, {} as Record<string, RaceResult[]>);
     ```
   - I retested, but the issue persisted, suggesting the `session_type` values in the data might not be as expected.

3. **Further Debugging**:
   - I inspected the data more closely in the `Step 1 - Chinese GP data` log and noticed that some `session_type` values might have been inconsistent (e.g., `"Race"` vs. `"race"`).
   - I updated the normalization in the `useEffect` hook to ensure consistency:
     ```typescript
     const updatedData = data.map(item => ({
       ...item,
       session_type: (item.session_type || 'race').trim().toLowerCase(),
     }));
     ```
   - I retested, but the race results still did not render, indicating a potential issue with the data in Cosmos DB or the API response.

4. **Cross-Check with Backend Data**:
   - I queried Cosmos DB to verify the data:
     ```sql
     SELECT * FROM c WHERE c.raceName = "Chinese Grand Prix"
     ```
     - Confirmed: 40 items (20 for `2025_2_Race`, 20 for `2025_2_Sprint`), with `session_type` values as `"race"` and `"sprint"`.
   - I tested the API endpoint directly (`https://f1-tracker-api.azurewebsites.net/api/GetRaceData`) and confirmed the data was returned correctly.
   - The issue likely lies in the rendering condition or a subtle bug in the grouping logic, which requires further investigation.

### Step 6: Finalize and Test the Frontend
1. **Test the App**:
   - I ran the app on Expo Go:
     ```bash
     npx expo start --tunnel -c
     ```
   - **Race Results Page**:
     - The Australian Grand Prix displayed race results correctly (no sprint race).
     - The Chinese Grand Prix displayed only sprint results, with the race results still missing (issue noted for future resolution).
   - **Driver Standings Page**:
     - The standings displayed correctly, with points aggregated accurately thanks to the `session_key` deduplication logic.

2. **Prepare for Deployment**:
   - I ensured all changes were committed to the Git repository (as part of the full project deployment):
     ```bash
     git add .
     git commit -m "Complete frontend implementation with Race Results and Driver Standings"
     ```
   - The full project, including the frontend, was pushed to GitHub (see the full README for deployment details).

---

## Frontend Setup Instructions

To set up and run the frontend of `F1TrackerApp`, follow these steps:

1. **Clone the Repository**:
   ```bash
   git clone https://github.com/your-username/F1TrackerApp.git
   cd F1TrackerApp

    Install Dependencies:
    bash

    npm install

    Ensure Backend is Running:
        Follow the backend setup instructions to deploy the Azure Function App and populate Cosmos DB with race data.
        Verify the API endpoint (https://f1-tracker-api.azurewebsites.net/api/GetRaceData) returns data.
    Run the App:
        Start the Expo development server:
        bash

        npx expo start --tunnel -c

        Scan the QR code with the Expo Go app on your mobile device to launch the app.
    Test the Features:
        On the Race Results page, select the Chinese Grand Prix and note that only sprint results are displayed (race results issue is pending resolution).
        Navigate to the Driver Standings page to verify the leaderboard.

Frontend Challenges and Solutions

    Chinese Grand Prix Race Results Not Displaying:
        Challenge: The race results for the Chinese Grand Prix (session_type: "race") were not rendering, despite the sprint results showing correctly.
        Solution Attempted: I added debug logs, normalized session_type values, and verified the data in Cosmos DB. The issue persists, likely due to a rendering condition or data inconsistency.
    Duplicate Points in Standings:
        Challenge: Initially, duplicate entries in Cosmos DB caused points to be double-counted in the Driver Standings page.
        Solution: I added logic to track unique session_key values in standings.tsx, and the backend was updated to prevent duplicates.
    Styling Consistency:
        Challenge: Ensuring a consistent F1-themed UI across different screen sizes.
        Solution: I used a light background (#f5f5f5), F1 red accents (#e10600), and rounded containers with borders for a clean look.

Future Frontend Improvements

    Resolve Rendering Issue: Fix the bug where the Chinese Grand Prix race results are not displaying by further debugging the data and rendering logic.
    Add Sprint Dates: Display the sprint race date separately if it differs from the race date.
    Constructor Standings: Add a new page to display team standings.
    Enhanced Styling: Incorporate team colors or driver images for a more immersive F1 experience.
    Error Handling: Display user-friendly error messages if the API fails to load data.
    Performance Optimization: Implement caching with AsyncStorage to reduce API calls.

Contributing to the Frontend
If you’d like to contribute to the frontend of F1TrackerApp, please follow these steps:

    Fork the Repository:
    bash

    git clone https://github.com/your-username/F1TrackerApp.git
    cd F1TrackerApp

    Set Up the Environment:
        Install dependencies:
        bash

        npm install

        Ensure the backend API is running and accessible.
    Create a New Branch:
    bash

    git checkout -b feature/your-feature-name

    Make Changes:
        Modify index.tsx or standings.tsx, or add new pages in the app directory.
        Test your changes:
        bash

        npx expo start --tunnel -c

    Commit and Push:
    bash

    git add .
    git commit -m "Add feature: your feature description"
    git push origin feature/your-feature-name

    Submit a Pull Request:
        Create a pull request on GitHub with a detailed description of your changes.


---

### Notes for You
- **Focus on Frontend**: This section details the frontend development process, including setting up the project, building the Race Results and Driver Standings pages, adding navigation, handling sprint and race separation, styling, and debugging the Chinese Grand Prix race results issue.
- **Customization**:
  - Replace `your-username` with your actual GitHub username in the `Frontend Setup Instructions` and `Contributing to the Frontend` sections.
  - The API endpoint (`https://f1-tracker-api.azurewebsites.net/api/GetRaceData`) should match your deployed Function App URL.
- **Rendering Issue**: I’ve documented the ongoing issue with the Chinese Grand Prix race results not displaying, including the debugging steps taken. If you have additional logs or data insights, we can resolve this and update the README.
- **Integration**: You can append this section to your existing backend README to create a complete project README, or use it as a standalone frontend documentation section.
- **Contributing Section**: The contributing guidelines are tailored for the frontend but left open-ended since the full README would include a license and contact section (which can be added if needed).

Let me know if you’d like to expand on any part, resolve the rendering issue, or combine this with the backend README to create the full project documentation!

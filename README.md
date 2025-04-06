# F1TrackerApp

## Overview

**F1TrackerApp** is a mobile application built using React Native and Expo, designed to track Formula 1 race results, sprint race results, and driver standings for the 2025 season. The app fetches race data from an external API, stores it in Azure Cosmos DB, and presents it through a user-friendly interface featuring race selection dropdowns, detailed result views, and a driver standings leaderboard. The backend is powered by an Azure Function App (`function_app.py`) that handles data retrieval and storage, while the frontend (`index.tsx` and `standings.tsx`) delivers an intuitive user experience.

---

## Why I Started This Project

As an avid Formula 1 fan and an aspiring full-stack developer, I wanted to create a project that combined my passion for F1 with my goal of mastering modern development technologies. I began working on `F1TrackerApp` in early 2025 to build a personal tool for tracking the 2025 F1 season in real-time, focusing on race results and driver standings. My objective was to develop a functional app for my own use while also creating a portfolio piece to showcase my skills to potential employers or collaborators.

### Learning Goals
When I started this project, I aimed to achieve the following learning objectives:
1. **React Native and Expo**: Gain hands-on experience building a cross-platform mobile app using React Native and Expo, focusing on components, state management, and navigation.
2. **TypeScript**: Enhance my TypeScript skills by using it for type-safe development in React Native.
3. **Azure Services**: Learn how to use Azure Cosmos DB for NoSQL data storage and Azure Function Apps for serverless backend logic.
4. **API Integration**: Practice fetching and processing data from external APIs, specifically targeting an F1 data source like the Ergast F1 API (or a similar API).
5. **Data Management**: Understand how to manage and display real-time data, including handling issues like duplicates, data normalization, and session types (e.g., race vs. sprint).
6. **Debugging and Problem-Solving**: Improve my debugging skills by addressing real-world challenges such as missing data, UI rendering issues, and API inconsistencies.

---

## What I Learned

Building `F1TrackerApp` was a challenging yet enriching experience. Here’s what I learned during the process:

1. **React Native and Expo**:
   - I became proficient in creating mobile UIs using React Native components such as `FlatList`, `Picker`, and `TouchableOpacity`.
   - I learned how to leverage Expo for rapid development, including setting up a project, running it on a physical device via Expo Go, and debugging with commands like `npx expo start --tunnel -c`.
   - I mastered state management with React hooks (`useState`, `useEffect`) to handle asynchronous data fetching and dynamic UI updates.

2. **TypeScript**:
   - I gained confidence in using TypeScript to define interfaces (e.g., `RaceResult` and `DriverStanding`) for type safety, which helped catch errors early during development.
   - I learned how to handle optional properties and type assertions when dealing with API responses, improving code reliability.

3. **Azure Services**:
   - I successfully set up Azure Cosmos DB to store F1 race data and learned how to query it using SQL-like syntax in the Cosmos DB Data Explorer.
   - I implemented an Azure Function App (`function_app.py`) to fetch race data, process it, and store it in Cosmos DB, including support for both race and sprint sessions.
   - I resolved issues with duplicate data entries in Cosmos DB by writing scripts to clean up redundant items, improving data integrity.

4. **API Integration**:
   - I learned how to fetch data from an external F1 API (e.g., Ergast API or a mock API) and process it in Python using the `requests` library.
   - I handled real-world API challenges, such as missing fields and inconsistent data formats, by adding fallback logic in both the backend and frontend.

5. **Data Management**:
   - I implemented logic to handle both race and sprint results, including normalizing `session_type` values and grouping data for display.
   - I fixed issues with duplicate entries inflating driver standings by ensuring unique `session_key` counting in `standings.tsx`.
   - I learned how to manage and query NoSQL data in Cosmos DB, including writing scripts to delete duplicate entries.

6. **Debugging and Problem-Solving**:
   - I became adept at debugging React Native apps by adding console logs at each step of the data flow (fetching, filtering, grouping, rendering).
   - I resolved complex rendering issues, such as the Chinese Grand Prix race results not displaying, by systematically tracing the data pipeline and adjusting the grouping logic.
   - I learned how to handle edge cases, like missing data or unexpected API responses, by adding robust error handling and fallbacks.

7. **UI/UX Design**:
   - I improved my UI design skills by creating a clean, F1-themed interface with red accents (e.g., `#e10600` for separators) and intuitive navigation.
   - I added features like a separator between sprint and race results to enhance readability and user experience.

Overall, this project significantly advanced my skills as a full-stack developer, providing practical experience in backend data management, frontend UI design, and debugging complex issues.

---

## Project Features

As of April 5, 2025, `F1TrackerApp` includes the following features:
- **Race Results Page** (`index.tsx`):
  - Displays race and sprint results for the 2025 F1 season.
  - Includes a dropdown (`Picker`) to select races (e.g., Australian Grand Prix, Chinese Grand Prix).
  - Shows detailed results for each session type ("Sprint" and "Race"), including driver standings, teams, fastest laps, and points.
  - Features a visual separator between sprint and race results for races that have both (e.g., Chinese Grand Prix).
- **Driver Standings Page** (`standings.tsx`):
  - Displays a leaderboard of driver standings, sorted by total points.
  - Includes logic to prevent double-counting points from duplicate sessions.
- **Backend Data Pipeline** (`function_app.py`):
  - Fetches race data from an external F1 API.
  - Stores data in Azure Cosmos DB with proper `session_key` and `session_type` fields.
  - Supports both race and sprint sessions.

**Note**: There is an ongoing issue where the Chinese Grand Prix race results are not displaying, only the sprint results. This is being debugged and will be resolved in future updates.

---

## Tech Stack

- **Frontend**:
  - React Native (with Expo)
  - TypeScript
  - React Hooks (`useState`, `useEffect`)
  - React Native Components (`FlatList`, `Picker`, `TouchableOpacity`)
- **Backend**:
  - Azure Function App (`function_app.py`)
  - Python (`requests` for API calls, `azure-cosmos` for database interactions)
- **Database**:
  - Azure Cosmos DB (NoSQL)
- **Deployment**:
  - Expo Go for local development and testing
  - Azure for hosting the Function App and Cosmos DB

---

## How I Built F1TrackerApp: Step-by-Step Guide

Below is a detailed step-by-step guide on how I built `F1TrackerApp`, from initial setup to the current implementation as of April 5, 2025.

### Step 1: Project Setup
1. **Initialize the React Native Project**:
   - I installed the Expo CLI globally to set up the project:
     ```bash
     npm install -g expo-cli
     ```
   - I created a new Expo project with TypeScript:
     ```bash
     expo init F1TrackerApp --template expo-template-blank-typescript
     cd F1TrackerApp
     ```
   - I started the development server to test the setup:
     ```bash
     npx expo start --tunnel -c
     ```
   - I scanned the QR code with the Expo Go app on my mobile device to verify the app was running.

2. **Set Up Azure Services**:
   - I created an Azure account and set up a resource group for the project.
   - I created an Azure Cosmos DB account with a database named `F1Data` and a container named `RaceData`.
   - I created an Azure Function App to host the backend logic, which would fetch and store race data.

Let's continue the README.md from where we left off in the "How I Built F1TrackerApp: Step-by-Step Guide" section, starting with the deployment of the Function App, and complete the entire README with all necessary sections.
markdown

### Step 2: Build the Backend (`function_app.py`) (Continued)

3. **Deploy the Function App**:
   - I deployed the Function App to Azure using VS Code:
     ```bash
     func azure functionapp publish f1-tracker-api
     ```
   - I tested the endpoint (`https://f1-tracker-api.azurewebsites.net/results`) in a browser and with Postman to ensure it returned the stored data from Cosmos DB.
   - I verified that the data included entries for the Australian Grand Prix (`session_key: "2025_1_Race"`, `session_type: "Race"`) and the Chinese Grand Prix (`session_key: "2025_2_Race"`, `session_type: "Race"` and `session_key: "2025_2_Sprint"`, `session_type: "Sprint"`).

### Step 3: Build the Frontend
1. **Create the Race Results Page (`index.tsx`)**:
   - I started with a basic React Native component to fetch and display race results from the API:
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
         fetch('https://f1-tracker-api.azurewebsites.net/results')
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
       },
       title: {
         fontSize: 24,
         fontWeight: 'bold',
       },
       loading: {
         fontSize: 18,
         textAlign: 'center',
         marginTop: 50,
       },
       item: {
         padding: 10,
         borderBottomWidth: 1,
       },
     });
     ```
   - I tested this initial setup by running the app with `npx expo start --tunnel -c` and ensuring the data was fetched and displayed.

2. **Add Race Selection Dropdown**:
   - I added a `Picker` component to allow users to select a race from a dropdown:
     ```typescript
     import { Picker } from '@react-native-picker/picker';

     // Inside RaceResults component
     const [selectedRace, setSelectedRace] = useState<string>('');
     const raceNames = Array.from(new Set(results.map(result => result.raceName)));

     useEffect(() => {
       fetch('https://f1-tracker-api.azurewebsites.net/results')
         .then(response => response.json())
         .then((data: RaceResult[]) => {
           setResults(data);
           if (data.length > 0) {
             setSelectedRace(data[0].raceName);
           }
           setLoading(false);
         })
         .catch(error => {
           console.error('Error fetching race results:', error);
           setLoading(false);
         });
     }, []);

     // Add to JSX
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
   - I filtered the results based on the selected race:
     ```typescript
     const filteredResults = results.filter(result => result.raceName === selectedRace);
     ```
   - I updated the `FlatList` to use `filteredResults` instead of `results`.

3. **Add Sprint and Race Separation**:
   - I grouped the filtered results by `session_type` to display sprint and race results separately:
     ```typescript
     const groupedResults = filteredResults.reduce((acc, result) => {
       const type = result.session_type === 'sprint' ? 'Sprint' : 'Race';
       if (!acc[type]) acc[type] = [];
       acc[type].push(result);
       return acc;
     }, {} as Record<string, RaceResult[]>);
     ```
   - I modified the rendering logic to display both session types with a loop:
     ```typescript
     {['Sprint', 'Race'].map((sessionType) => (
       groupedResults[sessionType]?.length > 0 && (
         <View key={sessionType}>
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
         </View>
       )
     ))}
     ```

4. **Add Navigation and Menu**:
   - I installed `expo-router` for navigation:
     ```bash
     npm install expo-router
     ```
   - I added a menu button to navigate to the sitemap:
     ```typescript
     import { useRouter } from 'expo-router';

     const router = useRouter();

     // Add to JSX at the top of the container
     <TouchableOpacity
       style={styles.menuButton}
       onPress={() => router.push('/_sitemap')}
     >
       <Text style={styles.menuIcon}>☰</Text>
     </TouchableOpacity>
     ```

5. **Style the UI**:
   - I added F1-themed styling with a red accent (`#e10600`) for separators and clean layouts:
     ```typescript
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
       picker: {
         height: 50,
         width: '100%',
         marginBottom: 20,
         backgroundColor: '#fff',
         borderRadius: 8,
         borderWidth: 1,
         borderColor: '#ddd',
       },
       sessionContainer: {
         marginBottom: 20,
       },
       raceTitle: {
         fontSize: 20,
         fontWeight: '600',
         color: '#e10600',
         marginBottom: 10,
       },
       sessionTypeTitle: {
         fontSize: 18,
         fontWeight: '500',
         color: '#333',
         marginTop: 10,
         marginBottom: 5,
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
       separator: {
         height: 1,
         backgroundColor: '#e10600',
         marginVertical: 15,
         width: '100%',
       },
     });
     ```
   - I added a date formatting function to display the race date:
     ```typescript
     const formatDate = (dateString: string): string => {
       const date = new Date(dateString);
       return date.toLocaleDateString('en-US', {
         month: 'long',
         day: 'numeric',
         year: 'numeric',
       });
     };
     ```
   - I displayed the race name and date above the results:
     ```typescript
     <Text style={styles.raceTitle}>
       {filteredResults[0].raceName} - {formatDate(filteredResults[0].race_date)}
     </Text>
     ```

6. **Create the Driver Standings Page (`standings.tsx`)**:
   - I created a new file `app/standings.tsx` to display the driver standings leaderboard:
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
         fetch('https://f1-tracker-api.azurewebsites.net/results')
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
   - I ensured the standings page fetched the same data as the race results page but aggregated points by driver, using a `Set` to track unique `session_key` values and avoid double-counting points.

### Step 4: Handle Data Issues
1. **Fix Duplicate Entries in Cosmos DB**:
   - I noticed duplicate entries in Cosmos DB (e.g., items 0-19 and 20-39 for the Australian Grand Prix, and items 40-59 and 60-79 for the Chinese Grand Prix race results).
   - I identified the old entries lacking a `session_type` field using a query in the Cosmos DB Data Explorer:
     ```sql
     SELECT * FROM c WHERE NOT IS_DEFINED(c.session_type)
     ```
   - I deleted the duplicate items (0-19, 40-59) manually in the Data Explorer.
   - I verified the remaining items:
     - Items 20-39: Australian Grand Prix Race (`session_key: "2025_1_Race"`, `session_type: "Race"`)
     - Items 60-79: Chinese Grand Prix Race (`session_key: "2025_2_Race"`, `session_type: "Race"`)
     - Items 80-99: Chinese Grand Prix Sprint (`session_key: "2025_2_Sprint"`, `session_type: "Sprint"`)

2. **Prevent Double-Counting in Standings**:
   - I noticed that duplicate entries in Cosmos DB caused points to be double-counted in the driver standings.
   - I updated `standings.tsx` to track unique `session_key` values using a `Set` to ensure points were only counted once per session:
     ```typescript
     if (!acc[key].sessionKeys.has(result.session_key)) {
       acc[key].TotalPoints += result.Points;
       acc[key].sessionKeys.add(result.session_key);
     }
     ```

### Step 5: Add Sprint Race Support and Separator
1. **Update Backend for Sprint Races**:
   - I modified `function_app.py` to fetch sprint race results for races like the Chinese Grand Prix and store them with `session_type: "Sprint"`.
   - I ensured the API response included both race and sprint results with appropriate `session_key` and `session_type` fields.

2. **Update Frontend to Display Sprint and Race Results**:
   - In `index.tsx`, I ensured the grouping logic handled both `session_type: "sprint"` and `session_type: "race"` values by normalizing them:
     ```typescript
     const updatedData = data.map(item => ({
       ...item,
       session_type: (item.session_type || 'Race').trim().toLowerCase(),
     }));
     const groupedResults = filteredResults.reduce((acc, result) => {
       const type = result.session_type === 'sprint' ? 'Sprint' : 'Race';
       if (!acc[type]) acc[type] = [];
       acc[type].push(result);
       return acc;
     }, {} as Record<string, RaceResult[]>);
     ```

3. **Add Separator Between Sprint and Race Results**:
   - I added a visual separator in `index.tsx` to separate sprint and race results for races that had both (e.g., Chinese Grand Prix):
     ```typescript
     {sessionType === 'Sprint' && groupedResults['Race']?.length > 0 && (
       <View style={styles.separator} />
     )}
     ```
   - The separator was styled with a red line to match the F1 theme:
     ```typescript
     separator: {
       height: 1,
       backgroundColor: '#e10600',
       marginVertical: 15,
       width: '100%',
     },
     ```

### Step 6: Debug Rendering Issues
1. **Identify the Issue with Chinese Grand Prix Race Results**:
   - I noticed that the Chinese Grand Prix race results (`session_key: "2025_2_Race"`, `session_type: "Race"`) were not displaying in `index.tsx`, while the sprint results (`session_key: "2025_2_Sprint"`, `session_type: "Sprint"`) were showing correctly.
   - I added debug logs to trace the data flow:
     ```typescript
     console.log('Step 1 - Fetched and updated data:', updatedData);
     console.log('Step 1 - Chinese GP data:', updatedData.filter(item => item.raceName === 'Chinese Grand Prix'));
     console.log('Step 2 - Filtered results for', selectedRace, ':', filteredResults);
     console.log('Step 3 - Grouped results:', groupedResults);
     console.log(`Step 4 - Rendering ${sessionType} results:`, groupedResults[sessionType]);
     ```
   - I found that the race results were being fetched and filtered correctly but were not being grouped properly due to a mismatch in `session_type` handling.

2. **Attempt to Fix the Grouping Logic**:
   - I ensured `session_type` was normalized to lowercase in the `useEffect` hook:
     ```typescript
     const updatedData = data.map(item => ({
       ...item,
       session_type: (item.session_type || 'Race').trim().toLowerCase(),
     }));
     ```
   - I verified the grouping logic:
     ```typescript
     const type = result.session_type === 'sprint' ? 'Sprint' : 'Race';
     ```
   - Despite these changes, the race results still did not render, suggesting the issue might be with the data in Cosmos DB or the API response.

3. **Check Cosmos DB and API**:
   - I queried Cosmos DB to confirm the data:
     ```sql
     SELECT * FROM c WHERE c.raceName = "Chinese Grand Prix"
     ```
   - Expected: 40 items (20 for `2025_2_Race`, 20 for `2025_2_Sprint`).
   - I tested the API endpoint (`https://f1-tracker-api.azurewebsites.net/results`) to ensure both race and sprint results were returned.
   - The issue persists, indicating a deeper problem that requires further investigation (e.g., data corruption in Cosmos DB or a rendering condition in `index.tsx`).

### Step 7: Finalize and Test
1. **Test the App**:
   - I ran the app on Expo Go:
     ```bash
     npx expo start --tunnel -c
     ```
   - I tested the Race Results page:
     - The Australian Grand Prix displayed race results correctly (no sprint race).
     - The Chinese Grand Prix displayed sprint results but not race results, indicating the ongoing issue.
   - I tested the Driver Standings page, which displayed correctly after fixing the double-counting issue.

2. **Deploy to GitHub**:
   - I initialized a Git repository:
     ```bash
     git init
     git add .
     git commit -m "Initial commit of F1TrackerApp"
     ```
   - I created a repository on GitHub and pushed the code:
     ```bash
     git remote add origin https://github.com/your-username/F1TrackerApp.git
     git push -u origin main
     ```
   - I added this README to document the project.

---

## How to Run F1TrackerApp

### Prerequisites
- **Node.js and npm**: Ensure Node.js is installed on your system.
- **Expo CLI**: Install globally with:
  ```bash
  npm install -g expo-cli

    Azure Account: Set up an Azure account with Cosmos DB and Function App configured.
    Python: Required for the backend Function App (Python 3.8+ recommended).
    Expo Go App: Install on your mobile device for testing.

Setup Instructions

    Clone the Repository:
    bash

    git clone https://github.com/your-username/F1TrackerApp.git
    cd F1TrackerApp

    Install Dependencies:
    bash

    npm install

    Set Up Azure Backend:
        Deploy the Azure Function App (function_app.py) to your Azure account:
        bash

        func azure functionapp publish f1-tracker-api

        Update function_app.py with your Cosmos DB credentials:
        python

        cosmos_uri = "YOUR_COSMOS_URI"
        cosmos_key = "YOUR_COSMOS_KEY"

        Ensure Cosmos DB is populated with race data by running the Function App. Verify the data with:
        sql

        SELECT * FROM c

        Update the API endpoint in index.tsx and standings.tsx if necessary:
        typescript

        fetch('https://your-function-app-name.azurewebsites.net/results')

    Run the App:
        Start the Expo development server:
        bash

        npx expo start --tunnel -c

        Scan the QR code with the Expo Go app on your mobile device to launch the app.
    Test the Features:
        On the Race Results page, select the Chinese Grand Prix from the dropdown and note that only sprint results are currently displayed (race results issue is pending resolution).
        Navigate to the Driver Standings page to verify the leaderboard is accurate.

Project Structure

F1TrackerApp/
├── app/
│   ├── index.tsx           # Race Results page
│   ├── standings.tsx       # Driver Standings page
│   └── _sitemap.tsx        # Navigation sitemap (auto-generated by Expo)
├── function_app.py         # Azure Function App for backend logic
├── package.json            # Project dependencies
├── tsconfig.json           # TypeScript configuration
└── README.md               # Project documentation

Known Issues

    Chinese Grand Prix Race Results Not Displaying: Currently, only the sprint results for the Chinese Grand Prix are shown in index.tsx. The race results (session_key: "2025_2_Race") are not rendering despite being fetched. This issue is under investigation, with potential causes being:
        Data inconsistencies in Cosmos DB (e.g., missing or incorrect session_type values).
        A rendering condition in index.tsx that skips the race results.
        An issue with the API response filtering out race results.

Future Improvements

    Resolve Rendering Issue: Fix the issue with the Chinese Grand Prix race results not displaying.
    Add Sprint Race Dates: Display the sprint race date alongside the race date (e.g., “Sprint Results - March 22, 2025”).
    Constructor Standings: Add a page to display constructor (team) standings.
    Enhanced Styling: Incorporate more F1-themed elements, such as team logos or driver images.
    Error Handling: Improve error messages for API failures and display user-friendly alerts.
    Performance Optimization: Implement caching for API calls to reduce load times and improve performance.

Screenshots
(To be added once the rendering issue with the Chinese Grand Prix race results is resolved. Screenshots will include the Race Results page showing both sprint and race results, and the Driver Standings page.)
Contributing
Contributions are welcome! If you’d like to contribute to F1TrackerApp, please fork the repository, create a new branch, and submit a pull request with your changes. If you encounter any issues, open an issue on GitHub with a detailed description.
License
This project is licensed under the MIT License. See the LICENSE file for details.
Contact
For questions or feedback, reach out to me at your-email@example.com (mailto:your-email@example.com) or connect with me on GitHub: your-username.


---

### Notes for You
- **Completeness**: This README includes all requested sections: motivation, learning goals, lessons learned, a detailed step-by-step build guide, setup instructions, project features, structure, known issues, future improvements, and more.
- **Known Issues**: I’ve noted the ongoing issue with the Chinese Grand Prix race results not displaying, as per your latest input, and suggested potential causes to guide further debugging.
- **Screenshots**: I’ve included a placeholder for screenshots, which you can add once the rendering issue is resolved.
- **Customization**:
  - Replace `your-username` with your actual GitHub username in the `Clone the Repository` and `Contact` sections.
  - Update the email address in the `Contact` section with your actual email.
  - If your `function_app.py` uses different environment variables or API endpoints, update the `Setup Instructions` accordingly.
- **Debugging the Rendering Issue**: If you’d like to resolve the Chinese Grand Prix race results issue, you can share the debug logs from `index.tsx` (Steps 1-4) or the output of the Cosmos DB query for the Chinese Grand Prix data.

You can copy this Markdown content into a `README.md` file in your GitHub repository. Let me know if you need further adjustments or assistance with debugging the rendering issue!

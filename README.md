# 📦 Importing Data to Firestore Database

> A hands-on guide and lab documentation for importing structured data into a Firestore database using Google Cloud Shell and Node.js.

![Node](https://img.shields.io/badge/node.js-18.x-green.svg)
![Firestore](https://img.shields.io/badge/database-Firestore-orange.svg)
![Google Cloud](https://img.shields.io/badge/cloud-GCP-blue.svg)

---
## 🔧 Tech Stack Used

<p align="left">
  <img src="https://img.shields.io/badge/Cloud-Google_Cloud_Platform-blue?logo=googlecloud&logoColor=white" />
  <img src="https://img.shields.io/badge/Database-Firestore-orange?logo=firebase" />
  <img src="https://img.shields.io/badge/Runtime-Node.js-green?logo=node.js" />
  <img src="https://img.shields.io/badge/CLI-gcloud-informational?logo=googlecloud" />
  <img src="https://img.shields.io/badge/Logging-Google_Cloud_Logging-blueviolet?logo=googlecloud" />
</p>

**Cloud Platform**
- Google Cloud Platform (GCP)
  - Firestore (NoSQL database)
  - Cloud Shell
  - Cloud Logging
  - `gcloud` CLI

**Backend & Data Handling**
- Node.js (server-side scripting)
- npm (package management)

**Key npm Packages**
- `@google-cloud/firestore`
- `@google-cloud/logging`
- `csv-parse`
- `faker`

**Data Format**
- CSV (Comma-Separated Values)

**Command Line Tools**
- Git
- gcloud CLI

---

## 🧭 Table of Contents

- [Overview](#overview)  
- [Prerequisites](#prerequisites)  
- [Step-by-Step Guide](#step-by-step-guide)  
  1. [Activate Cloud Shell](#1-activate-cloud-shell)  
  2. [Authenticate & Configure Project](#2-authenticate--configure-project)  
  3. [Set Up Firestore](#3-set-up-firestore)  
  4. [Clone the Lab Repository](#4-clone-the-lab-repository)  
  5. [Install Dependencies](#5-install-dependencies)  
  6. [Import CSV Data to Firestore](#6-import-csv-data-to-firestore)  
  7. [Generate Test Data](#7-generate-test-data)  
  8. [Import Generated Data](#8-import-generated-data)  
- [Conclusion](#conclusion)

---

## 🧾 Overview

Google Cloud Platform (GCP) provides tools for managing databases, deploying AI/ML pipelines, and building full-scale applications. Firestore is one of GCP’s real-time NoSQL document databases optimised for scalability and performance.

In this lab, we will:

- Set up a Firestore instance.
- Write scripts to import CSV data into Firestore.
- Generate synthetic customer data.
- Enable logging using Google Cloud Logging.

---

## 🛠️ Prerequisites

- Google Cloud account (or Qwiklabs temporary project).
- Basic knowledge of Node.js and `npm`.
- Familiarity with Cloud Shell.

---

## 🚀 Step-by-Step Guide

### 1. Activate Cloud Shell

- Open Google Cloud Console.
- Click **Activate Cloud Shell** (top-right).
- Pre-installed tools include: `gcloud`, `git`, and `npm`.

---

### 2. Authenticate & Configure Project

```bash
gcloud auth list
gcloud config list project
```
Output:

```bash
[core]
project = <project_ID>
```

---

### 3. Set Up Firestore

1. Go to Navigation Menu > Firestore.
2. Click Select Native Mode.
3. Choose a region and click Create Database.

📌 Certainly:

Both modes offer high performance with robust consistency, although they exhibit distinct appearances and are tailored for specific usage scenarios.
Native Mode excels in enabling numerous users to have concurrent access to shared data. It boasts real-time updates and a direct channel connecting your database to web or mobile clients.
On the other hand, Datastore Mode prioritises exceptional throughput, catering to intensive reading and writing operations.

---

### 4. Clone Repository

```bash
git clone https://github.com/rosera/product-name
cd product-name/lab01
```
Inspect the package.json file for dependencies and scripts.

---

### 5. Install Dependencies

Install required libraries:
```bash
npm install @google-cloud/firestore @google-cloud/logging csv-parse
```

Your package.json should now include:
```json
"dependencies": {
  "@google-cloud/firestore": "^6.4.1",
  "@google-cloud/logging": "^10.3.1",
  "csv-parse": "^4.4.5"
}
```

---

### 6. Import CSV Data to Firestore

Update importTestData.js:

🔹 Add Required Modules
```javascript
const { promisify } = require("util");
const parse = promisify(require("csv-parse"));
const { readFile } = require("fs").promises;
const { Firestore } = require("@google-cloud/firestore");
const { Logging } = require("@google-cloud/logging");
```

🔹 Initialise Firestore and Logging
```javascript
const db = new Firestore();

const logName = "pet-theory-logs-importTestData";
const logging = new Logging();
const log = logging.log(logName);
const resource = { type: "global" };
```

🔹 Firestore Batch Import Function
```javascript
function writeToFirestore(records) {
  const batchCommits = [];
  let batch = db.batch();
  records.forEach((record, i) => {
    const docRef = db.collection("customers").doc(record.email);
    batch.set(docRef, record);
    if ((i + 1) % 500 === 0) {
      console.log(`Writing record ${i + 1}`);
      batchCommits.push(batch.commit());
      batch = db.batch();
    }
  });
  batchCommits.push(batch.commit());
  return Promise.all(batchCommits);
}
```

🔹 Main Import Function with Logging
```javascript
async function importCsv(csvFileName) {
  const fileContents = await readFile(csvFileName, "utf8");
  const records = await parse(fileContents, { columns: true });
  try {
    await writeToFirestore(records);
  } catch (e) {
    console.error(e);
    process.exit(1);
  }
  console.log(`Wrote ${records.length} records`);

  const success_message = `Success: importTestData - Wrote ${records.length} records`;
  const entry = log.entry({ resource }, { message: success_message });
  log.write([entry]);
}

importCsv(process.argv[2]).catch(console.error);
```

---

### 7. Generate Test Data

Install Faker:
```bash
npm install faker@5.5.3
```

Edit createTestData.js:

🔹 Add Logging Setup
```javascript
const { Logging } = require("@google-cloud/logging");
const logName = "pet-theory-logs-createTestData";
const logging = new Logging();
const log = logging.log(logName);
const resource = { type: "global" };
```

🔹 Create Customer Data
```javascript
async function createTestData(recordCount) {
  const fileName = `customers_${recordCount}.csv`;
  const f = fs.createWriteStream(fileName);
  f.write("id,name,email,phone\n");
  for (let i = 0; i < recordCount; i++) {
    const id = faker.datatype.number();
    const firstName = faker.name.firstName();
    const lastName = faker.name.lastName();
    const name = `${firstName} ${lastName}`;
    const email = faker.internet.email(firstName, lastName).toLowerCase();
    const phone = faker.phone.phoneNumber();
    f.write(`${id},${name},${email},${phone}\n`);
  }
  console.log(`Created file ${fileName} containing ${recordCount} records.`);

  const success_message = `Success: createTestData - Created file ${fileName} containing ${recordCount} records.`;
  const entry = log.entry({ resource }, { name: fileName, recordCount, message: success_message });
  log.write([entry]);
}
```

Run:
```bash
node createTestData.js 1000
```

---

### 8. Import Generated Data

Run the import:
```bash
node importTestData.js customers_1000.csv
```

If you see a CSV-parse error, reinstall it:
```bash
npm install csv-parse
```

Create and import a larger dataset (optional):
```bash
node createTestData.js 20000
node importTestData.js customers_20000.csv
```

---

### ✅ Conclusion

You’ve successfully:
  - Created test data using Faker
  - Imported data into Firestore using a batch write method
  - Logged operations using Google Cloud Logging

This hands-on lab provides a complete mini-pipeline for structured data ingestion into Firestore using GCP-native tools and Node.js.

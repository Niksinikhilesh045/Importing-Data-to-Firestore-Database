# 📦 Importing Data to Firestore Database

> A hands-on guide and lab documentation for importing structured data into a Firestore database using Google Cloud Shell and Node.js.

![License](https://img.shields.io/badge/license-MIT-blue.svg)
![Node](https://img.shields.io/badge/node.js-18.x-green.svg)
![Firestore](https://img.shields.io/badge/database-Firestore-orange.svg)
![Google Cloud](https://img.shields.io/badge/cloud-GCP-blue.svg)

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

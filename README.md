# Importing Data to Firestore Database
 Description and lab work on importing data to Firestore Database

Google Cloud Platform gives you a public cloud vendor that offers a suite of computing services to do everything from data management to delivering web and video over the web to AI and machine learning tools.

Creating a GitHub repository with content for importing data into a Firestore database involves several steps, including setting up the Firestore project, creating the data, and writing code to import that data. Below, I'll provide you with a general outline of the process and some example code snippets you can use to get started.

Cloud shell gives a wide area of development tools to create a database, retrieve and modify projects in the Goggle cloud Shell

# Here is importing data through projects and pre-installed files and data as part of GCP Skills boost

Step 1. Click Activate Cloud Shell Activate Cloud Shell icon at the top of the Google Cloud console.

gcloud is the command-line tool for Google Cloud. It comes pre-installed on Cloud Shell and supports tab-completion.

Step 2. You can list the active account name with this command:

    gcloud auth list

And list projects through configuration list

    gcloud config list project

# Output:

[core]
project = <project_ID>
Example output:

[core]
project = qwiklabs-gcp-44776a13dea667a6

step 3. Setting up Firestore in Google cloud 

    1.In the Cloud Console, go to the Navigation menu and select Firestore.

    2.Click the Select Native Mode button.

Certainly:

Both modes offer high performance with robust consistency, although they exhibit distinct appearances and are tailored for specific usage scenarios.

Native Mode excels in enabling concurrent access to shared data by numerous users. It boasts attributes such as real-time updates and a direct channel connecting your database to web or mobile clients.

On the other hand, Datastore Mode prioritizes exceptional throughput, catering to intensive reading and writing operations.

    3. In the Select a location dropdown, choose a database region closest to your location and then click Create Database.

Step 4. Writing Database Import code

    In Cloud Shell, run the following command to clone the Pet Theory repository:

        git clone https://github.com/rosera/product-name

        As a reference, and consideration in importing data and code pre-installed.
    
    Then change your current working directory to lab01:

        cd pet-theory/lab01

        In the directory you can see Products's package.json. This file lists the packages that your Node.js project depends on and makes your build reproducible, and therefore easier to share with others.

        An example package.json is shown below:

        {
	        "name": "lab01",
	        "version": "1.0.0",
	        "description": "This is lab01 of the Pet Theory labs",
	        "main": "index.js",
	        "scripts": {
		    "test": "echo \"Error: no test specified\" && exit 1"
	        },
	        "keywords": [],
	        "author": "Patrick - IT",
	        "license": "MIT",
	        "dependencies": {
		        "csv-parse": "^4.4.5"
	    }

        }
    
    Run the following command to do so:

        npm install @google-cloud/firestore

    To enable the app to write logs to Cloud Logging, install an additional module:

        npm install @google-cloud/logging
    
    After successful completion of the command, the package.json will be automatically updated to include the new peer dependencies, and will look like this.

        ...
        "dependencies": {
          "@google-cloud/firestore": "^6.4.1",
          "@google-cloud/logging": "^10.3.1",
          "csv-parse": "^4.4.5"
        }
        Now it's time to take a look at the script that reads the CSV file of customers and writes one record in Firestore for each line in the         CSV file. Patrick's original application is shown below:

        const { promisify } = require("util");
        const parse = promisify(require("csv-parse"));
        const { readFile } = require("fs").promises;
        if (process.argv.length < 3) {
        	console.error("Please include a path to a csv file");
        	process.exit(1);
        }
        function writeToDatabase(records) {
        	records.forEach((record, i) => {
        		console.log(
        			`ID: ${record.id} Email: ${record.email} Name: ${record.name} Phone: ${record.phone}`
        		);
        	});
        	return;
        }
        async function importCsv(csvFileName) {
        	const fileContents = await readFile(csvFileName, "utf8");
        	const records = await parse(fileContents, { columns: true });
        	try {
        		await writeToDatabase(records);
        	} catch (e) {
        		console.error(e);
        		process.exit(1);
        	}
        	console.log(`Wrote ${records.length} records`);
        }
        importCsv(process.argv[2]).catch((e) => console.error(e));

    It takes the output from the input CSV file and imports it into the legacy database. Next, update this code to write to Firestore.

    Open the file pet-theory/lab01/importTestData.js.

        To reference the Firestore API via the application, you need to add the peer dependency to the existing codebase.

    //its an exmple code and work 

    Add the following Firestore dependency on line 4 of the file:

        const { Firestore } = require("@google-cloud/firestore");

    Ensure that the top of the file looks like this:

        const { promisify } = require("util");
        const parse = promisify(require("csv-parse"));
        const { readFile } = require("fs").promises;
        const { Firestore } = require("@google-cloud/firestore"); // add this
    
    Add the following code underneath line 9, or the if (process.argv.length < 3) conditional:

                const db = new Firestore();
        function writeToFirestore(records) {
        	const batchCommits = [];
        	let batch = db.batch();
        	records.forEach((record, i) => {
        		var docRef = db.collection("customers").doc(record.email);
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
    
    The above code snippet declares a new database object, which references the database created earlier in the lab. The function uses a batch process in which each of the records is processed in turn and sets a document reference based on the identifier added. At the end of the function, the batch content is written to the database.

    Next, add logging for the application. To reference the Logging API via the application, add the peer dependency to the existing codebase. Add the line following line just below the other require statements at the top of the file:
        
        const {Logging} = require('@google-cloud/logging');

    Ensure that the top of the file looks like this:

        const { promisify } = require("util");
        const parse = promisify(require("csv-parse"));
        const { readFile } = require("fs").promises;
        const { Firestore } = require("@google-cloud/firestore");
        const { Logging } = require("@google-cloud/logging"); //add this

    Add a few constant variables and initialize the Logging client. Add those just below the above lines in the file (~line 5), like this:
        
        const logName = "pet-theory-logs-importTestData";
        // Creates a Logging client
        const logging = new Logging();
        const log = logging.log(logName);
        const resource = {
        	type: "global",
        };

    Add code to write the logs in importCsv function just below the line console.log(Wrote ${records.length} records); which should look like this:

        // A text log entry
        success_message = `Success: importTestData - Wrote ${records.length} records`;
        const entry = log.entry(
        	{ resource: resource },
        	{ message: `${success_message}` }
        );
        log.write([entry]);

    After these updates, your importCsv function code block should look like the following:

        async function importCsv(csvFileName) {
        	const fileContents = await readFile(csvFileName, "utf8");
        	const records = await parse(fileContents, { columns: true });
        	try {
        		await writeToFirestore(records);
        		//await writeToDatabase(records);
        	} catch (e) {
        		console.error(e);
        		process.exit(1);
        	}
        	console.log(`Wrote ${records.length} records`);
        	// A text log entry
        	success_message = `Success: importTestData - Wrote ${records.length} records`;
        	const entry = log.entry(
        		{ resource: resource },
        		{ message: `${success_message}` }
        	);
        	log.write([entry]);
        }

Step 5. Creating test data

    First, install the "faker" library, which will be used by the script that generates the fake customer data. Run the following command to update the dependency in package.json:

        npm install faker@5.5.3

    Now open the file named createTestData.js with the code editor and inspect the code. Ensure it looks like the following:

        const fs = require("fs");
        const faker = require("faker");
        function getRandomCustomerEmail(firstName, lastName) {
        	const provider = faker.internet.domainName();
        	const email = faker.internet.email(firstName, lastName, provider);
        	return email.toLowerCase();
        }
        async function createTestData(recordCount) {
        	const fileName = `customers_${recordCount}.csv`;
        	var f = fs.createWriteStream(fileName);
        	f.write("id,name,email,phone\n");
        	for (let i = 0; i < recordCount; i++) {
        		const id = faker.datatype.number();
        		const firstName = faker.name.firstName();
        		const lastName = faker.name.lastName();
        		const name = `${firstName} ${lastName}`;
        		const email = getRandomCustomerEmail(firstName, lastName);
        		const phone = faker.phone.phoneNumber();
        		f.write(`${id},${name},${email},${phone}\n`);
        	}
        	console.log(`Created file ${fileName} containing ${recordCount} records.`);
        }
        recordCount = parseInt(process.argv[2]);
        if (process.argv.length != 3 || recordCount < 1 || isNaN(recordCount)) {
        	console.error("Include the number of test data records to create. Example:");
        	console.error("    node createTestData.js 100");
        	process.exit(1);
        }
        createTestData(recordCount);

    Add Logging for the codebase. On line 3, add the following reference for the Logging API module from the application code:

        const { Logging } = require("@google-cloud/logging");
    
    Now, add a few constant variables and initialize the Logging client. Add those just below the const statements:

        const logName = "pet-theory-logs-createTestData";
        // Creates a Logging client
        const logging = new Logging();
        const log = logging.log(logName);
        const resource = {
        	// This example targets the "global" resource for simplicity
        	type: "global",
        };

    Add code to write the logs in the createTestData function just below the line console.log(Created file ${fileName} containing ${recordCount} records.); which will look like this:

        // A text log entry
 const success_message = `Success: createTestData - Created file ${fileName} containing ${recordCount} records.`;
        const entry = log.entry(
        	{ resource: resource },
        	{
        		name: `${fileName}`,
        		recordCount: `${recordCount}`,
        		message: `${success_message}`,
        	}
        );
        log.write([entry]);

    Run the following command to configure your Project ID in Cloud Shell, replacing PROJECT_ID with your Qwiklabs Project ID:

        gcloud config set project PROJECT_ID

    Now set the project ID as an environment variable:

        PROJECT_ID=$(gcloud config get-value project)
    
    Run the following command in Cloud Shell to create the file customers_1000.csv, which will contain 1000 records of test data:

        node createTestData 1000

    Step 6. Imporing the test Customer Data

        To test the import capability, use both the import script and the test data created earlier:

            node importTestData customers_1000.csv
        
        If you get an error that resembles the following:

            npm install csv-parse

        At this point, if you are feeling adventurous, feel free to create a larger test data file and import it into the Firestore database:

            node createTestData 20000
            node importTestData customers_20000.csv

# The end of the conclusion gets onto import data to Firstore Database

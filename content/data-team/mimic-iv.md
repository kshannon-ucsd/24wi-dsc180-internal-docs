---
title: "Data Team Documentation"
date: 2025-01-21
---

# Obtaining MIMIC 4 Database

## Overview
The document guides you through the installation of the MIMIC 4 database along with concept tables. 

---

## Accessing the MIMIC Dataset

The MIMIC-IV database is a freely accessible, de-identified dataset containing detailed clinical data for patients. To access the data, follow these steps:

1. **Create a PhysioNet account** at [https://physionet.org](https://physionet.org) using your UCSD email.
2. Complete the required **CITI training course** for human research protection.
3. Upload your training certificate to PhysioNet.
4. **Apply for credentialing** on PhysioNet, listing Kyle Shannon (kshannon@ucsd.edu) as your supervisor.
5. Once approved, sign the **Data Use Agreement** for MIMIC-IV datasets.

---

## Development Environment Setup

### 1. PostgreSQL Database Setup

- **Create the MIMIC Database**:
  1. Clone the mimic-code repository to a directory of your choosing using the following:
     ```bash
     git clone https://github.com/MIT-LCP/mimic-code.git
     ```
  2. Go to the [MIMIC Github](https://github.com/MIT-LCP/mimic-code/tree/main/mimic-iv/buildmimic/postgres) and make ensure you are in mimic-iv/buildmimic/postgres
  3. Follow the instructions in the README file using the wget command provided to you on the [MIMIC 4 website](https://physionet.org/content/mimiciv/3.1/)
        - There have been updates to MIMIC 4 since the creation of this script, replace any 2.2 you see in the provided code in the GitHub with the current version number of MIMIC 4
            - You can find the version number at the end of the wget command 

---

## pgAdmin 4 Database Access

You can manage the PostgreSQL database using pgAdmin. Here’s how to connect:

1. Open pgAdmin and create a new PostgreSQL connection.
2. Use the following settings:
   - Host: `localhost`
   - Port: `5432`
   - Database: `mimiciv`
   - Username: `postgres`
   - Password: `postgres`
     

---
## Concept Tables

1. **Download concepts tables from MIT-LCP/mimic-code GitHub**
   
   The project requires some tables from the `mimic-iv/concepts_progres` folder in the GitHub. You can find the scripts here:
   - [MIT-LCP/mimic-code GitHub](https://github.com/MIT-LCP/mimic-code)
---

## Concepts Tables Setup

 1. Clone the mimic-code repository to a directory of your choosing using the following:
     ```bash
     git clone https://github.com/MIT-LCP/mimic-code.git
     ```
  2. In your terminal, navigate to mimic-iv/concepts_postgres
  3. Connect to the Database:
     ```bash
     psql -U postgres -d mimiciv
     ```
  4. Run SQL script
     ```bash
     \i postgres-make-concepts.sql
     ```
  - For more detailed instruction, refer to the [README.md](https://github.com/MIT-LCP/mimic-code/blob/main/mimic-iv/concepts_postgres/README.md) file on MIT's GitHub.

    
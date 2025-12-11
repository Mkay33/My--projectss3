Building a Simple Database App Using Amazon RDS + Python/Flask
Connection Pooling • Backup Strategies • Why Not to Run Databases on EC2

This guide explains how to build a simple database-powered application using Amazon RDS (Relational Database Service) with a Python/Flask backend.
You will learn how to provision an RDS instance, connect to it from Flask, implement connection pooling, enable backups, and understand best practices.

📌 Overview

You will build:

A simple Flask web app

Connected to an RDS database (MySQL / PostgreSQL)

Using connection pooling for performance

With automated backups & snapshots

Following best practices (including why not to run DBs on EC2)

🏗️ 1. Create an RDS Database
Steps:

Go to AWS Console → RDS

Click Create Database

Choose Standard Create

Select an Engine:

MySQL (popular, beginner-friendly)

PostgreSQL (recommended for enterprise apps)
<img width="968" height="837" alt="image" src="https://github.com/user-attachments/assets/396266b1-9025-490a-8bee-17c3656dd7dd" />

Select Free Tier or Dev/Test template
<img width="932" height="837" alt="image" src="https://github.com/user-attachments/assets/9c0d439d-bfc3-48ef-bd70-eb000e1ac91a" />

Configure Settings:

DB Instance Identifier → my-flask-db

Master Username → admin
<img width="1871" height="558" alt="image" src="https://github.com/user-attachments/assets/5bdb27f2-e9b4-48a1-b6c5-252eede76423" />

Password → choose securely

Choose Instance Class:

Free-tier: db.t3.micro
<img width="1940" height="862" alt="image" src="https://github.com/user-attachments/assets/81c6e504-704b-43e9-81ac-c0cc67aff05f" />

Storage:

General Purpose SSD (gp3)

Connectivity:

Select a VPC
<img width="1843" height="842" alt="image" src="https://github.com/user-attachments/assets/41892523-a45a-4443-8361-c51d4e015fd1" />

Create or choose a Security Group
<img width="1848" height="863" alt="image" src="https://github.com/user-attachments/assets/6d01adff-605d-42be-aeb2-14e9ca474ef6" />

Allow inbound port 3306 (MySQL) or 5432 (Postgres) from your application server or local IP

Create the database.
<img width="1910" height="732" alt="image" src="https://github.com/user-attachments/assets/ca5ae630-83e7-4582-88e1-1f1fa4cfb5bc" />

Important:

⛔ Do NOT set your database to public unless needed for learning/testing.

🧩 2. Build a Simple Python/Flask App

Install Flask + database drivers:

pip install flask pymysql sqlalchemy psycopg2-binary

Basic Flask App Structure:
project/
│── app.py
│── config.py
│── requirements.txt
│── templates/
│── static/

🔌 3. Connect Flask to RDS Using SQLAlchemy (With Connection Pooling)
config.py
import os

DB_HOST = os.getenv("DB_HOST", "your-rds-endpoint")
DB_USER = "admin"
DB_PASSWORD = "yourpassword"
DB_NAME = "mydbname"

SQLALCHEMY_DATABASE_URI = (
    f"mysql+pymysql://{DB_USER}:{DB_PASSWORD}@{DB_HOST}/{DB_NAME}"
)
<img width="1907" height="722" alt="image" src="https://github.com/user-attachments/assets/0d246ef1-a8d0-4d71-b0b6-31ef3b5f2649" />

app.py (with connection pooling)
from flask import Flask, request, jsonify
from sqlalchemy import create_engine, text
from config import SQLALCHEMY_DATABASE_URI

app = Flask(__name__)
<img width="976" height="800" alt="image" src="https://github.com/user-attachments/assets/20f49e71-a5dd-4653-ae2c-d036a3e4d21d" />

# Create pooled DB engine
engine = create_engine(
    SQLALCHEMY_DATABASE_URI,
    pool_size=5,          # number of persistent connections
    max_overflow=10,      # temporary extra connections
    pool_timeout=30,      # seconds to wait if pool is full
    pool_recycle=1800     # refresh stale connections (30 min)
)

@app.route("/users", methods=["POST"])
def create_user():
    data = request.json
    with engine.connect() as conn:
        conn.execute(
            text("INSERT INTO users (name, email) VALUES (:name, :email)"),
            {"name": data["name"], "email": data["email"]}
        )
    return jsonify({"message": "User created"}), 201

@app.route("/users", methods=["GET"])
def get_users():
    with engine.connect() as conn:
        results = conn.execute(text("SELECT * FROM users")).fetchall()
        users = [dict(row._mapping) for row in results]
    return jsonify(users)

if __name__ == "__main__":
    app.run(debug=True)
<img width="993" height="982" alt="image" src="https://github.com/user-attachments/assets/f370b679-2d50-41da-bd63-7b875cee8357" />
<img width="1881" height="868" alt="image" src="https://github.com/user-attachments/assets/7b8128ca-d38f-456f-9d23-5e4b71069fbf" />

🗃️ 4. Create a Users Table in RDS

Connect using a MySQL/Postgres client:

CREATE TABLE users (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

🔁 5. Understanding Connection Pooling
<img width="1349" height="108" alt="image" src="https://github.com/user-attachments/assets/1d5d9f9d-8298-426b-bacf-116343a4339c" />
<img width="1152" height="228" alt="image" src="https://github.com/user-attachments/assets/2bf6c88a-1da3-443c-a5bc-2e65520707db" />

🔒 6. Backup Strategies in RDS
<img width="1592" height="616" alt="image" src="https://github.com/user-attachments/assets/e70e25b7-fe3c-4af0-b37c-712e51e62b4e" />

Amazon RDS provides:

✅ Automated Backups

Daily backups

Point-in-time recovery

Retention: 1–35 days

Configurable under:
RDS → Databases → Modify → Backup

✅ Manual Snapshots

On-demand, not automatically deleted

Good before major system changes

✅ Multi-AZ Deployment

Real-time standby replica in another Availability Zone

Automatic failover

Recommended for production

Backup Best Practices:

Enable 7–30 days retention

Take snapshots before deployments

Store snapshots long-term for compliance

Enable auto minor version upgrades

🛑 7. Why You Should NOT Run Databases on EC2

Some beginners try launching MySQL/Postgres manually on EC2 — don’t do this.

❌ Problems With Running DBs on EC2:
1. No automatic backups

You must script your own backup strategy.

2. No automated failover

If your EC2 crashes, your database is gone.

3. No monitoring

You must install, configure, patch, and maintain everything.

4. No scaling

RDS scales vertically & horizontally; EC2 requires manual tuning.

5. No Multi-AZ

RDS gives cross-AZ redundancy automatically.

6. Security risks

Wrong firewall setup = public unsecured database.

7. Time-consuming administration

Patching, upgrades, maintenance → all manual.

In Summary:

RDS = Managed Database
EC2 = “Do it yourself” database (high risk, high maintenance)

Only run databases on EC2 for very special use cases (custom DB engines, unusual requirements).

🚀 8. Testing Your App

Run Flask locally:

python app.py


Add a user:

curl -X POST -H "Content-Type: application/json" \
-d '{"name":"John Doe","email":"john@example.com"}' \
http://127.0.0.1:5000/users


Retrieve users:
<img width="999" height="147" alt="image" src="https://github.com/user-attachments/assets/fe5501d8-d102-4baf-8938-c4bc5e3a839f" />

curl http://127.0.0.1:5000/users

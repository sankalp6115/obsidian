**Duration:** 13 hours  
**Learning Style:** Formal definitions + Theory + Practical examples  
**Prerequisites:** Week 1-4 completed  
**Goal:** Master database design, SQL basics, and SQLAlchemy ORM for practical applications

---

## Table of Contents

1. [SQL & Database Basics - 2 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#sql--database-basics---2-hours)
2. [SQLAlchemy ORM Fundamentals - 5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#sqlalchemy-orm-fundamentals---5-hours)
3. [Alembic Migrations - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#alembic-migrations---25-hours)
4. [SQLAlchemy + FastAPI Integration - 2.5 Hours](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#sqlalchemy--fastapi-integration---25-hours)
5. [Week 5 Checkpoint Project](https://claude.ai/chat/d11194de-eb9b-484d-a2a4-76cdf59d77ed#week-5-checkpoint-project)

---

# SQL & DATABASE BASICS - 2 HOURS

## Part 1: Relational Database Fundamentals

### Formal Definition: Relational Database

**Relational Database:** A database that organizes data into structured tables with rows and columns, where data is related through keys. Based on relational algebra.

**Key concepts:**

- **Table:** Collection of related data (like a spreadsheet)
- **Row:** A single record in a table
- **Column:** A field/attribute in a table
- **Primary Key:** Unique identifier for each row
- **Foreign Key:** Reference to primary key in another table
- **Schema:** Structure defining tables and relationships

### Table Structure Example

```
Table: users
┌────────┬──────────────┬──────────────────┬────────┐
│ id (PK)│ username     │ email            │ role   │
├────────┼──────────────┼──────────────────┼────────┤
│ 1      │ alice        │ alice@example.com│ admin  │
│ 2      │ bob          │ bob@example.com  │ user   │
│ 3      │ charlie      │ charlie@ex.com   │ editor │
└────────┴──────────────┴──────────────────┴────────┘

Table: posts
┌────────┬────────────────┬───────────────┬──────────────┬────────────┐
│ id (PK)│ title          │ content       │ user_id (FK) │ created_at │
├────────┼────────────────┼───────────────┼──────────────┼────────────┤
│ 1      │ First Post     │ Hello World   │ 1 (alice)    │ 2024-01-01 │
│ 2      │ My Adventure   │ I went to...  │ 2 (bob)      │ 2024-01-02 │
│ 3      │ Tech Tips      │ Here's how... │ 1 (alice)    │ 2024-01-03 │
└────────┴────────────────┴───────────────┴──────────────┴────────────┘

Relationship:
- user_id in posts references id in users
- Alice (id=1) has 2 posts (id=1, id=3)
- Bob (id=2) has 1 post (id=2)
```

### Formal Definition: ACID Properties

**ACID:** Guarantees that ensure database reliability.

- **Atomicity:** Transaction is "all or nothing" - either completes fully or not at all
    
    ```
    Transfer money from Alice to Bob:
    1. Debit Alice's account
    2. Credit Bob's account
    If step 2 fails, step 1 rolls back automatically
    ```
    
- **Consistency:** Database stays in valid state before and after transaction
    
    ```
    Account balances can never be negative
    Even if transaction fails mid-way
    ```
    
- **Isolation:** Concurrent transactions don't interfere
    
    ```
    Two users updating same record simultaneously
    Each sees consistent version
    Changes don't corrupt data
    ```
    
- **Durability:** Committed data survives system failures
    
    ```
    After transaction commits, data persists
    Even if server crashes immediately after
    ```
    

---

## Part 2: SQL Basics for FastAPI Developers

### Formal Definition: SQL (Structured Query Language)

**SQL:** A standardized language for querying and manipulating data in relational databases.

**Important note for FastAPI:** You'll mostly use ORMs (SQLAlchemy) that generate SQL, but understanding SQL helps debug and optimize.

### SELECT - Retrieving Data

```sql
-- Basic SELECT
SELECT id, username, email FROM users;
-- Returns all user IDs, usernames, emails

-- With WHERE clause (filtering)
SELECT * FROM users WHERE role = 'admin';
-- Returns all columns for admin users only

-- With ORDER BY (sorting)
SELECT username, email FROM users ORDER BY username ASC;
-- Returns users sorted alphabetically

-- With LIMIT (pagination)
SELECT * FROM users LIMIT 10 OFFSET 20;
-- Returns 10 users starting from position 20
-- (skip 20, take 10)

-- With JOIN (combining tables)
SELECT users.username, posts.title
FROM users
JOIN posts ON users.id = posts.user_id;
-- Returns username and their posts
```

### INSERT - Adding Data

```sql
-- Basic INSERT
INSERT INTO users (username, email, role)
VALUES ('alice', 'alice@example.com', 'admin');
-- Adds new user row

-- Multiple rows
INSERT INTO users (username, email, role) VALUES
('alice', 'alice@example.com', 'admin'),
('bob', 'bob@example.com', 'user'),
('charlie', 'charlie@example.com', 'editor');
```

### UPDATE - Modifying Data

```sql
-- Update specific row
UPDATE users SET email = 'newemail@example.com'
WHERE id = 1;
-- Changes email for user with id=1

-- Update multiple rows
UPDATE users SET role = 'user' WHERE role IS NULL;
-- Sets role to 'user' for anyone without role
```

### DELETE - Removing Data

```sql
-- Delete specific row
DELETE FROM users WHERE id = 1;
-- Removes user with id=1

-- Delete multiple rows
DELETE FROM posts WHERE user_id = 1;
-- Removes all posts by user 1
```

---

## Part 3: Relationships and Keys

### Formal Definition: Primary Key

**Primary Key:** A column (or columns) that uniquely identifies each row. Every table should have a primary key.

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,  -- id is primary key
    username VARCHAR(50),
    email VARCHAR(100)
);
-- Each user has unique id
-- No two users can have same id
```

### Formal Definition: Foreign Key

**Foreign Key:** A column that references the primary key of another table. Establishes relationships between tables.

```sql
CREATE TABLE posts (
    id INT PRIMARY KEY,
    title VARCHAR(200),
    user_id INT,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
-- user_id references users.id
-- Every post must belong to existing user
-- Can't have post with user_id=999 if user doesn't exist
```

### Relationship Types

**1. One-to-Many (1:M)**

```
One User → Many Posts
User Alice (id=1) → Post 1, Post 3, Post 5
User Bob (id=2) → Post 2, Post 4

Implemented via foreign key in "many" side
```

**2. Many-to-Many (M:M)**

```
Many Users ↔ Many Tags
User can have many tags
Tag can be used by many users

Implemented via junction table:
Table: user_tags
┌────────┬────────┐
│ user_id│ tag_id │
├────────┼────────┤
│ 1      │ 5      │
│ 1      │ 7      │
│ 2      │ 5      │
└────────┴────────┘
```

**3. One-to-One (1:1)**

```
One User ↔ One Profile
User has exactly one profile
Profile belongs to exactly one user

Foreign key in one table with UNIQUE constraint
```

---

# SQLALCHEMY ORM FUNDAMENTALS - 5 HOURS

## Part 1: Understanding ORMs

### Formal Definition: ORM (Object-Relational Mapping)

**ORM:** A programming technique that maps database tables to Python classes, and rows to instances of those classes. Allows you to work with databases using object-oriented code instead of writing SQL.

### Problem Without ORM

```python
# ❌ Manual SQL - error-prone, verbose
import sqlite3

conn = sqlite3.connect('database.db')
cursor = conn.cursor()

# Create table
cursor.execute('''
    CREATE TABLE users (
        id INTEGER PRIMARY KEY,
        username TEXT,
        email TEXT
    )
''')

# Insert user
cursor.execute(
    'INSERT INTO users (username, email) VALUES (?, ?)',
    ('alice', 'alice@example.com')
)
conn.commit()

# Query users
cursor.execute('SELECT * FROM users')
users = cursor.fetchall()
for user in users:
    print(user[0], user[1], user[2])  # Which is which?

# Update user
cursor.execute(
    'UPDATE users SET email = ? WHERE id = ?',
    ('newemail@example.com', 1)
)
conn.commit()

conn.close()

# Problems:
# 1. No type safety
# 2. SQL strings error-prone
# 3. Manual connection management
# 4. No validation
# 5. Hard to refactor
```

### Solution With ORM

```python
# ✅ SQLAlchemy ORM - clean, safe, maintainable
from sqlalchemy import Column, Integer, String, create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

Base = declarative_base()

# Define table as Python class
class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    username = Column(String)
    email = Column(String)

# Create database connection
engine = create_engine('sqlite:///database.db')
Base.metadata.create_all(engine)
SessionLocal = sessionmaker(bind=engine)
session = SessionLocal()

# Create user (OOP style)
user = User(username='alice', email='alice@example.com')
session.add(user)
session.commit()

# Query users
users = session.query(User).all()
for user in users:
    print(user.username, user.email)  # Type-safe!

# Update user
user = session.query(User).filter(User.id == 1).first()
user.email = 'newemail@example.com'
session.commit()

session.close()

# Benefits:
# 1. Type-safe
# 2. No SQL strings
# 3. Automatic connection pooling
# 4. Built-in validation
# 5. Easy to refactor
# 6. Pythonic API
```

---

## Part 2: Defining ORM Models

### Basic Model Definition

```python
from sqlalchemy import Column, Integer, String, DateTime, Boolean
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime

Base = declarative_base()

class User(Base):
    """
    User table model
    __tablename__ specifies database table name
    """
    __tablename__ = "users"
    
    # Column definitions
    # Column(type, constraints)
    
    id = Column(Integer, primary_key=True)  # Auto-incrementing ID
    username = Column(String(50), unique=True, nullable=False)  # Required, must be unique
    email = Column(String(100), unique=True, nullable=False)  # Required, must be unique
    full_name = Column(String(100))  # Optional (nullable=True by default)
    is_active = Column(Boolean, default=True)  # Defaults to True
    created_at = Column(DateTime, default=datetime.utcnow)  # Auto-set to current time
    
    def __repr__(self):
        """String representation of User"""
        return f"<User(id={self.id}, username='{self.username}')>"

# Usage
user = User(
    username='alice',
    email='alice@example.com',
    full_name='Alice Wonder'
)
# created_at and is_active set to defaults automatically
```

### Column Types

```python
from sqlalchemy import (
    String, Integer, Float, Boolean, DateTime, 
    Date, Time, Text, JSON, Enum
)
from enum import Enum as PyEnum

# String/Text
username = Column(String(50))  # VARCHAR(50)
bio = Column(Text)  # TEXT (unlimited length)

# Numbers
age = Column(Integer)  # INT
price = Column(Float)  # FLOAT
rating = Column(Integer)  # 0-5 stars

# Booleans
is_active = Column(Boolean)  # TRUE/FALSE

# Date/Time
created_at = Column(DateTime)  # DATETIME
birth_date = Column(Date)  # DATE only
wake_time = Column(Time)  # TIME only

# JSON (store complex data)
metadata = Column(JSON)  # Stores JSON objects

# Enum (restricted choices)
class UserRole(PyEnum):
    ADMIN = "admin"
    EDITOR = "editor"
    USER = "user"

role = Column(Enum(UserRole), default=UserRole.USER)
```

### Column Constraints

```python
from sqlalchemy import Column, Integer, String, ForeignKey, CheckConstraint

# Primary Key
id = Column(Integer, primary_key=True)

# Unique
email = Column(String, unique=True)  # No duplicates

# Not Null
username = Column(String, nullable=False)  # Required

# Default values
is_active = Column(Boolean, default=True)
created_at = Column(DateTime, default=datetime.utcnow)

# Foreign Key
user_id = Column(Integer, ForeignKey('users.id'))  # References users table

# Check Constraint
age = Column(Integer, CheckConstraint('age >= 0 AND age <= 150'))

# Multiple constraints
email = Column(
    String(100),
    unique=True,        # Must be unique
    nullable=False,     # Required
    index=True          # Index for faster queries
)
```

---

## Part 3: Relationships

### Formal Definition: Relationship

**Relationship:** A SQLAlchemy feature that creates a Python attribute linking two related models. Automatically handles foreign keys and joins.

### One-to-Many Relationship

**Definition:** One parent has many children.

```python
from sqlalchemy import Column, Integer, String, ForeignKey, DateTime
from sqlalchemy.orm import relationship
from datetime import datetime

Base = declarative_base()

class User(Base):
    """Parent model"""
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    username = Column(String)
    
    # Relationship: User has many Posts
    # relationship() creates Python attribute
    # back_populates creates reverse relationship
    posts = relationship(
        "Post",
        back_populates="author",
        cascade="all, delete-orphan"  # Delete posts when user deleted
    )

class Post(Base):
    """Child model"""
    __tablename__ = "posts"
    
    id = Column(Integer, primary_key=True)
    title = Column(String)
    user_id = Column(Integer, ForeignKey("users.id"))  # Foreign key to user
    
    # Relationship: Post belongs to one User
    # back_populates creates reverse relationship
    author = relationship(
        "User",
        back_populates="posts"
    )

# Usage
user = User(username="alice")
post = Post(title="My Post", author=user)  # Set relationship

# Can access related data through relationship
print(user.posts)  # All posts by this user
print(post.author)  # User who wrote this post
```

**Cascade options:**

```
"save-update" - Add related object when parent is saved
"delete" - Delete related objects when parent is deleted
"delete-orphan" - Delete if relationship is removed
"all" - Includes all above
```

### Many-to-Many Relationship

**Definition:** Objects can have many related objects, and those objects can have many of the first type.

```python
from sqlalchemy import Table, Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

Base = declarative_base()

# Junction table - connects users and tags
user_tags = Table(
    'user_tags',
    Base.metadata,
    Column('user_id', Integer, ForeignKey('users.id'), primary_key=True),
    Column('tag_id', Integer, ForeignKey('tags.id'), primary_key=True)
)

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True)
    username = Column(String)
    
    # Many-to-many relationship
    tags = relationship(
        "Tag",
        secondary=user_tags,  # Use junction table
        back_populates="users"
    )

class Tag(Base):
    __tablename__ = "tags"
    
    id = Column(Integer, primary_key=True)
    name = Column(String)
    
    # Reverse relationship
    users = relationship(
        "User",
        secondary=user_tags,
        back_populates="tags"
    )

# Usage
user = User(username="alice")
tag1 = Tag(name="python")
tag2 = Tag(name="fastapi")

# Set many-to-many relationship
user.tags = [tag1, tag2]  # Alice has Python and FastAPI tags

# Access related objects
for tag in user.tags:
    print(tag.name)  # python, fastapi

for user in tag1.users:
    print(user.username)  # alice (and others with python tag)
```

---

## Part 4: CRUD Operations with SQLAlchemy

### Formal Definition: CRUD

**CRUD:** Create, Read, Update, Delete - basic database operations.

### Create (INSERT)

```python
from sqlalchemy.orm import Session

def create_user(db: Session, username: str, email: str) -> User:
    """Create new user"""
    user = User(username=username, email=email)
    db.add(user)  # Add to session
    db.commit()  # Commit to database
    db.refresh(user)  # Refresh to get DB-generated ID
    return user

# Usage
session = SessionLocal()
new_user = create_user(session, "alice", "alice@example.com")
print(new_user.id)  # ID was auto-generated by database
```

### Read (SELECT)

```python
def get_user(db: Session, user_id: int) -> User:
    """Get user by ID"""
    return db.query(User).filter(User.id == user_id).first()

def get_user_by_username(db: Session, username: str) -> User:
    """Get user by username"""
    return db.query(User).filter(User.username == username).first()

def get_all_users(db: Session, skip: int = 0, limit: int = 10) -> list:
    """Get all users with pagination"""
    return db.query(User).offset(skip).limit(limit).all()

def get_active_users(db: Session) -> list:
    """Get only active users"""
    return db.query(User).filter(User.is_active == True).all()

# Usage
session = SessionLocal()
user = get_user(session, 1)
users = get_all_users(session, skip=0, limit=10)
```

### Update (UPDATE)

```python
def update_user(db: Session, user_id: int, email: str) -> User:
    """Update user email"""
    user = db.query(User).filter(User.id == user_id).first()
    if user:
        user.email = email  # Modify attribute
        db.commit()  # Commit changes
        db.refresh(user)  # Refresh to get updated data
    return user

# Usage
session = SessionLocal()
updated_user = update_user(session, 1, "newemail@example.com")
```

### Delete (DELETE)

```python
def delete_user(db: Session, user_id: int) -> bool:
    """Delete user"""
    user = db.query(User).filter(User.id == user_id).first()
    if user:
        db.delete(user)  # Mark for deletion
        db.commit()  # Commit deletion
        return True
    return False

# Usage
session = SessionLocal()
success = delete_user(session, 1)
```

---

## Part 5: Querying and Filtering

### Formal Definition: Query

**Query:** An object that builds database queries in SQLAlchemy. Uses method chaining for complex queries.

### Basic Queries

```python
# Get all rows
all_users = db.query(User).all()

# Get first row
first_user = db.query(User).first()

# Get by primary key
user = db.query(User).get(1)  # Same as .filter(User.id == 1).first()

# Count rows
count = db.query(User).count()

# Check if exists
exists = db.query(User).filter(User.id == 1).first() is not None
```

### Filtering

```python
# Equal
db.query(User).filter(User.username == "alice")

# Not equal
db.query(User).filter(User.username != "alice")

# Greater than / Less than
db.query(User).filter(User.id > 5)
db.query(User).filter(User.id <= 10)

# IN (multiple values)
db.query(User).filter(User.id.in_([1, 2, 3]))

# LIKE (pattern matching)
db.query(User).filter(User.username.like("a%"))  # Starts with 'a'

# Boolean
db.query(User).filter(User.is_active == True)

# Multiple conditions (AND)
db.query(User).filter(
    User.is_active == True,
    User.username == "alice"
)

# OR condition
from sqlalchemy import or_
db.query(User).filter(
    or_(
        User.username == "alice",
        User.username == "bob"
    )
)

# NOT condition
from sqlalchemy import not_
db.query(User).filter(
    not_(User.is_active == False)
)
```

### Sorting and Pagination

```python
# Order by (ascending)
db.query(User).order_by(User.username).all()

# Order by (descending)
db.query(User).order_by(User.id.desc()).all()

# Multiple order by
db.query(User).order_by(User.username, User.id).all()

# Pagination
skip = 20
limit = 10
db.query(User).offset(skip).limit(limit).all()
# Skip 20, take next 10 (items 20-29)

# Limit only
db.query(User).limit(5).all()  # Get first 5
```

### Joining Tables

```python
# Join users with their posts
db.query(User, Post).join(Post).all()

# Get users with their posts
users_with_posts = db.query(User).join(
    Post, User.id == Post.user_id
).distinct().all()

# Get specific columns after join
db.query(User.username, Post.title).join(Post).all()

# Get user and count their posts
from sqlalchemy import func
db.query(
    User.username,
    func.count(Post.id).label("post_count")
).join(Post).group_by(User.id).all()
```

---

# ALEMBIC MIGRATIONS - 2.5 HOURS

## Part 1: Understanding Migrations

### Formal Definition: Database Migration

**Database Migration:** A version-controlled set of changes to the database schema. Allows you to evolve your database while keeping it synced across environments (dev, staging, production).

### Why Migrations Matter

```python
# ❌ WITHOUT migrations - dangerous
# Manual approach:
# 1. Developer makes schema change locally
# 2. Tries to remember to run ALTER TABLE on production
# 3. Forgets - production schema out of sync
# 4. Code expects new column, production doesn't have it
# 5. Production breaks!

# ✅ WITH migrations - safe
# 1. Developer creates migration file
# 2. Migration describes exact change
# 3. Migration applied to all environments in order
# 4. All environments stay in sync
# 5. Can rollback if something breaks
```

### Migration Concepts

```
Schema Version 1:
Users table: id, username, email

Developer adds column:
ALTER TABLE users ADD COLUMN created_at DATETIME

Schema Version 2:
Users table: id, username, email, created_at

Developer renames column:
ALTER TABLE users RENAME COLUMN created_at TO joined_at

Schema Version 3:
Users table: id, username, email, joined_at

Each step is tracked, reversible, and documented
```

---

## Part 2: Alembic Setup and Usage

### Installation and Initialization

```bash
# Install Alembic
pip install alembic

# Initialize Alembic in project
alembic init alembic

# Project structure after init:
# myproject/
# ├── alembic/
# │   ├── versions/           # Migration files
# │   ├── env.py              # Configuration
# │   ├── script.py.mako      # Migration template
# │   └── alembic.ini         # Settings
# └── main.py                 # Your app
```

### Alembic Configuration

```python
# alembic/env.py - Configure Alembic to use your models

from logging.config import fileConfig
from sqlalchemy import engine_from_config
from sqlalchemy import pool
from alembic import context
from app.database import Base  # Your models
from app.models import *  # Import all models

# Get database URL from config
config = context.config
sqlalchemy_url = config.get_main_option("sqlalchemy.url")

# Tell Alembic about your models
target_metadata = Base.metadata

def run_migrations_offline():
    """Run migrations in offline mode"""
    # Used in CI/CD pipelines
    context.configure(
        url=sqlalchemy_url,
        target_metadata=target_metadata,
        literal_binds=True,
        dialect_opts={"paramstyle": "named"},
    )
    
    with context.begin_transaction():
        context.run_migrations()

def run_migrations_online():
    """Run migrations in online mode"""
    configuration = config.get_section(config.config_ini_section)
    configuration["sqlalchemy.url"] = sqlalchemy_url
    
    connectable = engine_from_config(
        configuration,
        prefix="sqlalchemy.",
        poolclass=pool.NullPool,
    )
    
    with connectable.connect() as connection:
        context.configure(
            connection=connection,
            target_metadata=target_metadata
        )
        
        with context.begin_transaction():
            context.run_migrations()

if context.is_offline_mode():
    run_migrations_offline()
else:
    run_migrations_online()
```

### alembic.ini Configuration

```ini
# alembic.ini - Database connection string

[alembic]
sqlalchemy.url = postgresql://user:password@localhost/dbname

# For SQLite (development)
# sqlalchemy.url = sqlite:///./test.db

# For different environments:
# sqlalchemy.url = driver://user:pass@host/database
```

---

## Part 3: Creating and Running Migrations

### Auto-Generate Migrations

**Formal Definition: Auto-Migration** Alembic automatically detects changes to your models and generates migration files.

```bash
# Create initial migration from existing models
alembic revision --autogenerate -m "Initial migration"

# This creates:
# alembic/versions/a1b2c3d4_initial_migration.py

# The migration compares current models to database
# Detects all tables, columns, relationships
# Generates SQL to create/modify schema
```

### Migration File Structure

```python
# alembic/versions/a1b2c3d4_initial_migration.py

from alembic import op
import sqlalchemy as sa

# Unique identifier for this migration
revision = 'a1b2c3d4'
down_revision = None  # No previous migration
branch_labels = None
depends_on = None

def upgrade():
    """
    Run this migration forward
    Applies changes to database
    """
    op.create_table(
        'users',
        sa.Column('id', sa.Integer(), nullable=False),
        sa.Column('username', sa.String(50), nullable=False),
        sa.Column('email', sa.String(100), nullable=False),
        sa.Column('is_active', sa.Boolean(), server_default='true'),
        sa.Column('created_at', sa.DateTime()),
        sa.PrimaryKeyConstraint('id'),
        sa.UniqueConstraint('username'),
        sa.UniqueConstraint('email')
    )

def downgrade():
    """
    Run this migration backward
    Reverts changes
    """
    op.drop_table('users')
```

### Migration Workflow

```bash
# 1. Add new column to your model
# In models.py:
# class User(Base):
#     __tablename__ = "users"
#     ...
#     phone = Column(String(20))  # NEW

# 2. Auto-generate migration
alembic revision --autogenerate -m "Add phone to users"

# This creates file:
# alembic/versions/b2c3d4e5_add_phone_to_users.py
# With:
# def upgrade(): op.add_column('users', sa.Column('phone', ...))
# def downgrade(): op.drop_column('users', 'phone')

# 3. Review the migration file
# Verify it's correct

# 4. Run migration
alembic upgrade head

# 5. Verify database changed
# SELECT * FROM users;  -- Now has phone column

# 6. Commit migration file to git
git add alembic/versions/b2c3d4e5_add_phone_to_users.py
git commit -m "Add phone column to users"
```

### Running Migrations

```bash
# Upgrade to latest migration
alembic upgrade head

# Upgrade to specific migration
alembic upgrade a1b2c3d4

# Downgrade (rollback) to previous
alembic downgrade -1

# Downgrade to specific migration
alembic downgrade a1b2c3d4

# Show current migration version
alembic current

# Show all migrations
alembic history

# Show pending migrations (not yet applied)
alembic heads
```

---

## Part 4: Handling Complex Migrations

### Data Migrations

```python
# Sometimes you need to transform data during migration
# alembic/versions/c3d4e5f6_migrate_user_data.py

def upgrade():
    """
    Add new column and populate with transformed data
    """
    # Add new column
    op.add_column('users', sa.Column('full_name', sa.String(100)))
    
    # Get connection to execute custom SQL
    connection = op.get_bind()
    
    # Combine first_name and last_name into full_name
    connection.execute("""
        UPDATE users 
        SET full_name = first_name || ' ' || last_name
    """)
    
    # Remove old columns
    op.drop_column('users', 'first_name')
    op.drop_column('users', 'last_name')

def downgrade():
    """Reverse the transformation"""
    # Re-add old columns
    op.add_column('users', sa.Column('first_name', sa.String(50)))
    op.add_column('users', sa.Column('last_name', sa.String(50)))
    
    # Split full_name back
    connection = op.get_bind()
    connection.execute("""
        UPDATE users 
        SET first_name = SUBSTR(full_name, 1, INSTR(full_name, ' ') - 1),
            last_name = SUBSTR(full_name, INSTR(full_name, ' ') + 1)
    """)
    
    # Remove full_name
    op.drop_column('users', 'full_name')
```

### Handling Relationships in Migrations

```python
# Create foreign key in migration
def upgrade():
    # Add foreign key column
    op.add_column('posts', sa.Column('user_id', sa.Integer()))
    
    # Create foreign key constraint
    op.create_foreign_key(
        'fk_posts_user_id',     # Constraint name
        'posts',                 # Table
        'users',                 # Referenced table
        ['user_id'],             # Column
        ['id'],                  # Referenced column
        ondelete='CASCADE'       # Delete posts when user deleted
    )

def downgrade():
    op.drop_constraint('fk_posts_user_id', 'posts')
    op.drop_column('posts', 'user_id')
```

---

# SQLALCHEMY + FASTAPI INTEGRATION - 2.5 HOURS

## Part 1: Integration Setup

### Database Configuration

```python
# database.py - Database setup for FastAPI

from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
from typing import Generator

# Database URL
# Format: dialect+driver://username:password@host:port/database
SQLALCHEMY_DATABASE_URL = "postgresql://user:password@localhost/fastapi_db"

# For SQLite (testing/development)
# SQLALCHEMY_DATABASE_URL = "sqlite:///./test.db"

# Create engine
# echo=True shows all SQL statements (useful for debugging)
engine = create_engine(
    SQLALCHEMY_DATABASE_URL,
    echo=False,
    pool_pre_ping=True  # Verify connections before using
)

# Create SessionLocal - factory for creating sessions
SessionLocal = sessionmaker(
    autocommit=False,
    autoflush=False,
    bind=engine
)

# Base class for models
Base = declarative_base()

# Dependency that provides session to endpoints
def get_db() -> Generator:
    """
    Dependency function for FastAPI endpoints
    Provides database session
    """
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

### Model Definition

```python
# models.py - SQLAlchemy models

from sqlalchemy import Column, Integer, String, DateTime, Boolean, ForeignKey
from sqlalchemy.orm import relationship
from datetime import datetime
from database import Base

class User(Base):
    __tablename__ = "users"
    
    id = Column(Integer, primary_key=True, index=True)
    username = Column(String, unique=True, index=True)
    email = Column(String, unique=True, index=True)
    hashed_password = Column(String)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relationship to posts
    posts = relationship("Post", back_populates="author")
    
    def __repr__(self):
        return f"<User(id={self.id}, username='{self.username}')>"

class Post(Base):
    __tablename__ = "posts"
    
    id = Column(Integer, primary_key=True, index=True)
    title = Column(String, index=True)
    content = Column(String)
    published = Column(Boolean, default=False)
    user_id = Column(Integer, ForeignKey("users.id"))
    created_at = Column(DateTime, default=datetime.utcnow)
    
    # Relationship to user
    author = relationship("User", back_populates="posts")
    
    def __repr__(self):
        return f"<Post(id={self.id}, title='{self.title}')>"
```

### Pydantic Schemas

```python
# schemas.py - Pydantic models for requests/responses

from pydantic import BaseModel
from typing import Optional, List
from datetime import datetime

# ===== User Schemas =====

class UserCreate(BaseModel):
    """For creating users"""
    username: str
    email: str
    password: str

class UserUpdate(BaseModel):
    """For updating users"""
    email: Optional[str] = None
    username: Optional[str] = None

class UserResponse(BaseModel):
    """For returning users (no password!)"""
    id: int
    username: str
    email: str
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True  # Allow ORM mode

# ===== Post Schemas =====

class PostCreate(BaseModel):
    """For creating posts"""
    title: str
    content: str
    published: bool = False

class PostResponse(BaseModel):
    """For returning posts"""
    id: int
    title: str
    content: str
    published: bool
    author_id: int
    created_at: datetime
    
    class Config:
        from_attributes = True

class UserWithPosts(UserResponse):
    """User with their posts"""
    posts: List[PostResponse] = []
    
    class Config:
        from_attributes = True
```

---

## Part 2: CRUD Operations in FastAPI

### Create Operation

```python
# main.py

from fastapi import FastAPI, Depends, HTTPException, status
from sqlalchemy.orm import Session
from database import engine, Base, get_db
from models import User, Post
from schemas import UserCreate, UserResponse, PostCreate, PostResponse

# Create tables
Base.metadata.create_all(bind=engine)

app = FastAPI()

@app.post("/users/", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
def create_user(
    user: UserCreate,
    db: Session = Depends(get_db)
):
    """
    Create new user
    
    Request body: {"username": "alice", "email": "alice@example.com", "password": "..."}
    Response: User object with id, username, email
    """
    # Check if user exists
    existing_user = db.query(User).filter(User.username == user.username).first()
    if existing_user:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="Username already taken"
        )
    
    # Create new user
    db_user = User(
        username=user.username,
        email=user.email,
        hashed_password=hash_password(user.password)  # Hash password!
    )
    
    # Add to session
    db.add(db_user)
    db.commit()  # Commit to database
    db.refresh(db_user)  # Refresh to get ID
    
    return db_user
```

### Read Operation

```python
@app.get("/users/{user_id}", response_model=UserResponse)
def get_user(
    user_id: int,
    db: Session = Depends(get_db)
):
    """Get user by ID"""
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(
            status_code=status.HTTP_404_NOT_FOUND,
            detail="User not found"
        )
    
    return user

@app.get("/users/", response_model=List[UserResponse])
def list_users(
    skip: int = 0,
    limit: int = 10,
    db: Session = Depends(get_db)
):
    """List all users with pagination"""
    users = db.query(User).offset(skip).limit(limit).all()
    return users
```

### Update Operation

```python
@app.put("/users/{user_id}", response_model=UserResponse)
def update_user(
    user_id: int,
    user_update: UserUpdate,
    db: Session = Depends(get_db)
):
    """Update user"""
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Update fields if provided
    if user_update.email:
        user.email = user_update.email
    if user_update.username:
        user.username = user_update.username
    
    db.commit()
    db.refresh(user)
    return user
```

### Delete Operation

```python
@app.delete("/users/{user_id}", status_code=status.HTTP_204_NO_CONTENT)
def delete_user(
    user_id: int,
    db: Session = Depends(get_db)
):
    """Delete user"""
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    db.delete(user)
    db.commit()
    
    return None
```

---

## Part 3: Relationships and Joins

### Querying with Relationships

```python
@app.get("/users/{user_id}/posts", response_model=UserWithPosts)
def get_user_with_posts(
    user_id: int,
    db: Session = Depends(get_db)
):
    """
    Get user and all their posts
    Automatically joins posts through relationship
    """
    user = db.query(User).filter(User.id == user_id).first()
    
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Relationship automatically loads posts
    return user  # Returns user with posts included

@app.get("/posts/", response_model=List[PostResponse])
def list_posts(
    user_id: Optional[int] = None,
    db: Session = Depends(get_db)
):
    """
    List posts, optionally filtered by user
    """
    query = db.query(Post)
    
    if user_id:
        query = query.filter(Post.user_id == user_id)
    
    posts = query.all()
    return posts
```

### Creating Related Objects

```python
@app.post("/users/{user_id}/posts", response_model=PostResponse)
def create_post(
    user_id: int,
    post: PostCreate,
    db: Session = Depends(get_db)
):
    """Create post for user"""
    # Verify user exists
    user = db.query(User).filter(User.id == user_id).first()
    if not user:
        raise HTTPException(status_code=404, detail="User not found")
    
    # Create post
    db_post = Post(
        **post.dict(),
        user_id=user_id
    )
    
    db.add(db_post)
    db.commit()
    db.refresh(db_post)
    
    return db_post
```

---

# WEEK 5 CHECKPOINT PROJECT

## Project: Blog API with Database

### Objective

Build a complete Blog API using SQLAlchemy ORM with:

1. User management (create, read, update, delete)
2. Post management linked to users
3. Database migrations with Alembic
4. Relationships between tables
5. Complex queries with filtering and pagination

### Project Structure

```
blog_api/
├── alembic/
│   ├── versions/
│   ├── env.py
│   ├── script.py.mako
│   └── alembic.ini
├── app/
│   ├── __init__.py
│   ├── main.py
│   ├── database.py
│   ├── models.py
│   ├── schemas.py
│   └── crud.py
├── requirements.txt
├── .env
└── README.md
```

### Requirements

#### 1. Database Models (SQLAlchemy)

**User Model:**

- id (primary key, auto-increment)
- username (unique, required)
- email (unique, required)
- hashed_password (required)
- is_active (boolean, default True)
- created_at (datetime, auto-set)
- Relationship: has many Posts

**Post Model:**

- id (primary key, auto-increment)
- title (required, indexed)
- content (required)
- published (boolean, default False)
- user_id (foreign key to users)
- created_at (datetime, auto-set)
- Relationship: belongs to User

**Comment Model (bonus):**

- id (primary key)
- content (required)
- post_id (foreign key)
- user_id (foreign key)
- created_at (datetime)

#### 2. Pydantic Schemas

- UserCreate (username, email, password)
- UserUpdate (email, username - optional)
- UserResponse (no password field)
- PostCreate (title, content, published)
- PostUpdate (title, content, published - all optional)
- PostResponse (includes author)
- UserWithPosts (user + list of posts)

#### 3. CRUD Operations

Users:

- [ ] Create user
- [ ] Get user by ID
- [ ] List users (paginated)
- [ ] Update user
- [ ] Delete user
- [ ] Get user with posts

Posts:

- [ ] Create post (for current user)
- [ ] Get post by ID
- [ ] List posts (paginated, filterable)
- [ ] Update post (only owner)
- [ ] Delete post (only owner/admin)
- [ ] Search posts by title

#### 4. Database Migrations

- [ ] Initial migration (users, posts)
- [ ] Add created_at fields
- [ ] Create indexes on commonly queried fields
- [ ] Can rollback migrations

#### 5. Validation and Errors

- [ ] Username/email uniqueness validation
- [ ] 404 for non-existent resources
- [ ] 403 for permission errors (update others' posts)
- [ ] Proper status codes (201 for create, 204 for delete)

#### 6. Relationships

- [ ] Get user with all their posts
- [ ] Get post with author details
- [ ] Delete posts when user deleted (cascade)
- [ ] Efficient querying (avoid N+1)

### API Endpoints

```
# Users
POST   /users/                     - Create user
GET    /users/                     - List users
GET    /users/{user_id}            - Get user
PUT    /users/{user_id}            - Update user
DELETE /users/{user_id}            - Delete user
GET    /users/{user_id}/posts      - Get user's posts

# Posts
POST   /posts/                     - Create post
GET    /posts/                     - List posts
GET    /posts/{post_id}            - Get post
PUT    /posts/{post_id}            - Update post
DELETE /posts/{post_id}            - Delete post
GET    /posts/search?q=...         - Search posts
```

### Testing Checklist

- [ ] Can create users
- [ ] Username/email uniqueness enforced
- [ ] Can create posts linked to users
- [ ] Can get user with all posts
- [ ] Pagination works correctly
- [ ] Filtering by user_id works
- [ ] Search by title works
- [ ] Can't update others' posts (403)
- [ ] Delete cascade works (user deleted → posts deleted)
- [ ] Alembic migrations work
- [ ] Can rollback migrations
- [ ] Database queries are efficient

---

# EXERCISE SOLUTIONS

## SQLAlchemy Exercises

### Exercise 1: Complex Query

**Exercise:**

```python
# Get all published posts by a specific user,
# ordered by created_at (newest first),
# with pagination (skip=10, limit=5)
```

**Solution:**

```python
def get_user_published_posts(
    user_id: int,
    skip: int = 0,
    limit: int = 10,
    db: Session = Depends(get_db)
) -> List[Post]:
    return db.query(Post).filter(
        Post.user_id == user_id,
        Post.published == True
    ).order_by(
        Post.created_at.desc()
    ).offset(skip).limit(limit).all()
```

### Exercise 2: Search Functionality

**Exercise:**

```python
# Search posts by title (partial match, case-insensitive)
```

**Solution:**

```python
def search_posts(
    search_term: str,
    db: Session = Depends(get_db)
) -> List[Post]:
    return db.query(Post).filter(
        Post.title.ilike(f"%{search_term}%")
    ).all()
```

---

# SUMMARY AND CHECKLIST

## Week 5 Learning Path Summary

**Total Time: 13 Hours**

### SQL & Database Basics (2 hours)

- ✅ Relational database concepts
- ✅ Tables, rows, columns
- ✅ Primary keys and foreign keys
- ✅ Basic SQL (SELECT, INSERT, UPDATE, DELETE)
- ✅ ACID properties
- ✅ Relationships (1:M, M:M, 1:1)

### SQLAlchemy ORM Fundamentals (5 hours)

- ✅ ORM concepts and benefits
- ✅ Model definition
- ✅ Column types and constraints
- ✅ Relationships (one-to-many, many-to-many)
- ✅ CRUD operations
- ✅ Querying and filtering
- ✅ Joins and complex queries
- ✅ Lazy loading and eager loading

### Alembic Migrations (2.5 hours)

- ✅ Migration concepts
- ✅ Auto-generating migrations
- ✅ Running migrations
- ✅ Reverting (downgrading)
- ✅ Data migrations
- ✅ Handling relationships in migrations

### SQLAlchemy + FastAPI Integration (2.5 hours)

- ✅ Database configuration
- ✅ Session management in FastAPI
- ✅ Models and schemas separation
- ✅ CRUD endpoints
- ✅ Relationships in FastAPI
- ✅ Efficient querying

## Week 5 Success Criteria

By end of Week 5, you should be able to:

1. **Design database schemas** - Tables, relationships, constraints
2. **Use SQLAlchemy ORM** - Define models, relationships, queries
3. **Create migrations** - Auto-generate, run, rollback
4. **Build CRUD APIs** - With database backing
5. **Query efficiently** - Avoid N+1 problems
6. **Manage relationships** - Join tables, cascade deletes

## Key Takeaways

- **ORM abstracts SQL** - Write Pythonic code, not SQL strings
- **Relationships are powerful** - Automatically handle joins
- **Sessions manage connections** - Proper resource management
- **Migrations are critical** - Version control for schemas
- **Query building is composable** - Chain methods for complex queries
- **Type safety throughout** - ORM models + Pydantic schemas

## Common Mistakes to Avoid

1. ❌ Mixing ORM queries with raw SQL
2. ❌ Not committing after changes
3. ❌ Forgetting to refresh objects after insert
4. ❌ N+1 query problem (query in loop)
5. ❌ Not setting foreign keys properly
6. ❌ Forgetting cascade delete when needed
7. ❌ Not using migrations (manual schema changes)

## Next Steps

After Week 5, you have database integration! Week 6 covers SQLModel and advanced patterns.

**Before moving to Week 6:**

- [ ] Build Blog API checkpoint project
- [ ] All CRUD operations working
- [ ] Relationships functioning correctly
- [ ] Migrations created and working
- [ ] Database queries efficient
- [ ] Proper validation and errors

---

# ADDITIONAL RESOURCES

## Official Documentation

- SQLAlchemy ORM: https://docs.sqlalchemy.org/orm/
- Alembic: https://alembic.sqlalchemy.org/
- FastAPI + SQLAlchemy: https://fastapi.tiangolo.com/advanced/sql-databases/

## Databases

- PostgreSQL: https://www.postgresql.org/
- SQLite: https://www.sqlite.org/ (good for development/testing)

## Tools

- pgAdmin: PostgreSQL GUI admin tool
- SQLiteStudio: SQLite GUI tool
- DBeaver: Universal database tool

## Common Patterns

- Connection pooling: Reuse DB connections efficiently
- N+1 prevention: Eager load related objects
- Soft deletes: Mark deleted, don't remove

---

**End of Week 5 Content**

Excellent! You now have production-ready database skills. Week 6 covers SQLModel and advanced patterns. 🚀
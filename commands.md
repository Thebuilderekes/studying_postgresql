-- Duplicate a database
``CREATE DATABASE target_db WITH TEMPLATE source_db;``

## Option 2: Copy to a Different Server (Dump & Restore)

If you need to move the copy to a separate physical server, you must export it to a file from your terminal (not inside psql), move the file, and restore it.

``pg_dump -U role_name -d source_db -f database_backup.sql``

-- connect to a database after logging in to a database
`` \c databasename``

-- Create user role with password while in a database
``CREATE user role_name with password 'superSecret' login;``
or
 ``CREATE ROLE role_name with login password 'supersecret';``


-- Allow the user to connect to the database
``GRANT CONNECT ON DATABASE target_db TO role_name;``

-- Allow the user to create new schemas or objects (Optional)
``GRANT CREATE ON DATABASE target_db TO role_name;``

-- Allow the user to see the schema
``GRANT USAGE ON SCHEMA database_name TO role_name;``

-- Grant Read-Only access to all existing tables
``GRANT SELECT ON ALL TABLES IN SCHEMA database_name TO role_name;``

-- OR Grant Full Read/Write access to all existing tables
``GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA database_name  TO
role_name;``

-- Grant permissions for sequences (required if tables use auto-incrementing IDs)
``GRANT USAGE, SELECT ON ALL SEQUENCES IN SCHEMA database_name TO role_name;``
i

## Managing tables
-- To see the size of a database
``\l+ databasename``

-- to see the details of a table
``\d tablename``

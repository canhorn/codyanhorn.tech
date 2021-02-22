---
layout: post
author: Cody Merritt Anhorn
title: .NET - Simple Database Migration
categories: [blog]
tags: [.NET, C#, Database, Dapper]
---

This article has some source code for a light Database Migration processor, using a few tables for tracking and Dapper for the execution of the SQL. 

## Background

This code was developed mainly to see how hard it would be for me to create a Database Migration feature by myself. It turned out to be easier than I thought, since it mainly just two parts. The first is a way to track and run scripts, this is done with a simple table that keeps track of the Name and Execution Date of the script, this way any scripts with the same name will not be executed. The second was a way to add new migrations, the easiest way that I landed on was to create SQL files, the ways I structure these files is in a way that they are non-destructive. The non-destructive part is optional, since if they scripts run once the processing would not run them again.

I have included the source for my personal implementation of a Database Migration processor, it also includes a few scripts. One to setup the MigrationScripts database, and another to create a Registration table for tracking a user and some details about them. I have include a few resources, the first is a good reference I used to get started on this processor. Another for details on how to structure SQL scripts to check for existing tables and the other detailing some pitfalls to be aware of when running these in Kubernetes.

## Resources

[Database migrations made simple](https://www.kenneth-truyers.net/2016/06/02/database-migrations-made-simple/)

[How to check if a database exists in SQL Server?](https://stackoverflow.com/questions/679000/how-to-check-if-a-database-exists-in-sql-server/)

[Running database migrations when deploying to Kubernetes](https://andrewlock.net/deploying-asp-net-core-applications-to-kubernetes-part-7-running-database-migrations/amp/)

## Source Code

~~~ csharp
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlClient;
using System.IO;
using System.Linq;
using Dapper;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;

internal class DatabaseMigration
{
    internal static void Run(
        // This assumes the usage of ASP.NET Core Services
        // But could be replaced to create SQLConnection's without them
        IServiceProvider services
    )
    {
        var connection = services.GetRequiredService<IDbConnection>();
        var connectionStringDatabase = connection.Database;
        // Check to make sure that our database exists
        CheckCreateDatabase(
            // This takes the current connection string and replaces it with master.
            // This way we can make selects against the database to check for an existing database.
            connection.ConnectionString.Replace(
                connectionStringDatabase,
                "master"
            ),
            connectionStringDatabase
        );

        // Check if the migrations table exists, otherwise execute the first script (which creates that table) 
        if (connection.ExecuteScalar<int>(
            @"SELECT count(1) FROM sys.tables
                WHERE Name = 'MigrationScripts'"
            ) == 0
        )
        {
            // This is the name of my initialization script, checkout below for what this script does.
            var initialScript = "0000-Migrations-Setup.sql";
            connection.Execute(
                GetSql(
                    initialScript,
                    services.GetRequiredService<IWebHostEnvironment>()
                )
            );
            // Insert a record into our database so we know it was run in this database
            connection.Execute(
                @"INSERT INTO MigrationScripts (Name, ExecutionDate) 
                  VALUES (@Name, GETDATE())",
                new
                {
                    Name = initialScript
                }
            );
        }

        // Get all scripts that have been executed from the database 
        var executedScripts = connection.Query<string>("SELECT Name FROM MigrationScripts");
        // Get all the Migrations Scripts, checkout the scripts for an example of loading them from the file system.
        var files = GetMigrationFiles(
            services.GetRequiredService<IWebHostEnvironment>()
        ).Where(
            fileName => !executedScripts.Contains(
                Path.GetFileName(fileName)
            )
        );

        // Run Migrations that are not already in MigrationScripts Table
        foreach (var file in files)
        {
            connection.Execute(
                File.ReadAllText(file)
            );
            // Record Script was Executed, so we keep track, and restarts don't run the same script twice.
            connection.Execute(
                @"INSERT INTO MigrationScripts (Name, ExecutionDate) 
                  VALUES (@Name, GETDATE())",
                new
                {
                    Name = Path.GetFileName(file)
                }
            );
        }
    }

    // This will return the all the Text from the supplied fileName.
    // The root path is the same used by GetMigrationFiles to get a list of all files.
    private static string GetSql(
        string fileName,
        IWebHostEnvironment env
    ) => File.ReadAllText(
        Path.Combine(
            // This is where this application keeps the Database Scripts.
            env.ContentRootPath,
            "App_Data",
            "Database",
            "Scripts",
            fileName
        )
    );

    /// Grabs all the Migration file FullNames from a location relative to the ContentRootPath of the running WebHost Environment.
    // It will return them ordered by name
    private static IList<string> GetMigrationFiles(
        IWebHostEnvironment env
    ) => Directory.GetFiles(
        Path.Combine(
            // This is where this application keeps the Database Scripts.
            env.ContentRootPath,
            "App_Data",
            "Database",
            "Scripts"
        )
    ).OrderBy(
        path => path
    ).ToList();

    /// This will use the connection string and check to see if the Server has the Database.
    // If the Server does not have the Database it will create a new one with the name provided.
    private static void CheckCreateDatabase(
        string connectionString,
        string database
    )
    {
        using var connection = new SqlConnection(connectionString);
        // A simple script to check for the existence of a database, by name.
        var databases = connection.ExecuteScalar<int>(
            "SELECT count(1) FROM sys.sysdatabases WHERE name = @Name",
            new
            {
                Name = database
            }
        );
        if (databases == 0)
        {
            // Create the Database
            connection.Execute(
                $"CREATE DATABASE [{database}];"
            );
        }
    }
}
~~~

~~~ sql
-- Check for existing, skip is already found in Database
IF NOT EXISTS(SELECT 1 FROM sys.Objects
    WHERE Object_id = OBJECT_ID(N'[dbo].[MigrationScripts]')
        AND Type = N'U')
BEGIN
    CREATE TABLE [dbo].[MigrationScripts] (
        [Name]  NVARCHAR (320) NOT NULL,
        [ExecutionDate]  NVARCHAR (320) NOT NULL,
    );
END
~~~

~~~ sql
-- Check for existing, skip is already found in Database
IF NOT EXISTS(SELECT 1 FROM sys.Objects
    WHERE Object_id = OBJECT_ID(N'[dbo].[Registration]')
        AND Type = N'U')
BEGIN
    -- Create Registration table
    CREATE TABLE [dbo].[Registration] (
        [UserId] NVARCHAR (320) PRIMARY KEY,
        [Email] NVARCHAR (320) NOT NULL,
        [Name] NVARCHAR (320) NOT NULL,
        [Status] NVARCHAR (320) DEFAULT 'PENDING',
        [GameDetails] NVARCHAR (MAX) NULL,
        [Why] NVARCHAR (MAX) NULL,
    );
END

~~~


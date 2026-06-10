---
layout: null
title: The MySQL UE4 Plugin Documentation
---

<style>
  body {
    font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
    background-color: #ffffff;
    color: #2b2b2b;
    margin: 0;
    padding: 40px;
    line-height: 1.5;
    -webkit-font-smoothing: antialiased;
  }
  
  .document-container {
    max-width: 900px;
    margin: 0 auto;
    background: #ffffff;
    padding: 20px;
  }

  /* Header Banner Layout */
  .hero-banner {
    display: flex;
    align-items: center;
    background-color: #cc6666; /* Matching the muted red/coral color banner */
    height: 140px;
    position: relative;
    margin-bottom: 60px;
    margin-top: 40px;
  }

  /* The Logo Circle Element */
  .logo-circle {
    width: 170px;
    height: 170px;
    background-color: #ffffff;
    border: 5px solid #cc6666;
    border-radius: 50%;
    position: absolute;
    left: 40px;
    top: -20px;
    display: flex;
    align-items: center;
    justify-content: center;
    box-shadow: 0 4px 10px rgba(0,0,0,0.05);
    z-index: 10;
  }

  .logo-text {
    color: #cc6666;
    font-weight: 900;
    font-size: 2.8rem;
    letter-spacing: -1px;
  }

  /* Banner Text Configuration */
  .banner-titles {
    margin-left: 250px; /* Pushes text to the right of the overlapping circle */
    color: #000000;
  }

  .banner-titles h1 {
    font-size: 2.2rem;
    font-weight: 900;
    text-transform: uppercase;
    margin: 0;
    letter-spacing: 0.5px;
  }

  .banner-titles p {
    font-size: 0.85rem;
    font-weight: bold;
    text-transform: uppercase;
    margin: 5px 0 0 0;
    letter-spacing: 0.5px;
  }

  /* Responsive Two-Column Layout Grid */
  .main-grid {
    display: grid;
    grid-template-columns: 1fr 1.4fr;
    gap: 60px;
    margin-top: 20px;
  }

  /* Structural Headings with Thick Red Underlines */
  .section-title {
    font-size: 1.3rem;
    font-weight: 900;
    text-transform: uppercase;
    margin-top: 0;
    margin-bottom: 15px;
    letter-spacing: 0.5px;
    color: #000000;
  }

  .title-underline {
    width: 100%;
    height: 7px;
    background-color: #cc6666;
    margin-bottom: 20px;
  }

  /* Content Styling & Micro-Typography */
  p.doc-text {
    font-size: 0.95rem;
    color: #333333;
    margin-bottom: 15px;
    text-align: justify;
  }

  .subsection-heading {
    font-size: 0.85rem;
    font-weight: 900;
    text-transform: uppercase;
    margin-top: 25px;
    margin-bottom: 10px;
    letter-spacing: 0.5px;
  }

  code.inline-code {
    background-color: #f7f7f9;
    padding: 2px 5px;
    border: 1px solid #e1e1e8;
    border-radius: 3px;
    font-family: Monaco, Consolas, Courier, monospace;
    font-size: 0.85rem;
    color: #333333;
  }

  div.code-block-highlight {
    font-family: Monaco, Consolas, Courier, monospace;
    font-weight: bold;
    font-style: italic;
    font-size: 0.9rem;
    margin-top: 15px;
    padding-left: 10px;
    color: #000000;
  }
</style>

<div class="document-container">

  <!-- Header Section Wrapper -->
  <div class="hero-banner">
    <div class="logo-circle">
      <span class="logo-text">UE4</span>
    </div>
    <div class="banner-titles">
      <h1>The MySQL UE4 Plugin</h1>
      <p>Created by Austin Berry</p>
    </div>
  </div>

  <div>
    ```cpp
    // Fill out your copyright notice in the Description page of Project Settings.
#include "MySQLPrivatePCH.h"
#include "MySQLEngine.h"

MySQLEngine::MySQLEngine()
{
}

void MySQLEngine::OpenConnection(std::string DatabaseAddress, std::string Username, std::string Password, std::string DatabaseName, bool OverrideConnection)
{
	if (MySQLDatabaseManager::GetInstance()->GetConnection()->isConnected == false || OverrideConnection == true)
		MySQLDatabaseManager::GetInstance()->InitializeDatabaseConnection(DatabaseAddress, Username, Password, DatabaseName);
	else
	{
		MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "Cannot open a new connection while another connection is open. If you are purposly trying to leave multiple connections open on your SQL server, then add 'true' as an extra parameter onto the openconnection function.\n\t\t While this error has automatically been handled, it is reccommended to go back to your source file and correct the extra OpenConnection() statement added.\n\t\tWARNING: By doing so, you will cause processes on your sql server to infinitly run until manually killing the processes. This could also cause future refused MySQL connections.");
	}
}

void MySQLEngine::OpenConnection(ConnectionConfigurationData ConnectionData)
{
	OpenConnection(ConnectionData.DatabaseAddress, ConnectionData.Username, ConnectionData.Password, ConnectionData.DatabaseName, false);
}

void MySQLEngine::CloseConnection()
{
	if (MySQLDatabaseManager::GetInstance()->GetConnection() != nullptr)
	{
		if (MySQLDatabaseManager::GetInstance()->GetConnection()->isConnected)
		{
			MySQLDatabaseManager::GetInstance()->CloseMySQLConnection();
		}

		else MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "Cannot Close a Connection that Doesn't exist.");
	}

	else
	{
		MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "Cannot Close Connection - Connection Returned Null");
	}
}

bool MySQLEngine::ExecuteQuery(std::string Query)
{
	if(MySQLDatabaseManager::GetInstance()->GetConnection() != nullptr)
		if (MySQLDatabaseManager::GetInstance()->GetConnection()->isConnected)
		{
			try
			{
				MySQLDatabaseManager::GetInstance()->ExecuteMySQLQuery(Query, MySQLDatabaseManager::GetInstance()->GetConnection());
				return true; //return true if query executed successfully
			}
			catch (...)
			{
				MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "The command was not successfully executed. Please contact administrator for further issues.");
				return false;
			}
		}
	MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "There doesn't seem to be a connection to the sql server. Command failed.");
	return false;
}

MySQLQueryResult MySQLEngine::FetchResults(std::string Query)
{
	if (MySQLDatabaseManager::GetInstance()->GetConnection() != nullptr)
		if (MySQLDatabaseManager::GetInstance()->GetConnection()->isConnected)
		{
			try
			{
				MySQLQueryResult Results = MySQLDatabaseManager::GetInstance()->FetchMySQLQueryResults(Query, MySQLDatabaseManager::GetInstance()->GetConnection());
				return Results;
			}
			catch (...)
			{
				MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "There was an error trying to Fetch Results. Returning an empty QueryResult...");
				return MySQLQueryResult();
			}
		}
		else
		{
			MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "There is no connection to the SQL server.");
			return MySQLQueryResult();
		}
	else
	{
		MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "Cannot Close Connection - Connection Returned Null");
		return MySQLQueryResult();
	}
}

MySQLRowResult MySQLEngine::GetRowWhereColumnEquals(MySQLQueryResult * QueryResults, std::string ColumnName, std::string ColumnValue)
{
	MySQLRowResult RowOut = MySQLRowResult();
	int count = 0;

	for (auto Row : QueryResults->Rows)
	{
		for (auto Val : Row.KeyValue)
		{
			if (Val.Key == ColumnName)
			{
				if (Val.Value == ColumnValue)
				{
					if (RowOut.KeyValue.Max() == 0)
						RowOut = Row;
					else
					{
						count++;
					}
				}
				else
				{
					continue;
				}
			}
			else
				continue;
		}
	}

	if (RowOut.KeyValue.Max() != 0)
	{
		if (count > 0)
		{
			MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "There was more than one row with the value of " + ColumnValue + " in the " + ColumnName + " column. Row Result will only return the first value found.\n\t\t" + std::to_string(count) + " other instances found. Total Rows Matching Argument: " + std::to_string((count + 1)));
			return RowOut;
		}
		else return RowOut;
	}

	MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "No Column exists where column is equal to " + ColumnName + " and value is equal to " + ColumnValue);
	return MySQLRowResult();
}

TArray<MySQLRowResult> MySQLEngine::GetRowsWhereColumnValuesEqual(MySQLQueryResult * QueryResults, std::string ColumnName, std::string ColumnValue)
{
	TArray<MySQLRowResult> Rows;

	for (auto Row : QueryResults->Rows)
	{
		for (auto Val : Row.KeyValue)
		{
			if (Val.Key == ColumnName)
			{
				if (Val.Value == ColumnValue)
				{
					Rows.Add(Row);
				}
				else
				{
					continue;
				}
			}
			else
				continue;
		}
	}
	
	if (Rows.Max() == 0)
	{
		MySQLDatabaseManager::GetInstance()->GetConnection()->MySQLErrorManager(false, "No Column exists where column is equal to " + ColumnName + " and value is equal to " + ColumnValue);
		return TArray<MySQLRowResult>();
	}

	return Rows;
}

ConnectionConfigurationData::ConnectionConfigurationData()
{
	DatabaseAddress = "";
	Username = "";
	Password = "";
	DatabaseName = "";
}

ConnectionConfigurationData::ConnectionConfigurationData(std::string DatabaseAddr, std::string UsrName, std::string Passwd, std::string DBName)
{
	DatabaseAddress = DatabaseAddr;
	Username = UsrName;
	Password = Passwd;
	DatabaseName = DBName;
}

    ```
  </div>

  <!-- Content Section Wrapper -->
  <div class="main-grid">
    
    <!-- Left Column (Purpose & Notes) -->
    <div>
      <div class="section-title">Purpose</div>
      <div class="title-underline"></div>
      <p class="doc-text">The MySQL plugin allows Oracle's MySQL to operate within the realm of the Unreal Engine 4 C++ API. It works based off of strings; the developer will need to convert data types accordingly.</p>
      
      <div class="section-title" style="margin-top: 50px;">! Notes</div>
      <div class="title-underline"></div>
      <p class="doc-text">It is important to remember that the MySQL plugin will need the <code class="inline-code">mysqlcppconn.dll</code> which is located in the plugin's directory. (<code class="inline-code">./MySQL/MySQL/ThirdParty/mysqlcppconn.dll</code>) The dynamic link library should be dropped alongside the project's executable file, or the program will receive a run-time error.</p>
    </div>
    
    <!-- Right Column (Documentation Setup Instructions) -->
    <div style="border-left: 1px solid #e1e4e8; padding-left: 40px;" markdown="1">
      <div class="section-title">Plugin Pre-Usage Setup Documentation</div>
      <div class="title-underline"></div>
      
      <div class="subsection-heading">Project's Target .cs File</div>
      <p class="doc-text">The plugin relies on the ability to use C++ exceptions. In order for this to properly work, the developer must verify that their <code class="inline-code">project.Target.cs</code> file is properly configured to allow the plugin to work properly. <em>(Note: The project may not compile if this step is not done)</em></p>
      
      <p class="doc-text">Inside of the Target.cs file, ensure that inside of the public ProjectTarget class implements:</p>
      
      <div class="code-block-highlight">UEBuildConfiguration.bForceEnableExceptions = true;</div>
      
      <p class="doc-text" style="margin-top: 15px;">inside of the class' constructor as shown in Figure 1.1.</p>
    </div>

  </div>
</div>

# Alternative code (dotnet core practical class 1)

This is just an alternative tecnhology to connect to SQL Server database. 

## Docker and SQL Server

To run SQL Server, you can use SQL Server or Docker (https://docs.docker.com/desktop/install/windows-install/) 

To run SQL server via docker, you can verify followig documentation: https://hub.docker.com/_/microsoft-mssql-server. 
This is not excludes the installation of SQL Server Management Studio (which is mandatory for a few practical classes).

Here is an example of how to run SQL Sever in Docker: 

```bash 
docker run -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=yourStrong!" -p 11433:1433 -d mcr.microsoft.com/mssql/server:2022-latest
```

## A web application in dotnetcore 

First, install dotnetcore - download it here: https://dotnet.microsoft.com/en-us/download

To create application:

```bash 
dotnet new webapp -o MyWebApp --no-https -f net7.0
```

Go to the aula01 directory in a command line:

```bash 
cd aula01
```
To add SqlClient

```bash 
dotnet add package Microsoft.Data.SqlClient
```

To run app in dev mode:

```bash 
dotnet watch 
```

Open browser,  http://localhost:5047 and see page.


Now it is the time to edit the code. 

- See `Pages\Index.cshtml.cs` and `Pages\Index.cshtml`

Forms builder help, https://beautifytools.com/html-form-builder.php.

You will need to add similar code like this:

```html

<div>
    @Html.Raw(@Model.Message)
  
    <form method="post" >
        <fieldset>
          <legend>Aula01 - BD</legend>
          <div>
            <label for="Server">Server:</label>
            <input type="text" name="server" value="" />
          </div>
          <div>
            <label for="Username">Username:</label>
            <input type="text" name="username" value="" />
          </div>
          <div>
            <label for="Password">Password:</label>
            <input type="password" name="password" value="" />
          </div>
          <div>
            <label>&nbsp;</label>
            <input type="submit" value="Submit" class="submit" />
          </div>
        </fieldset>
      </form>
</div>
```


 See `Pages\Index.cshtml.cs` and add the following code: 

```csharp 
    public string Message { get; private set; } = "";
    
    public void OnPost(){
        var server = Request.Form["server"];
        var username = Request.Form["username"];
        var password = Request.Form["password"];
        var databaseName = "Hello";
        Boolean connectionState = TestDBConnection(server, databaseName, username, password);
        if (connectionState)
            Message += $"Successful connection to database {databaseName} on the {server}" ;
        else
            Message += $"Failed to open connection to database due to the error";

    }

    private Boolean TestDBConnection(string dbServer, string dbName, string userName, string userPass)
    {

        SqlConnectionStringBuilder builderSQL = new SqlConnectionStringBuilder();
        builderSQL.DataSource = dbServer;  
        builderSQL.UserID = userName;    
        builderSQL.Password = userPass; 
        builderSQL.InitialCatalog = dbName;
        builderSQL.TrustServerCertificate = true; // to avoid issues with SSL (only for dev purposes).
        Console.Write("Connecting to SQL Server ... ");
        try{
            using (SqlConnection connection = new SqlConnection(builderSQL.ConnectionString))
            {
    
                connection.Open();
                if (connection.State == System.Data.ConnectionState.Open) {
                    Message += "<br />" + getTableContent(connection);
                    return true;
                }
                else{
                    return false;
                }

                
            }
        }
        catch(Exception e){
             return false;
        }
        return true;
    }

    private string getTableContent(SqlConnection CN)
        {
            string str = "";
         
            try
            {
                
                if (CN.State ==  System.Data.ConnectionState.Open)
                {
                    int cnt = 1;
                    SqlCommand sqlcmd = new SqlCommand("SELECT * FROM Hello", CN);
                    SqlDataReader reader;
                    reader = sqlcmd.ExecuteReader();

                    while (reader.Read())
                    {
                        str += cnt.ToString() + " - " + reader.GetInt32(reader.GetOrdinal("MsgID")) + ", ";
                        str += reader.GetString(reader.GetOrdinal("MsgSubject"));
                        str += "<br/>";
                        cnt += 1;
                    }
                }
            }
            catch (Exception ex)
            {
                str += "Failed to open connection to database due to the error ";
            }

  

            return str;
        }
```

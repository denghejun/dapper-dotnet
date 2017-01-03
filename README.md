## Sample
```
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Data.SqlClient;
using System.Linq;
using System.Text;
using System.Threading.Tasks;
using Dapper;

namespace dapper_dotnet
{
    class Program
    {
        private static SqlConnection _conn = null;
        private static SqlConnection GetOpenConnection()
        {
            var conn = new SqlConnection(ConfigurationManager.ConnectionStrings["Local"].ConnectionString);
            conn.Open();
            return conn;
        }

        protected static SqlConnection Connection => _conn ?? (_conn = GetOpenConnection());

        static void Main(string[] args)
        {
            using (var tran = Connection.BeginTransaction())
            {
                var rows = Connection.Execute("INSERT INTO tempdb.dbo.LocalUser VALUES(@Name,@Age)", new LocalUser() { Name = "Leo", Age = 100 }, transaction: tran);
                var users = Connection.Query<LocalUser>("SELECT Name,Age FROM tempdb.dbo.LocalUser WITH(NOLOCK) WHERE Name = @Name", new { Name = "Leo" }, transaction: tran);
                tran.Commit();
                Console.WriteLine(users.Count());
            }

            Console.ReadKey();
        }

    }

    class LocalUser
    {
        public string Name { get; set; }
        public int? Age { get; set; }
    }
}
```
```
<!-- App.config -->
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <connectionStrings>
    <!--SSPI means: Windows Authentication; or you can user "user id=xxx;password=yyy;" replace "Integrated Security=SSPI;"-->
    <add name="Local" connectionString="Data Source=.\localsqlinstance;Initial Catalog=tempdb;Integrated Security=SSPI;"></add>
  </connectionStrings>
  <startup>
    <supportedRuntime version="v4.0" sku=".NETFramework,Version=v4.5.2" />
  </startup>
</configuration>
```

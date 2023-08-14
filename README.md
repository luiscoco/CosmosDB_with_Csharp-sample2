# CosmosDB_with_Csharp-sample2

## Source code
```csharp
using System;
using System.IO;
using System.Threading.Tasks;
using Microsoft.Extensions.Configuration;
using Microsoft.Azure.Cosmos;
using System.Collections.Concurrent;
using System.ComponentModel;

namespace CosmosDBApp
{
    class Program
    {
        static async Task Main(string[] args)
        {
            IConfiguration configuration = new ConfigurationBuilder()
                        .SetBasePath(Directory.GetCurrentDirectory())
                        .AddJsonFile("appsettings.json")
                        .Build();

            CosmosSettings cosmosSettings = configuration.GetSection("CosmosSettings").Get<CosmosSettings>();

            using CosmosClient client = new(
                accountEndpoint: cosmosSettings.Endpoint,
                authKeyOrResourceToken: cosmosSettings.Key
            );

            Database database = await client.CreateDatabaseIfNotExistsAsync(cosmosSettings.DatabaseId);
            Console.WriteLine($"New database:\t{database.Id}");

            Microsoft.Azure.Cosmos.Container container = await database.CreateContainerIfNotExistsAsync(
                id: cosmosSettings.ContainerId,
                partitionKeyPath: "/categoryId",
                throughput: 400
            );
            Console.WriteLine($"New container:\t{container.Id}");

            Product newItem = new Product
            {
                id = "70b63682-b93a-4c77-aad2-65501347265f",
                categoryId = "61dba35b-4f02-45c5-b648-c6badc0cbd79",
                categoryName = "gear-surf-surfboards",
                name = "Yamba Surfboard",
                quantity = 12,
                sale = false
            };

            Product createdItem = await container.CreateItemAsync(newItem, new PartitionKey(newItem.categoryId));
            Console.WriteLine($"Created item:\t{createdItem.id}\t[{createdItem.categoryName}]");

            Product readItem = await container.ReadItemAsync<Product>(newItem.id, new PartitionKey(newItem.categoryId));

            var query = new QueryDefinition("SELECT * FROM products p WHERE p.categoryId = @categoryId")
                .WithParameter("@categoryId", newItem.categoryId);

            using FeedIterator<Product> feed = container.GetItemQueryIterator<Product>(queryDefinition: query);

            while (feed.HasMoreResults)
            {
                FeedResponse<Product> response = await feed.ReadNextAsync();
                foreach (Product item in response)
                {
                    Console.WriteLine($"Found item:\t{item.name}");
                }
            }
        }
    }

    public class CosmosSettings
    {
        public string Endpoint { get; set; }
        public string Key { get; set; }
        public string DatabaseId { get; set; }
        public string ContainerId { get; set; }
    }

    public class Product
    {
        public string id { get; set; }
        public string categoryId { get; set; }
        public string categoryName { get; set; }
        public string name { get; set; }
        public int quantity { get; set; }
        public bool sale { get; set; }
    }
}
```

## appsettings.json

```json
{
  "CosmosSettings": {
    "Endpoint": "https://luiscococosmosdb.documents.azure.com:443/",
    "Key": "2QSCj43kWVA4tTtHCF5ZZvQ9Jth1J32QYl5pmEkeZtjayl0c0cN7zpND572AHGeO1vXsfW4NcVACACDbRGt1ng==",
    "DatabaseId": "cosmicworks",
    "ContainerId": "products"
  }
}
```

















